## 准备

第一次尝试打包
- 前端 tweet-blog-vue3 版本 v0.0.0
- 后端 tweet-blog-hono 版本 v0.0.0

将前端打包，放在后端的 /static 目录

配置
```
package.json
{
  // ...
  "scripts": {
    // ...
    "build": "tsc"
  },
}

tsconfig.json
{
  "compilerOptions": {
    // ...
    "outDir": "./dist",
  },
  "exclude": ["node_modules"],
}

# 装过了
pnpm install typescript --save-dev
```

自己尝试tsc编译后运行，出了问题
```
在导入时不能省略index
import { httpPort } from './configs'
import { httpPort } from './configs/index'
```

解决方法是让ts编译为CommonJS（tsconfig.json）
```
    "target": "ESNext",
    "module": "CommonJS",
    "moduleResolution": "node",
    "esModuleInterop": true, // 使得 TS 能兼容 ES6 和 CommonJS 的导入导出方式
```

然后编译后的js不能识别路径别名，通过tsc-alias包解决
```
pnpm install tsc-alias --save-dev

"build": "tsc && tsc-alias"
```

关于默认端口，决定用 51125
```
算法：(年份/5 + 1) * 10000 + 日期
24年11月25日即 51125
```


## docker打包
是用家里另一台电脑ubuntu来跑docker

设置代理，以免镜像下载失败
```
# 测试
curl x.com
curl qq.com

# 这是在ubuntu中
export http_proxy=http://192.168.2.110:10811/
export https_proxy=http://192.168.2.110:10811/

# 这是在Dockerfile
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/
```

### 单阶段构建尝试
先来试着弄一下
```dockerfile
FROM node:20.12.2-alpine3.19 AS base

FROM base AS builder
WORKDIR /app

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 安装 gcompat 包以提供 GNU C Library (glibc) 的兼容性层
RUN apk add --no-cache gcompat

# 复制项目全部文件至工作目录 
# 注意：提前删除了/node_modules /data
COPY . .

# 安装 pnpm
RUN npm install -g pnpm

# 安装依赖
RUN pnpm install --frozen-lockfile

# 运行迁移 创建数据库表 生成 Prisma 客户端
RUN pnpm prisma migrate dev --name init

# 编译
RUN pnpm build

# 修剪依赖
RUN pnpm prune --prod

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

# 取消代理
ENV http_proxy=
ENV https_proxy=

CMD ["node", "dist/index.js"]
```

构建
```
docker build -t tweblog-test:0.0.0 .
```

运行
```sh
docker run -d \
	--name tweblog-test \
	-p 51125:51125 \
	tweblog-test:0.0.0
```

ok成功了
```
查看镜像 docker images
查看容器
	正在运行的 docker ps 
	所有容器 docker ps -a
进入容器 docker exec -it <container_id> /bin/sh

docker exec -it tweblog-test /bin/sh

给文件查看设置别名 alias ll='ls -la'
```

感觉有点大
```
tweblog-test       0.0.0     b4cbbb837912   27 minutes ago   449MB
```

### 多阶段构建

#### 尝试1 失败
```dockerfile
FROM node:20.12.2-alpine3.19 AS base

FROM base AS builder
WORKDIR /app

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 安装 gcompat 包以提供 GNU C Library (glibc) 的兼容性层
RUN apk add --no-cache gcompat

# 复制项目全部文件至工作目录 
# 注意：提前删除了/node_modules /data
COPY . .

# 安装 pnpm
RUN npm install -g pnpm

# 安装依赖
RUN pnpm install --frozen-lockfile

# 运行迁移 创建数据库表 生成 Prisma 客户端
RUN pnpm prisma migrate dev --name init

# 编译
RUN pnpm build

# 修剪依赖
RUN pnpm prune --prod

# 取消代理
ENV http_proxy=
ENV https_proxy=

FROM base AS runner
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 hono

COPY --from=builder --chown=hono:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=hono:nodejs /app/dist /app/dist
COPY --from=builder --chown=hono:nodejs /app/package.json /app/package.json
COPY --from=builder --chown=hono:nodejs /app/data /app/data
COPY --from=builder --chown=hono:nodejs /app/static /app/static
# COPY --from=builder --chown=hono:nodejs /app/node_modules/.pnpm/@prisma+client@5.18.0_prisma@5.18.0/node_modules/.prisma/client /app/node_modules/.pnpm/@prisma+client@5.18.0_prisma@5.18.0/node_modules/.prisma/client

USER hono

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

CMD ["node", "dist/index.js"]
```

