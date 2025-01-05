
## 测试
将两个前端打包，放在后端的 /static 目录
```
先试着检查一下类型
全面的类型检查应该使用 TypeScript 编译器 `tsc`
tsc --noEmit

在 Vue 项目中，scripts中有"type-check"
pnpm type-check

打包
pnpm build
```

后端也先试着编译运行一下
```
pnpm build
node dist/index.js

失败了，好像是因为 `node-fetch` 是 ES 模块，而自己需要使用 CommonJS 模块，应该降级
pnpm install node-fetch@2
pnpm install @types/node-fetch@2.6.12 --save-dev

再次 node dist/index.js 又出现了问题，forward系统初始化失败
排查过后发现是自动生成的 typesForwardStoreSchema 有问题，打印出来是undefined，
相比之下 typesAdminStoreSchema 打印出来则是正常的，看来是自动生成还有问题，
奇怪的是 pnpm dev 时从未出现这个问题
草，破案了，是因为自己 pnpm build 前没有删除 dist 目录，其残留文件造成了影响
具体因为自己之前将 src\schemas\types.ts 重构为了目录，而dist未清理导致是生效的还是之前 types.ts 所生成的 types.js 文件，以后 pnpm build 前一定要删除 dist

好了，现在 node dist/index.js 启动成功了，来测试推文导入和转发
好的，转发没问题，导入并处理图片也没问题。完全没问题了，可以开始docker打包了
```


## docker打包

### 改改dockerfile
增加了声明数据卷 `VOLUME ["/app/data"]`
```Dockerfile
FROM node:20.12.2-alpine3.19 AS base

# 构建阶段
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

# 复制文件 tsconfig.json 和 entrypoint.sh
COPY tsconfig.json entrypoint.sh ./
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

# 运行阶段
FROM base AS runner
WORKDIR /app

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

# 安装pnpm
RUN npm install -g pnpm && \
    npm cache clean --force

# 取消代理
ENV http_proxy=
ENV https_proxy=

# 复制文件
COPY --from=builder /app/entrypoint.sh /app/entrypoint.sh
COPY --from=builder /app/package.json /app/package.json
COPY --from=builder /app/node_modules /app/node_modules
COPY --from=builder /app/dist /app/dist
COPY --from=builder /app/prisma /app/prisma
COPY --from=builder /app/static /app/static

# 声明数据卷
VOLUME ["/app/data"]

# 设置端口
ENV TWEET_BLOG_HONO_PORT=51125
EXPOSE 51125

ENTRYPOINT ["sh", "entrypoint.sh"]
```

### 开始打包
准备
```
在自己的ubuntu上新建目录 `/root/tweblog-docker-build`，除了 node_modules 外，全部上传
```

设置代理，以免镜像下载失败
```
# 测试
curl www.google.com
curl x.com
curl qq.com

# 这是在ubuntu中
export http_proxy=http://192.168.2.110:10811/
export https_proxy=http://192.168.2.110:10811/

# 这是在Dockerfile
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/
```

#### 构建1【失败】
```
docker build -t tweblog-build-241228:0.0.0 .
```

OK 查看镜像，还行
```
docker images
REPOSITORY             TAG       IMAGE ID       CREATED         SIZE
tweblog-build-241228   0.0.0     e0890eff29a4   3 minutes ago   281MB
```

运行
```
docker run -d \
	--name tweblog-run-241228-131800 \
	-v /root/tweblog-run-241228-131800/data:/app/data \
	-p 51125:51125 \
	tweblog-build-241228:0.0.0

# 查看日志
docker logs tweblog-run-241228-131800
# 进入容器
docker exec -it tweblog-run-241228-131800 /bin/sh
```

哦哦，忘了在运行阶段复制 entrypoint.sh 了
```
root@dd-ubuntu:~/tweblog-docker-build# docker logs tweblog-run-241228-131800
sh: can't open 'entrypoint.sh': No such file or directory
```

#### 构建2【成功】
```
docker build -t tweblog-build-241228-132538:0.0.0 .

docker run -d \
	--name tweblog-run-241228-132538 \
	-v /root/tweblog-run-241228-132538/data:/app/data \
	-p 51125:51125 \
	tweblog-build-241228-132538:0.0.0

# 查看日志
docker logs tweblog-run-241228-132538
# 进入容器
docker exec -it tweblog-run-241228-132538 /bin/sh
```

