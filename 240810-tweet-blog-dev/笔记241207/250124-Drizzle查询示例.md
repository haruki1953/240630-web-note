### 250128-关系计数
```ts
// src\services\post-control\control-forward\forward-setting.ts
// 转发帖子统计
export const postControlForwardSettingPostCountService = async () => {
  // 帖子总数
  const totalPostCount = await drizzleDb.$count(
    drizzleSchema.posts,
    drizzleOrm.eq(drizzleSchema.posts.isDeleted, false)
  )
  // 遍历转发配置，统计转发的贴子数
  const { forwardSettingList } = forwardSystem.forwardStore()
  const forwardSettingPostList = await Promise.all(
    forwardSettingList.map(async (forwardSettingItem) => {
      const uuid = forwardSettingItem.uuid
      const count = await drizzleDb.$count(
        drizzleSchema.posts,
        drizzleOrm.and(
          drizzleOrm.eq(drizzleSchema.posts.isDeleted, false),
          drizzleOrm.exists(
            drizzleDb.select().from(drizzleSchema.postForwards)
              .where(drizzleOrm.and(
                drizzleOrm.eq(drizzleSchema.postForwards.postId, drizzleSchema.posts.id),
                drizzleOrm.eq(drizzleSchema.postForwards.forwardConfigId, uuid)
              ))
          )
        )
      )
      return {
        uuid,
        count
      }
    })
  )
  return {
    totalPostCount,
    forwardSettingPostList
  }
}
```

### 250128-关系查询、没有记录
```ts
// src\services\post-control\control-forward\forward-auto.ts
// 在数据库中寻找父帖，且应未被转帖
const p_G_NewToOld_PrismaPostFindUniqueByParentPostId = async (
  data: PostControlForwardAutoJsonType & {
    postItemParentPostId: string
  }
) => {
  const {
    forwardConfigId,
    postItemParentPostId
  } = data
  // 应未被转帖，即代表其转发记录中应没有forwardConfigId相等的
  const parentPostItem = await drizzleDb.query.posts.findFirst({
    where: drizzleOrm.and(
      drizzleOrm.eq(drizzleSchema.posts.id, postItemParentPostId),
      drizzleOrm.eq(drizzleSchema.posts.isDeleted, false),
      drizzleOrm.notExists(
        drizzleDb.select().from(drizzleSchema.postForwards)
          .where(drizzleOrm.and(
            drizzleOrm.eq(drizzleSchema.postForwards.postId, drizzleSchema.posts.id),
            drizzleOrm.eq(drizzleSchema.postForwards.forwardConfigId, forwardConfigId)
          ))
      )
    ),
    columns: {
      id: true,
      parentPostId: true
    }
  })
  return parentPostItem
}

// src\services\post-control\control-forward\forward-auto.ts
// 在数据库中根据数据查找未转发的帖子
const p_G_OldOrNew_PrismaPostFindManyByForwardingOrder = async (
  data: PostControlForwardAutoJsonType
) => {
  const {
    forwardConfigId,
    forwardingOrder,
    forwardingNumber
  } = data

  const orderByCreatedAt = (() => {
    if (forwardingOrder === 'old-to-new') {
      // 从旧到新
      return drizzleOrm.asc(drizzleSchema.posts.createdAt)
    } else { // 'new-to-old'
      // 从新到旧
      return drizzleOrm.desc(drizzleSchema.posts.createdAt)
    }
  })()
  const orderBy = [
    orderByCreatedAt,
    drizzleOrm.asc(drizzleSchema.posts.id)
  ]

  const forwardingPostList = await drizzleDb.query.posts.findMany({
    where: drizzleOrm.and(
      drizzleOrm.eq(drizzleSchema.posts.isDeleted, false),
      // 未转发
      drizzleOrm.notExists(
        drizzleDb.select().from(drizzleSchema.postForwards)
          .where(drizzleOrm.and(
            drizzleOrm.eq(drizzleSchema.postForwards.postId, drizzleSchema.posts.id),
            drizzleOrm.eq(drizzleSchema.postForwards.forwardConfigId, forwardConfigId)
          ))
      )
    ),
    orderBy,
    limit: forwardingNumber,
    columns: {
      id: true,
      parentPostId: true
    }
  })
  return forwardingPostList
}
```


