将web版和桌面版简单同步一下
```
在 tweet-blog-hono 新建分支 refactor/web-desktop-integration

将 tweblog-electron-hono/src 复制到 tweet-blog-hono/src

然后按照git的更改提示，将不需要的更改撤回，检查检查已更改的内容，一个个暂存

切换至 main 合并分支
```

## 开始换为Drizzle
```sh
# 在 tweet-blog-hono 新建分支 refactor/replace-prisma-with-drizzle

pnpm remove prisma @prisma/client

# prisma 文件夹要准备移除，记得dockerfile也要改
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
搜索 prisma 来逐个更改
[250124-Drizzle查询示例](笔记241207/250124-Drizzle查询示例.md)


### Drizzle的ESLint插件
https://orm.drizzle.team/docs/eslint-plugin

#### 安装
```sh
pnpm add -D eslint-plugin-drizzle@0.2.3
pnpm add -D @typescript-eslint/parser@6.21.0

# 已安装
pnpm add -D @typescript-eslint/eslint-plugin@6.4.0 
```
![](assets/Pasted%20image%2020250128123609.png)

#### 配置
`.eslintrc.json`
```json
{
    "env": {
        "es2021": true,
        "node": true
    },
    "extends": [
        "standard-with-typescript",
        "plugin:drizzle/recommended"
    ],
    "parserOptions": {
        "ecmaVersion": "latest",
        "sourceType": "module",
        "project": "./tsconfig.json"
    },
    "plugins": [
        "@typescript-eslint",
        "drizzle"
    ],
    "rules": {
        "@typescript-eslint/explicit-function-return-type": "off"
    }
}
```
![](assets/Pasted%20image%2020250128123300.png)

#### 示例
现在就有效果了，比如 `drizzleDb.delete` 如果没有where，就会报错
![](assets/Pasted%20image%2020250128124036.png)
![](assets/Pasted%20image%2020250128124131.png)
```
Without `.where(...)` you will delete all the rows in a table. If you didn't want to do it, please use `drizzleDb.delete(...).where(...)` instead. Otherwise you can ignore this rule hereeslint[drizzle/enforce-delete-with-where](https://github.com/drizzle-team/eslint-plugin-drizzle)
```


### 换为Drizzle之后的检查
```sh
# eslint 检查
pnpm lint
# 类型检查
pnpm tsc --noEmit
```

#### 事务错误
发现不管是手动rollback还是抛出错误，事务都无法回滚。好像事务函数不能是异步的，只能是同步的才支持回滚
https://github.com/drizzle-team/drizzle-orm/issues/1472
```
通过在查询后添加 .run() 或 .all() 可以让查询变成同步的
如果不需要返回值使用 .run() ，需要返回值使用 .all()
.returning()
.all()
```
[【250126-操作、关联、事务】250124-Drizzle查询示例](笔记241207/250124-Drizzle查询示例.md#250126-操作、关联、事务)

#### 设置索引
```
给 createdAt 和 外键设置索引
createdAt 设置为降序

pnpm drizzle-kit generate
```

#### 迁移记录合并
```
在 drizzle 文件夹中，删除自己想要合并的sql，然后还要在meta里修改，删除对应的snapshot，修改_journal，删除对应的项

然后再次 generate

pnpm drizzle-kit generate --name=test-202501291551

自己现在是全部删除了，重新初始化
pnpm drizzle-kit generate --name=init-202501291636
```

### 合并至主分支
```
refactor/replace-prisma-with-drizzle
master
```

### 同步至桌面版
```
Web版 tweet-blog-hono
桌面版 tweblog-electron-hono
后端版已经从 prisma 更换为了 drizzle ，现在给桌面版也同步更换一下

桌面版新建分支 refactor/drizzle-from-tbh

在桌面版删除
prisma
src

从Web版复制这些
drizzle
src
.eslintrc.json
.gitignore
drizzle.config.ts
tsconfig.json
```

然后要补齐所需的包
```sh
yarn remove prisma @prisma/client

yarn add drizzle-orm@0.38.4 better-sqlite3@11.8.1
yarn add -D drizzle-kit@0.30.2 @types/better-sqlite3@7.6.12 eslint-plugin-drizzle@0.2.3 @typescript-eslint/parser@6.21.0
```

接下来对比git的修改记录来修改，修改后进行检查
```sh
yarn lint
yarn tsc --noEmit
```

运行，修正了小问题，成功，开始尝试打包，OK
```sh
export http_proxy=http://127.0.0.1:10809/
export https_proxy=http://127.0.0.1:10809/

yarn make
```

切换至主分支，合并
