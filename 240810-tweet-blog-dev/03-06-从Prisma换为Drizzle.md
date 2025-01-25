将web版和桌面版简单同步一下
```
在 tweet-blog-hono 新建分支 refactor/web-desktop-integration

将 tweblog-electron-hono/src 复制到 tweet-blog-hono/src

然后按照git的更改提示，将不需要的更改撤回，检查检查已更改的内容，一个个暂存

切换至 main 合并分支
```

## 开始换为Drizzle
```
在 tweet-blog-hono 新建分支 refactor/replace-prisma-with-drizzle

pnpm remove prisma @prisma/client

prisma 文件夹要准备移除，记得dockerfile也要改
```

官方文档 https://orm.drizzle.team/docs/get-started/sqlite-new
```sh
# 官方默认教程是用 libsql，但自己还是用 better-sqlite3 更保险
pnpm add drizzle-orm better-sqlite3
pnpm add -D drizzle-kit @types/better-sqlite3
```

### 初始化
一点熟悉drizzle的自言自语
```
drizzle-kit CLI 进行迁移需要依赖 drizzle.config.ts
而在代码中进行的 await migrate(db); 是与 drizzle.config.ts 无关的吗
也就是说在运行时是不需要 drizzle.config.ts 的对吗。对

drizzle中所有在运行时的操作都是不需要drizzle.config.ts的对吗，也就是说最终生产环境移除drizzle.config.ts也可以吗。对

现在有思路了，可以不在drizzle.config.ts写数据库路径，不使用drizzle-kit migrate，但使用drizzle-kit generate是没问题的。
在schema更改后使用 drizzle-kit generate，然后再代码中每次启动时进行 migrate(db)
```

在换为Drizzle的同时，做出优化
```
为posts添加updatedAt字段
为images添加createdAt与updatedAt字段

图片的话，或许应该修改前端处理逻辑

Drizzle好像不支持隐式多对多，这样的话返回的数据结构就不一样了，还是在前端改一改吧

Drizzle好像不支持api式的写入，看来要大改了

const databaseFile = 'database-1_0_0.sqlite'
使用这样的数据库命名，当有了不向前兼容的更新后，更改版本号即可实现使用新数据库
其中的版本即代表最早为基础一直迁移至今的数据库

```

Drizzle数据模型 `src\db\schema.ts`
```
schema 修改后，执行 

测试使用
pnpm drizzle-kit push

正规迁移
pnpm drizzle-kit generate
pnpm drizzle-kit generate --name=init-202501241834
pnpm drizzle-kit migrate
```

```ts
// drizzle-kit 要依赖 drizzle.config.ts
import { defineConfig } from 'drizzle-kit'
import { databaseConfig } from './src/configs'

export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dbCredentials: {
    url: `file:${databaseConfig.databasePath}`
  },
  dialect: 'sqlite'
})
```

### 更改查询代码
```
搜索 prisma

日志分页查询 adminLogGetByCursorService

```