构建
```
docker build -t tweblog-test:0.0.1 .
```

运行
```sh
docker stop tweblog-test
docker rm tweblog-test

docker run -d \
	--name tweblog-test-1 \
	-p 51126:51125 \
	tweblog-test:0.0.1
```

有问题了，获取推文时弹出了错误，修改后再尝试

#### 尝试2 成功
```dockerfile
FROM node:20.12.2-alpine3.19 AS base

FROM base AS builder
WORKDIR /app

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 复制项目全部文件至工作目录 
# 注意：提前删除了/node_modules /data
COPY . .

# 安装 pnpm、安装依赖、运行迁移、编译和修剪依赖
RUN npm install -g pnpm && \
    pnpm install --frozen-lockfile && \
    pnpm prisma migrate dev --name init && \
    pnpm build && \
    pnpm prune --prod

# 取消代理
ENV http_proxy=
ENV https_proxy=

FROM base AS runner
WORKDIR /app

COPY --from=builder /app /app

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

CMD ["node", "dist/index.js"]
```

构建
```
docker build -t tweblog-test:0.0.2 .
```

运行
```sh
docker run -d \
	--name tweblog-test-2 \
	-p 51127:51125 \
	tweblog-test:0.0.2

docker exec -it tweblog-test-2 /bin/sh
```

哦，这次好了，是因为权限相关的问题吗

#### 尝试3 失败
再试验一次
```dockerfile
FROM node:20.12.2-alpine3.19 AS base

FROM base AS builder
WORKDIR /app

#COPY package.json pnpm-lock.yaml tsconfig.json src prisma static  ./
# 复制项目全部文件至工作目录 
# 注意：提前删除了/node_modules /data
COPY . .

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 安装 pnpm、安装依赖、运行迁移、编译和修剪依赖
RUN npm install -g pnpm && \
    pnpm install --frozen-lockfile && \
    pnpm prisma migrate dev --name init && \
    pnpm build && \
    pnpm prune --prod

# 取消代理
ENV http_proxy=
ENV https_proxy=

FROM base AS runner
WORKDIR /app

COPY --from=builder /app/node_modules /app/node_modules
COPY --from=builder /app/package.json /app/package.json
COPY --from=builder /app/dist /app/dist
COPY --from=builder /app/data /app/data
COPY --from=builder /app/static /app/static

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

CMD ["node", "dist/index.js"]
```

构建
```
docker build -t tweblog-test:0.0.3 .
```

运行
```sh
docker run -d \
	--name tweblog-test-3 \
	-p 51128:51125 \
	tweblog-test:0.0.3

docker exec -it tweblog-test-3 /bin/sh
```

获取推文时又弹出了错误？现在看来可能时有文件忘了拷贝？prisma是运行时必须的？

#### 尝试4 成功
又加上了权限，且复制了prisma文件夹
```dockerfile
FROM node:20.12.2-alpine3.19 AS base

FROM base AS builder
WORKDIR /app

# 复制项目全部文件至工作目录 
# 注意：提前删除了/node_modules /data
COPY . .

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 安装 pnpm、安装依赖、运行迁移、编译和修剪依赖
RUN npm install -g pnpm && \
    pnpm install --frozen-lockfile && \
    pnpm prisma migrate dev --name init && \
    pnpm build && \
    pnpm prune --prod

# 取消代理
ENV http_proxy=
ENV https_proxy=

FROM base AS runner
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 hono

COPY --from=builder --chown=hono:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=hono:nodejs /app/package.json /app/package.json
COPY --from=builder --chown=hono:nodejs /app/dist /app/dist
COPY --from=builder --chown=hono:nodejs /app/data /app/data
COPY --from=builder --chown=hono:nodejs /app/static /app/static
COPY --from=builder --chown=hono:nodejs /app/prisma /app/prisma

USER hono

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

CMD ["node", "dist/index.js"]
```

构建
```
docker build -t tweblog-test:0.0.4 .
```

运行
```sh
docker run -d \
	--name tweblog-test-4 \
	-p 51129:51125 \
	tweblog-test:0.0.4

docker exec -it tweblog-test-4 /bin/sh
```

成功，看来prisma文件夹在运行时是必须的


### 数据卷试验
```sh
docker run -d \
	--name tweblog-test-4-v \
	-v /root/tweblog-test-4-v/data:/app/data \
	-p 51130:51125 \
	tweblog-test:0.0.4

docker exec -it tweblog-test-4-v /bin/sh
```