### 250127-计数
```ts
// src\services\profile.ts
// 计数
export const profileGetDataService = async () => {
  // 这里没有where，回收站的帖子也会计数。会被访客调用，所以想这样节省性能
  const post = await drizzleDb.$count(drizzleSchema.posts).catch((error) => {
    logUtil.info({
      title: '推文数量统计失败',
      content: String(error)
    })
    throw new AppError('推文数量统计失败')
  })
  const image = await drizzleDb.$count(drizzleSchema.images).catch((error) => {
    logUtil.info({
      title: '图片数量统计失败',
      content: String(error)
    })
    throw new AppError('图片数量统计失败')
  })

  return {
    post,
    image
  }
}
```

### 250127-游标、关系、搜索
```ts
// src\services\post.ts
// 帖子分页查询
export const postGetByCursorService = async (
  cursorId: PostInferSelect['id'], query: PostGetByCursorQueryType
) => {
  // 在正式查询之前，首先要查询游标对应的数据
  const cursorData = await (async () => {
    if (cursorId == null) {
      return undefined
    }
    const data = await baseFindPostById(cursorId)
    if (data == null) {
      throw new AppError('推文游标无效')
    }
    return data
  })()

  // 游标条件
  const ddWhereCursor = (() => {
    if (cursorData == null) {
      return undefined
    }
    return drizzleOrm.or(
      // 查询createdAt比游标所指的小的，因为是createdAt降序排序
      drizzleOrm.lt(drizzleSchema.posts.createdAt, cursorData.createdAt),
      // 需要考虑到createdAt相同的情况，所以需要借助id来确认createdAt相同时的顺序
      // 借助id升序降序随便，但要与orderBy相同
      drizzleOrm.and(
        drizzleOrm.eq(drizzleSchema.posts.createdAt, cursorData.createdAt),
        drizzleOrm.gt(drizzleSchema.posts.id, cursorData.id)
      )
    )
  })()

  // orderBy
  const ddOrderBy = [
    drizzleOrm.desc(drizzleSchema.posts.createdAt),
    drizzleOrm.asc(drizzleSchema.posts.id)
  ]
  // Limit 分页个数
  const ddLimit = postConfig.postNumInPage

  // 查询条件，内容搜索
  const ddWhereContent = (() => {
    if (query.content == null) {
      return undefined
    }
    return drizzleOrm.or(
      // 和content对比
      drizzleOrm.like(
        drizzleSchema.posts.content,
        parseLikeInput(query.content).value
      ),
      // 还要和图片的alt对比
      // exists postsToImages
      drizzleOrm.exists(
        drizzleDb.select().from(drizzleSchema.postsToImages).where(
          drizzleOrm.and(
            drizzleOrm.eq(drizzleSchema.postsToImages.postId, drizzleSchema.posts.id),
            // exists images
            drizzleOrm.exists(
              drizzleDb.select().from(drizzleSchema.images).where(
                drizzleOrm.and(
                  drizzleOrm.eq(drizzleSchema.images.id, drizzleSchema.postsToImages.imageId),
                  // like images.alt
                  drizzleOrm.like(
                    drizzleSchema.images.alt,
                    parseLikeInput(query.content).value
                  )
                )
              )
            )
          )
        )
      )
    )
  })()

  const isDeleted = (() => {
    if (
      query.isDelete === 'false' ||
      query.isDelete === undefined
    ) {
      return false
    } else if (query.isDelete === 'true') {
      return true
    } else { // 'all'
      return undefined
    }
  })()
  // 查询条件，是否删除
  const ddWhereDel = (() => {
    if (isDeleted == null) {
      return undefined
    }
    return drizzleOrm.eq(drizzleSchema.posts.isDeleted, isDeleted)
  })()

  // 总查询条件
  const ddWhere = drizzleOrm.and(
    ddWhereCursor,
    ddWhereContent,
    ddWhereDel
  )

  // 数据库查询
  let posts = await drizzleDb.query.posts.findMany({
    where: ddWhere,
    orderBy: ddOrderBy,
    limit: ddLimit,
    with: {
      ...dbQueryWithOnPost,
      parentPost: {
        with: {
          ...dbQueryWithOnPost
        }
      }
    }
  }).catch((error) => {
    logUtil.info({
      title: '推文获取失败',
      content: String(error)
    })
    throw new AppError('推文获取失败')
  })
  // 查询后的处理
  posts = posts.map((post) => {
    return {
      ...post,
      // 根据条件控制是否获取parentPost
      parentPost: (() => {
        if (isDeleted == null) {
          // isDeleted为undefined即为不筛选
          return post.parentPost
        }
        if (post.parentPost == null) {
          return post.parentPost
        }
        // 获取对应isDeleted的
        if (post.parentPost.isDeleted === isDeleted) {
          return post.parentPost
        }
        // 否则返回null
        return null
      })()
    }
  })
  return dataDQWPostGetByCursorHandle(posts)
}
```