OK
```
测试推文导入 忘了配置代理了，先停一下
docker stop tweblog-run-241228-132538
docker start tweblog-run-241228-132538
OK 推文导入没问题
```

### 尝试导出与完善
```
REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
tweblog-build-241228-132538   0.0.0     82bb1a5889cd   2 hours ago     281MB

# 导出镜像
docker save -o tweblog-build-241228-132538.tar tweblog-build-241228-132538:0.0.0

# 尝试在自己的服务器里运行
# 将文件下载到当前目录 自动处理重定向
curl -L -O https://sakiko.top/d/onedrive/Sakiko/Project/Tweblog/20241228/tweblog-build-241228-132538.tar

# 加载镜像
docker load -i tweblog-build-241228-132538.tar

# 启动容器
docker run -d \
	--name tweblog-run-241228-132538 \
	-v /root/tweblog-run-241228-132538/data:/app/data \
	-p 51125:51125 \
	--restart unless-stopped \
	tweblog-build-241228-132538:0.0.0

# 查看日志
docker logs tweblog-run-241228-132538
# 进入容器
docker exec -it tweblog-run-241228-132538 /bin/sh
```

完善运行命令
```
mkdir -p ${HOME}/Tweblog/data

cd ${HOME}/Tweblog

docker run -d \
	--name Tweblog \
	-v ${HOME}/Tweblog/data:/app/data \
	-p 51125:51125 \
	--restart unless-stopped \
	tweblog:0.0.0

```

好了现在可以开始试着写vitepress文档了
[241228-vitepress](笔记241207/241228-vitepress.md)

### 正式的0.0.1版本打包
项目文档完善后的正式的0.0.1版本打包
```
后端测试
tsc --noEmit
pnpm build
node dist/index.js

两个前端测试，打包
tsc --noEmit
pnpm type-check

前端确保配置正确，打包
pnpm build

前后端提交

在ubuntu中创建 /root/tweblog-build-250104
拷贝所需文件（主要排除 node_modules .git）

su
```

刚才试了一下，好像对于运行run时的代理要在 [【配置Docker使用代理服务器】250103-ubuntu安装与配置](笔记241207/250103-ubuntu安装与配置.md#配置Docker使用代理服务器) `daemon.json`配置。而build时的代理还是要通过以下方式配置？
```sh
export http_proxy=http://192.168.2.110:10811/
export https_proxy=http://192.168.2.110:10811/
```

```sh
# 构建
docker build -t tweblog:0.0.1 .

# 运行
docker run -d \
	--name tweblog-run-250104104251 \
	-v ${HOME}/tweblog-run-250104104251/data:/app/data \
	-p 51125:51125 \
	--restart unless-stopped \
	tweblog:0.0.1

# 查看日志
docker logs tweblog-run-250104104251
# 进入容器
docker exec -it tweblog-run-250104104251 /bin/sh

# 查看镜像
docker images
```

打包完成
```
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
tweblog       0.0.1     efabd98bdf16   23 minutes ago   281MB
```


#### DockerHub
```
docker login

docker tag tweblog:0.0.1 harukiowo/tweblog:0.0.1

docker push harukiowo/tweblog:0.0.1

docker tag harukiowo/tweblog:0.0.1 harukiowo/tweblog:latest

docker push harukiowo/tweblog:latest

docker images
REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
harukiowo/tweblog   0.0.1     efabd98bdf16   3 hours ago     281MB
harukiowo/tweblog   latest    efabd98bdf16   3 hours ago     281MB
tweblog             0.0.1     efabd98bdf16   3 hours ago     281MB
```

#### 完善
基本完成，给前后端打标签推送
```
0.0.1
正式的0.0.1版本 https://hub.docker.com/r/harukiowo/tweblog
```

再导出一下
```
docker save -o tweblog-0_0_1.tar harukiowo/tweblog:0.0.1
```

完善文档
```
部署文档

先清空
cd /www/wwwroot/tweblog.com
rm -rf *
```

#### 试着测试一下
在vps部署，先把旧的删一下
```
docker ps
docker stop tweblog-run-241228-132538

清理所有未使用的数据，包括构建缓存、未使用的镜像、容器、卷和网络
docker system prune -a --volumes

rm -rf tweblog-run-241228-132538
```

测试
```
docker run -d \
	--name Tweblog \
	-v ${HOME}/Tweblog/data:/app/data \
	-p 51125:51125 \
	--restart unless-stopped \
	harukiowo/tweblog:0.0.1

docker logs Tweblog
```
ok没问题