容器启动失败，好像是创建json文件失败，是权限问题，再试试
```sh
docker run -d \
	--name tweblog-test-2-v \
	-v /root/tweblog-test-2-v/data:/app/data \
	-p 51131:51125 \
	tweblog-test:0.0.2

docker exec -it tweblog-test-2-v /bin/sh
```
这次启动成功了，看来还是不设置user比较好

设置数据卷后，数据库果然出了问题，获取推文会弹出错误
```
因为数据库文件 sqlite.db 在构建时 pnpm prisma migrate dev 被创建
当运行时绑定数据卷，好像就会将其覆盖

其他的json文件倒是没问题，因为是在运行时生成的
```


#### prisma
```
# 安装 pnpm、安装依赖、生成数据库、编译和修剪依赖
RUN npm install -g pnpm && \
    pnpm install --frozen-lockfile && \
    pnpm prisma migrate dev --name init && \
    pnpm build && \
    pnpm prune --prod

自己的做法是错的，在构建时应该只生成Prisma Client
pnpm prisma generate

然后再运行时进行迁移（不存在时生成数据库文件）
pnpm prisma migrate deploy

而且pnpm以及依赖的安装应该独立处理，以助于复用
# 生成PrismaClient、编译、修剪依赖、清理缓存
RUN pnpm prisma generate && \
    pnpm build && \
    pnpm prune --prod && \
    pnpm store prune

可以写一个启动脚本，其中包含迁移和项目的运行 entrypoint.sh
在运行前进行迁移，可以避免更新后数据库不同步的问题，也可以解决前面数据卷的问题
#!/bin/sh
pnpm prisma migrate deploy
node dist/index.js
```

entrypoint.sh中还应该加上判断迁移是否成功
```sh
#!/bin/sh
# Startup script for Prisma application
# This script runs database migrations and starts the application

echo "Starting database migrations..."
pnpm prisma migrate deploy

if [ $? -ne 0 ]; then
  echo "Error: Database migration failed. Aborting application startup."
  exit 1
fi

echo "Database migrations completed successfully"

echo "Starting application..."
node dist/index.js
```

#### 尝试1 运行失败
```dockerfile
FROM node:20.12.2-alpine3.19 AS base

FROM base AS builder
WORKDIR /app

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 复制安装依赖文件
COPY package.json pnpm-lock.yaml ./

# 安装 pnpm、安装依赖
RUN npm install -g pnpm && \
    pnpm install --frozen-lockfile

# 复制文件 tsconfig.json 和 start.sh
COPY tsconfig.json start.sh ./
# 复制目录 src、prisma 和 static
COPY ./src ./src
COPY ./prisma ./prisma
COPY ./static ./static

# 生成PrismaClient、编译、修剪依赖、清理缓存
RUN pnpm prisma generate && \
    pnpm build && \
    pnpm prune --prod && \
    pnpm store prune && \
    npm cache clean --force

# 取消代理
ENV http_proxy=
ENV https_proxy=

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

ENTRYPOINT ["sh", "entrypoint.sh"]
```

构建
```
docker build -t tweblog-test-prisma:0.0.1 .
```

运行
```sh
docker run -d \
	--name tweblog-test-prisma-v-1 \
	-v /root/tweblog-test-prisma-v-1/data:/app/data \
	-p 51132:51125 \
	tweblog-test-prisma:0.0.1

docker logs tweblog-test-prisma-v-1

docker exec -it tweblog-test-prisma-v-1 /bin/sh
```

运行失败
```
Starting database migrations...
undefined
 ERR_PNPM_RECURSIVE_EXEC_FIRST_FAIL  Command "prisma" not found
Error: Database migration failed. Aborting application startup.
```

#### 尝试2 运行成功
将prisma作为生产依赖而不是开发依赖
```
pnpm uninstall prisma
pnpm install prisma@5.18.0
```

重新复制文件

构建
```
docker build -t tweblog-test-prisma:0.0.2 .
```

运行
```sh
docker run -d \
	--name tweblog-test-prisma-v-2 \
	-v /root/tweblog-test-prisma-v-2/data:/app/data \
	-p 51133:51125 \
	tweblog-test-prisma:0.0.2

docker logs tweblog-test-prisma-v-2

docker exec -it tweblog-test-prisma-v-2 /bin/sh
```

很成功，只是镜像有点大了，450MB

### 尝试3 多阶段构建
明天弄



## TODO
用户、权限相关