### 250127-查询后的自行处理
```ts
// src\services\post.ts
// 帖子id查询
export const postGetByIdService = async (
  id: PostInferSelect['id'], query?: PostGetByIdQueryType
) => {
  // 是否也查询被删除的
  const isKeepIsDetele = (() => {
    if (
      query?.keepIsDetele == null ||
        query.keepIsDetele === 'false'
    ) {
      return false
    }
    return true
  })()
  const ddWhereDel = (() => {
    if (isKeepIsDetele) {
      return undefined
    }
    return drizzleOrm.eq(drizzleSchema.posts.isDeleted, false)
  })()
  const ddWhereId = drizzleOrm.eq(drizzleSchema.posts.id, id)

  let post = await drizzleDb.query.posts.findFirst({
    where: drizzleOrm.and(ddWhereId, ddWhereDel),
    with: {
      ...dbQueryWithOnPost,
      postImports: true,
      postForwards: true,
      parentPost: {
        // 好像对一的关系无法where，后面自己判断吧
        with: {
          ...dbQueryWithOnPost,
          postImports: true,
          postForwards: true
        }
      },
      // 回复，回复的回复
      replies: {
        where: ddWhereDel,
        with: {
          ...dbQueryWithOnPost,
          replies: {
            where: ddWhereDel,
            with: {
              ...dbQueryWithOnPost
            }
          }
        }
      }
    }
  }).catch((error) => {
    logUtil.info({
      title: '推文获取失败',
      content: String(error)
    })
    throw new AppError('推文获取失败')
  })
  if (post == null) {
    throw new AppError('推文不存在', 400)
  }

  post = {
    ...post,
    parentPost: (() => {
      if (isKeepIsDetele) {
        return post.parentPost
      }
      if (post.parentPost == null) {
        return post.parentPost
      }
      // 排除isDeleted
      if (post.parentPost.isDeleted) {
        return null
      }
      return post.parentPost
    })()
  }

  return dataDQWPostGetByIdHandle(post)
}
```

### 250126-操作、关联、事务

#### 事务错误
发现不管是手动rollback还是抛出错误，事务都无法回滚。好像事务函数不能是异步的，只能是同步的才支持回滚
https://github.com/drizzle-team/drizzle-orm/issues/1472

```
通过在查询后添加 .run() 或 .all() 可以让查询变成同步的
如果不需要返回值使用 .run() ，需要返回值使用 .all()
.returning()
.all()
```

