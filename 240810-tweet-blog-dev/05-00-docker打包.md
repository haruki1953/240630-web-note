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