### todo-查询后对数据的处理
```ts
// src\services\image.ts
// 通过id查询图片
export const imageGetByIdService = async (id: ImageInferSelect['id']) => {
```

### 250124-查询关系
```ts
// src\services\image.ts
// 查询关系
export const imageDeleteAllService = async () => {
  // drizzle好像不支持直接查询“没有帖子的图片”，需要自己过滤
  const images = (await drizzleDb.query.images.findMany({
    with: {
      postsToImages: true
    }
  })).filter(i => i.postsToImages.length === 0)

  const imgDelPromises = images.map(async (img) => {
    return await deleteImageByIdWhereNonePost(img.id).catch(() => null)
  })

  return await Promise.all(imgDelPromises)
}
```

### 250124-更新
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

### 250124-添加
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
      // 时间之类最后手动指定，因为默认值只以秒为精确度
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
  const cursorLog = await (async () => {
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

  // where使其从游标开始查询
  const selectWhere = (() => {
    if (cursorLog == null) {
      return undefined
    }
    return drizzleOrm.or(
      // 查询createdAt比游标所指的小的，因为是createdAt降序排序
      drizzleOrm.lt(drizzleSchema.logs.createdAt, cursorLog.createdAt),
      // 需要考虑到createdAt相同的情况，所以需要借助id来确认createdAt相同时的顺序
      // 借助id升序降序随便，但要与orderBy相同
      drizzleOrm.and(
        drizzleOrm.eq(drizzleSchema.logs.createdAt, cursorLog.createdAt),
        drizzleOrm.gt(drizzleSchema.logs.id, cursorLog.id)
      )
    )
  })()

  // orderBy createdAt降序 id升序
  const selectOrderBy = [
    drizzleOrm.desc(drizzleSchema.logs.createdAt),
    drizzleOrm.asc(drizzleSchema.logs.id)
  ]

  // selectLimit 分页个数
  const selectLimit = logConfig.logCursorTakeNum

  // selectOffset 分页查询时，需要跳过一个
  const selectOffset = (() => {
    if (cursorLog == null) {
      return 0
    }
    return 1
  })()

  // select方式的查询，功能更多
  // const logsList = await drizzleDb
  //   .select()
  //   .from(drizzleSchema.logs)
  //   .where(selectWhere)
  //   .orderBy(...selectOrderBy)
  //   .limit(selectLimit)
  //   .offset(selectOffset)
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
    limit: selectLimit,
    offset: selectOffset
  }).catch((error) => {
    logUtil.info({
      title: '日志获取失败',
      content: String(error)
    })
    throw new AppError('日志获取失败')
  })
  return logsList
}
```