```ts
// src\services\post.ts
// 帖子创建
export const postSendService = async (postInfo: PostSendJsonType) => {
  // 确认图片存在
  if (postInfo.images != null) {
    await Promise.all(postInfo.images.map(async (imgId) => {
      const imgData = await baseFindImageById(imgId)
      if (imgData == null) {
        throw new AppError('图片不存在', 400)
      }
    }))
  }
  // 确认父帖存在
  if (postInfo.parentPostId != null) {
    const postData = await baseFindPostById(postInfo.parentPostId)
    if (postData == null) {
      throw new AppError('父帖不存在', 400)
    }
  }

  const newDate = new Date()

  // 事务
  let addedPost
  try {
    addedPost = drizzleDb.transaction((drizzleTx) => {
      // 添加帖子
      const insertedPosts = drizzleTx.insert(drizzleSchema.posts)
        .values({
          // 时间之类最好手动指定，因为默认值只以秒为精确度
          createdAt: postInfo.createdAt ?? newDate,
          addedAt: newDate,
          updatedAt: newDate,
          content: postInfo.content ?? '',
          imagesOrder: (() => {
            if (postInfo.images == null) {
              return undefined
            }
            return JSON.stringify(postInfo.images)
          })(),
          parentPostId: postInfo.parentPostId,
          isDeleted: postInfo.isDeleted
        })
        .returning()
        .all()

      if (insertedPosts.length === 0) {
        throw new AppError('帖子添加失败', 500)
      }
      const addedPost = insertedPosts[0]

      // 关联图片
      ;(() => {
        if (postInfo.images == null) {
          return
        }
        if (postInfo.images.length === 0) {
          return
        }
        drizzleTx.insert(drizzleSchema.postsToImages)
          .values((() => {
            return postInfo.images.map((imageId) => {
              return {
                postId: addedPost.id,
                imageId
              }
            })
          })())
          .run()
      })()

      return addedPost
    })
  } catch (error) {
    logUtil.info({
      title: '推文添加失败',
      content: String(error)
    })
    throw new AppError('推文添加失败')
  }

  // 最后再次查询帖子数据
  const post = await drizzleDb.query.posts.findFirst({
    where: drizzleOrm.eq(drizzleSchema.posts.id, addedPost.id),
    with: dbQueryWithOnPost
  })
  if (post == null) {
    throw new AppError('帖子添加失败', 500)
  }

  return dataDQWPostSendHandle(post)
}

// src\services\post.ts
// 帖子更新
export const postUpdateService = async (postInfo: PostUpdateJsonType) => {
  // 父帖不能为当前推文
  if (postInfo.id === postInfo.parentPostId) {
    throw new AppError('父帖不能为当前推文', 400)
  }
  // 确认帖子存在
  const targetPost = baseFindPostById(postInfo.id)
  if (targetPost == null) {
    throw new AppError('帖子不存在', 400)
  }
  // 确认图片存在
  if (postInfo.images != null) {
    await Promise.all(postInfo.images.map(async (imgId) => {
      const imgData = await baseFindImageById(imgId)
      if (imgData == null) {
        throw new AppError('图片不存在', 400)
      }
    }))
  }
  // 确认父帖存在
  if (postInfo.parentPostId != null) {
    const postData = await baseFindPostById(postInfo.parentPostId)
    if (postData == null) {
      throw new AppError('父帖不存在', 400)
    }
  }

  const newDate = new Date()

  // 事务
  try {
    drizzleDb.transaction((drizzleTx) => {
      // 更新数据
      drizzleTx.update(drizzleSchema.posts)
        .set({
          // 时间之类最好手动指定，因为默认值只以秒为精确度
          createdAt: postInfo.createdAt,
          updatedAt: newDate,
          content: postInfo.content,
          imagesOrder: (() => {
            if (postInfo.images == null) {
              return undefined
            }
            return JSON.stringify(postInfo.images)
          })(),
          parentPostId: postInfo.parentPostId,
          isDeleted: postInfo.isDeleted
        })
        .where(drizzleOrm.eq(drizzleSchema.posts.id, postInfo.id))
        .run()

      // 更新图片
      ;(() => {
        if (postInfo.images == null) {
          return
        }
        // 首先清空本帖子与图片的关联
        drizzleTx.delete(drizzleSchema.postsToImages)
          .where(drizzleOrm.eq(drizzleSchema.postsToImages.postId, postInfo.id))
          .run()
        // 然后进行关联
        if (postInfo.images.length === 0) {
          return
        }
        drizzleTx.insert(drizzleSchema.postsToImages)
          .values((() => {
            return postInfo.images.map((imageId) => {
              return {
                postId: postInfo.id,
                imageId
              }
            })
          })())
          .run()
      })()
    })
  } catch (error) {
    logUtil.info({
      title: '推文修改失败',
      content: String(error)
    })
    throw new AppError('推文修改失败')
  }

  // 最后再次查询帖子数据
  const post = await drizzleDb.query.posts.findFirst({
    where: drizzleOrm.eq(drizzleSchema.posts.id, postInfo.id),
    with: dbQueryWithOnPost
  })
  if (post == null) {
    throw new AppError('帖子更新失败', 500)
  }
  return post
}

// src\services\post.ts
// 帖子删除
export const postDeleteService = async (id: PostInferSelect['id'], query: PostDeleteQueryType) => {
  // 查找帖子并包含需要的数据
  const targetPost = await drizzleDb.query.posts.findFirst({
    where: drizzleOrm.eq(drizzleSchema.posts.id, id),
    with: dbQueryWithOnPost
  })
  if (targetPost == null) {
    throw new AppError('帖子不存在', 400)
  }
  const post = dataDQWPostBaseHandle(targetPost)

  // 事务
  try {
    drizzleDb.transaction((drizzleTx) => {
      // 解除图片关联
      drizzleTx.delete(drizzleSchema.postsToImages)
        .where(drizzleOrm.eq(drizzleSchema.postsToImages.postId, id))
        .run()
      // 删除帖子
      drizzleTx.delete(drizzleSchema.posts)
        .where(drizzleOrm.eq(drizzleSchema.posts.id, id))
        .run()
    })
  } catch (error) {
    logUtil.info({
      title: '推文删除失败',
      content: String(error)
    })
    throw new AppError('推文删除失败')
  }

  // 尝试删除图片
  const deletedImages = await (async () => {
    if (query.delateImage == null || query.delateImage === 'false') {
      return []
    }
    // else (query.delateImage === 'true')
    return await Promise.all(
      post.images.map(async (img) => {
        return await deleteImageByIdWhereNonePost(
          img.id
        ).catch(() => null)
      })
    )
  })()

  return {
    deletedPost: post,
    deletedImages
  }
}
```


