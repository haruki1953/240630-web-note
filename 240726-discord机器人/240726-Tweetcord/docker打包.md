设置代理
```
export http_proxy=http://192.168.2.110:10811/
export https_proxy=http://192.168.2.110:10811/

export HTTP_PROXY=http://192.168.2.110:10811/
export HTTPS_PROXY=http://192.168.2.110:10811/

curl https://x.com
```
> 注意：此时如果使用`sudo curl https://x.com`则好像不行，
> sudo可能会使代理失效，可以直接su获取root权限

给本地ubuntu设置root可直接登录[blog.csdn.net/txl910514/article/details/131225224](https://blog.csdn.net/txl910514/article/details/131225224)
	#ubuntu设置root直接登录
	vim教程[runoob.com/linux/linux-vim.html](https://www.runoob.com/linux/linux-vim.html)
```
sudo vim /etc/ssh/sshd_config
# 找到 `#Authentication`，将 `PermitRootLogin` 参数修改为 `yes`。
# 找到 `#Authentication`，将 `PasswordAuthentication` 参数修改为 yes
# 若 `sshd_config` 配置文件中无此配置项，则添加 `PasswordAuthentication yes` 项即可
PermitRootLogin yes
PasswordAuthentication yes

sudo service ssh restart
```

## 安装docker
- https://docs.docker.com/engine/install/ubuntu/

[安装使用 `apt` 存储库](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
> 注意：su获取root权限，使用代理，去掉sudo

指定版本安装
```sh
# 列出存储库中的可用版本
apt-cache madison docker-ce | awk '{ print $3 }'
# ...
# 5:24.0.7-1~ubuntu.20.04~focal

# 选择所需的版本并安装：
VERSION_STRING=5:24.0.7-1~ubuntu.20.04~focal
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

验证 Docker 引擎安装是否成功
```
sudo docker run hello-world
```
拉取时失败了，看来要专门配置代理

配置 Docker 使用代理服务器
```sh
sudo vim /etc/docker/daemon.json
```
```json
{
  "proxies": {
    "http-proxy": "http://192.168.2.110:10811",
    "https-proxy": "http://192.168.2.110:10811",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```
```sh
sudo systemctl restart docker
```

ok成功

## 打包
```Dockerfile
FROM python:3.10.9-alpine3.17
WORKDIR /bot
COPY requirements.txt /bot/

# 设置代理
ENV http_proxy=http://192.168.2.110:10811/
ENV https_proxy=http://192.168.2.110:10811/

RUN pip install -r requirements.txt

# 取消代理
ENV http_proxy=
ENV https_proxy=

COPY . /bot/
CMD python bot.py
```

```sh
# 构建
docker build -t tweetcord_sakiko:0.4 .

# 运行
docker run -d \
	--name tweetcord_sakiko \
	-v /root/tweetcord_sakiko/data:/bot/data \
	tweetcord_sakiko:0.4

# 查看日志
docker logs tweetcord_sakiko

# 导出
docker save -o tweetcord_sakiko-0.4.tar tweetcord_sakiko:0.4
```

## 部署
```sh
# 解压
docker load -i tweetcord_sakiko-0.4.tar

# 运行
docker run -d \
--name tweetcord_sakiko \
-v /root/tweetcord_sakiko/data:/bot/data \
tweetcord_sakiko:0.4

# 查看日志
docker logs tweetcord_sakiko

# 设置容器开机自启【TODO】
```

discord机器人倒是好了，但好像不会同步twitter，明天再说吧

