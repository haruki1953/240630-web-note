## TODO

### 完善接口与接口文档
后端给所有的响应进行类型约束验证，尽量使其可以使用以下类型组合
```ts
export interface Image {
  id: number
  alt: string | null
  path: string
  addedAt: string
  smallSize: number
  largeSize: number
  originalSize: number
  originalPath: string | null
  twitterLargeImageLink: string | null
}

export interface PostBase {
  id: number
  createdAt: string
  addedAt: string
  content: string
  isDeleted: boolean
  parentPostId: number | null
  twitterId: string | null
  twitterLink: string | null
  _count: {
    replies: number
  }
}

export interface Post extends PostBase {
  images: Image[]
}

export interface PostData extends Post {
  parentPost: PostBase | null
}
```

查看接口文档，有许多接口都要修改
- 帖子发送
- ……

接口与接口文档的响应状态码也要完善

解决vscode类型预览长度限制