### 250126-游标分页同时关系查询
```ts
// src\services\image.ts
// 图片游标分页同时关系查询
export const imageGetByCursorService = async (
  cursorId: ImageInferSelect['id'], query: ImageGetByCursorQueryType
) => {
  // 在正式查询之前，首先要查询游标对应的数据
  const cursorData = await (async () => {
    if (cursorId == null) {
      return undefined
    }
    const data = await baseFindImageById(cursorId)
    if (data == null) {
      throw new AppError('图片获取失败 | 游标无效')
    }
    return data
  })()

  // 弃用通过 ImageGetByCursorQueryType 来筛选图片（250126弃用）

  // where使其从游标开始查询
  const ddWhere = (() => {
    if (cursorData == null) {
      return undefined
    }
    return drizzleOrm.or(
      // 查询createdAt比游标所指的小的，因为是createdAt降序排序
      drizzleOrm.lt(drizzleSchema.images.createdAt, cursorData.createdAt),
      // 需要考虑到createdAt相同的情况，所以需要借助id来确认createdAt相同时的顺序
      // 借助id升序降序随便，但要与orderBy相同
      drizzleOrm.and(
        drizzleOrm.eq(drizzleSchema.images.createdAt, cursorData.createdAt),
        drizzleOrm.gt(drizzleSchema.images.id, cursorData.id)
      )
    )
  })()

  // orderBy
  const ddOrderBy = [
    drizzleOrm.desc(drizzleSchema.images.createdAt),
    drizzleOrm.asc(drizzleSchema.images.id)
  ]

  // Limit 分页个数
  const ddLimit = postConfig.imageNumInPage

  const images = await drizzleDb.query.images.findMany({
    where: ddWhere,
    orderBy: ddOrderBy,
    limit: ddLimit,
    with: dbQueryWithOnImageGet
  })

  return dataDQWImageGetByCursorHandle(images)
}

// 数据库图片查询时，会with的数据，是在id与游标查询是复用的
export const dbQueryWithOnImageGet = {
  postsToImages: {
    with: {
      post: {
        with: {
          replies: true,
          postsToImages: true
        }
      }
    }
  }
} as const
```

### 250126-查询后对数据的处理
```ts
// src\services\image.ts
// 通过id查询图片
export const imageGetByIdService = async (id: ImageInferSelect['id']) => {
  const image = await drizzleDb.query.images.findFirst({
    where: drizzleOrm.eq(drizzleSchema.images.id, id),
    with: {
      postsToImages: {
        with: {
          post: {
            with: {
              replies: true,
              postsToImages: true
            }
          }
        }
      }
    }
  })
  if (image == null) {
    throw new AppError('图片不存在', 400)
  }
  // 数据整理，处理查询数据以符合换为drizzle之前的数据
  return dataDQWimageGetByIdHandle(image)
}

// src\utils\data\db-query-with.ts
// 在 src\services\image.ts 被调用
// 通过id查询图片
export const dataDQWimageGetByIdHandle = (data: DQWimageGetById) => {
  // 整理posts
  const posts = data.postsToImages.map((pTIitem) => {
    const _count = {
      // 计数 images
      images: pTIitem.post.postsToImages.length,
      // 计数 replies
      replies: pTIitem.post.replies.length
    }
    return {
      ...pTIitem.post,
      _count,
      postsToImages: undefined,
      replies: undefined
    }
  })
  const _count = {
    // 计数 posts
    posts: posts.length
  }
  return {
    ...data,
    posts,
    _count,
    postsToImages: undefined
  }
}

// eslint-disable-next-line @typescript-eslint/consistent-type-imports
import type { drizzleDb, drizzleSchema } from '@/db'
// src\types\data\db-query-with.d.ts
// 为了推断类型而使用的临时函数
const fnDQWimageGetById = async () => {
  return await drizzleDb.query.images.findFirst({
    where: drizzleOrm.eq(drizzleSchema.images.id, id),
    with: {
      postsToImages: {
        with: {
          post: {
            with: {
              replies: true,
              postsToImages: true
            }
          }
        }
      }
    }
  })
}
// 通过id查询图片数据库查询的类型
export type DQWimageGetById = NonNullable<PromiseReturnType<typeof fnDQWimageGetById>>
```

### 250125-查询关系
```ts
// src\services\image.ts
// 查询关系
export const imageDeleteAllService = async () => {
  // // drizzle好像不支持直接查询“没有帖子的图片”，需要自己过滤
  // const images = (await drizzleDb.query.images.findMany({
  //   with: {
  //     postsToImages: true
  //   }
  // })).filter(i => i.postsToImages.length === 0)
  // 好像是可以的
  const images = await drizzleDb.query.images.findMany({
    where: drizzleOrm.notExists(
      drizzleDb.select().from(drizzleSchema.postsToImages)
        .where(drizzleOrm.eq(
          drizzleSchema.postsToImages.imageId,
          drizzleSchema.images.id
        ))
    ),
    columns: {
      id: true
    }
  })

  const imgDelPromises = images.map(async (img) => {
    return await deleteImageByIdWhereNonePost(img.id).catch(() => null)
  })

  return await Promise.all(imgDelPromises)
}

```

### 250125-更新
```ts
// src\services\image.ts
// 更新
export const imageUpdateService = async (
  imageInfo: ImageUpdateJsonType
) => {
  // 查找图片
  const targetImage = await baseFindImageById(imageInfo.id)
  if (targetImage == null) {
    throw new AppError('图片不存在')
  }
  // 修改信息
  const updatedImage = await drizzleDb.update(drizzleSchema.images)
    .set({
      alt: imageInfo.alt,
      createdAt: imageInfo.createdAt
    })
    .where(drizzleOrm.eq(drizzleSchema.images.id, targetImage.id))
    .returning()
  return updatedImage
}

// src\services\image.ts
// 删除原图
export const imageDeleteOriginalService = async (id: ImageInferSelect['id']) => {
  // get info before update
  // 在更新前查询图片
  const image = await baseFindImageById(id)
  if (image == null) {
    throw new AppError('imageId不存在')
  }
  // first update database
  // 首先更新数据库
  const updatedImage = await drizzleDb.update(drizzleSchema.images)
    .set({
      originalSize: 0,
      originalPath: null
    })
    .where(drizzleOrm.eq(drizzleSchema.images.id, image.id))
    .returning()
  // delete file
  // 然后再在文件中删除
  imageSystem.deleteOriginalImage(image.originalPath)
  return updatedImage
}

// src\services\image.ts
// 删除全部原图
export const imageDeleteAllOriginalService = async () => {
  // update database
  // 更新数据库
  const imgOriginalDelList = await drizzleDb.update(drizzleSchema.images)
    .set({
      originalSize: 0,
      originalPath: null
    })
    .where(drizzleOrm.not(
      drizzleOrm.eq(drizzleSchema.images.originalSize, 0)
    ))
    .returning()
    .catch((error) => {
      logUtil.info({
        title: '删除全部原图 | 数据库更新失败',
        content: String(error)
      })
      throw new AppError('数据库更新失败')
    })
  // del file
  // 删除文件
  await imageSystem.deleteAllOriginalImage().catch((error) => {
    logUtil.info({
      title: '删除全部原图 | 图片删除失败',
      content: String(error)
    })
    throw new AppError('图片删除失败')
  })
  return imgOriginalDelList.length
}
```

### 250125-添加
```ts
// src\services\image.ts
// 图片添加接口
export const imageSendService = async (
  imageFile: File | Blob,
  imageInfo?: Omit<ImageUpdateJsonType, 'id'>
) => {
  // 处理图片
  const {
    path,
    originalPath,
    smallSize,
    largeSize,
    originalSize
  } = await imageSystem.processImage(imageFile).catch((error) => {
    logUtil.info({
      title: '图片添加失败 | 图片处理失败',
      content: String(error)
    })
    throw new AppError('图片处理失败')
  })

  const newDate = new Date()

  // 数据库添加图片
  const addedImage = await drizzleDb
    .insert(drizzleSchema.images)
    .values({
      path,
      originalPath,
      smallSize,
      largeSize,
      originalSize,
      alt: imageInfo?.alt,
      // 时间之类最好手动指定，因为默认值只以秒为精确度
      createdAt: imageInfo?.createdAt ?? newDate,
      addedAt: newDate,
      updatedAt: newDate
    })
    .returning()
  return addedImage
}
```

### 250124-删除
```ts
// src\services\base.ts
// 尝试删除图片
export const deleteImageByIdWhereNonePost = async (id: ImageInferSelect['id']) => {
  // 查找图片
  const img = await drizzleDb.query.images.findFirst({
    where: drizzleOrm.eq(drizzleSchema.images.id, id),
    with: {
      postsToImages: true
    }
  })
  if (img == null) {
    throw new AppError('图片不存在', 400)
  }

  // 确认其未被使用
  if (img.postsToImages.length !== 0) {
    throw new AppError('图片使用中', 400)
  }

  // 在表中删除
  await drizzleDb.delete(drizzleSchema.images)
    .where(drizzleOrm.eq(drizzleSchema.images.id, img.id))

  // 在文件中删除
  imageSystem.deleteImage(img.path, img.originalPath)
  return img
}
```

### 250124-批量删除
https://orm.drizzle.team/docs/delete
```ts
// src\services\admin.ts
// 日志清理
export const adminLogDeleteService = async (num: number) => {
  // 查询前 num 条记录
  const topNumLogs = await (async () => {
    const selectOrderBy = dbLogsOrderBy
    const logsSelect = await drizzleDb
      .select()
      .from(drizzleSchema.logs)
      .orderBy(...selectOrderBy)
      .limit(num)
    return logsSelect
  })().catch((error) => {
    logUtil.info({
      title: '日志查询失败',
      content: String(error)
    })
    throw new AppError('日志查询失败')
  })

  // 删除不在前 num 条记录中的所有其他记录
  const deletedLogs = await (async () => {
    const deleteWhere = drizzleOrm.notInArray(
      drizzleSchema.logs.id, topNumLogs.map(i => i.id)
    )
    const logsDelete = await drizzleDb
      .delete(drizzleSchema.logs)
      .where(deleteWhere)
      .returning()
    return logsDelete
  })().catch((error) => {
    logUtil.info({
      title: '日志删除失败',
      content: String(error)
    })
    throw new AppError('日志删除失败')
  })

  return {
    count: deletedLogs.length
  }
}
```

### 250124-非顺序主键游标分页查询
https://orm.drizzle.team/docs/guides/cursor-based-pagination
```ts
// src\services\admin.ts
// 日志分页查询
export const adminLogGetByCursorService = async (
  cursorId: AdminLogGetByCursorParamType['id'], query: AdminLogGetByCursorQueryType
) => {
  // 在正式查询之前，首先要查询游标对应的log
  const cursorData = await (async () => {
    if (cursorId == null) {
      return undefined
    }
    const logsQuery = await drizzleDb.query.logs.findFirst({
      where: drizzleOrm.eq(drizzleSchema.logs.id, cursorId)
    })
    if (logsQuery == null) {
      throw new AppError('日志获取失败 | 游标无效')
    }
    return logsQuery
  })()

  // where类型查询
  const selectWhereType = (() => {
    const typeList: LogTypeMapValues[] = []
    if (query.error !== 'false') {
      typeList.push(logTypeMap.error.key)
    }
    if (query.warning !== 'false') {
      typeList.push(logTypeMap.warning.key)
    }
    if (query.success !== 'false') {
      typeList.push(logTypeMap.success.key)
    }
    if (query.info !== 'false') {
      typeList.push(logTypeMap.info.key)
    }
    return drizzleOrm.inArray(drizzleSchema.logs.type, typeList)
  })()

  // where使其从游标开始查询
  const selectWhereCursor = (() => {
    if (cursorData == null) {
      return undefined
    }
    return drizzleOrm.or(
      // 查询createdAt比游标所指的小的，因为是createdAt降序排序
      drizzleOrm.lt(drizzleSchema.logs.createdAt, cursorData.createdAt),
      // 需要考虑到createdAt相同的情况，所以需要借助id来确认createdAt相同时的顺序
      // 借助id升序降序随便，但要与orderBy相同
      drizzleOrm.and(
        drizzleOrm.eq(drizzleSchema.logs.createdAt, cursorData.createdAt),
        drizzleOrm.gt(drizzleSchema.logs.id, cursorData.id)
      )
    )
  })()

  const selectWhere = drizzleOrm.and(selectWhereType, selectWhereCursor)

  // orderBy
  const selectOrderBy = dbLogsOrderBy

  // selectLimit 分页个数
  const selectLimit = logConfig.logCursorTakeNum

  // select方式的查询，功能更多
  // const logsList = await drizzleDb
  //   .select()
  //   .from(drizzleSchema.logs)
  //   .where(selectWhere)
  //   .orderBy(...selectOrderBy)
  //   .limit(selectLimit)
  //   .catch((error) => {
  //     logUtil.info({
  //       title: '日志获取失败',
  //       content: String(error)
  //     })
  //     throw new AppError('日志获取失败')
  //   })
  // query方式的查询，关系查询方便
  const logsList = await drizzleDb.query.logs.findMany({
    where: selectWhere,
    orderBy: selectOrderBy,
    limit: selectLimit
  }).catch((error) => {
    logUtil.info({
      title: '日志获取失败',
      content: String(error)
    })
    throw new AppError('日志获取失败')
  })
  return logsList
}

// 对日志查询的排序，要复用
// createdAt降序 id升序
const dbLogsOrderBy = [
  drizzleOrm.desc(drizzleSchema.logs.createdAt),
  drizzleOrm.asc(drizzleSchema.logs.id)
]
```