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

# 设置容器开机自启
docker update --restart=always tweetcord_sakiko
# 或（手动停止容器后不会开机自启）
docker update --restart=unless-stopped tweetcord_sakiko
```

discord机器人倒是好了，但好像不会同步twitter，明天再说吧

- 好像twitter token拿错了，是不是应该拿sakiko214的
- 配置configs.yml文件，还是尽量用预设的配置吧，等正常了再改

修改配置
```
docker exec -it tweetcord_sakiko /bin/sh

vi configs.yml
vi .env

exit

docker restart tweetcord_sakiko
```

```sh
docker rm -f tweetcord_sakiko

docker run -d \
--name tweetcord_sakiko \
-v /root/tweetcord_sakiko/data:/bot/data \
--restart=unless-stopped \
tweetcord_sakiko:0.4
```

## 测试-总结
再创建一个twitter号吧，好像是要必须保证在某个浏览器上登录账号，保证有推送通知，才能使其生效

哦现在好了，现在来详细 **总结：**
- 现在我有三个推特账号 `@harukiO_0` `@sakiko214` `@sakikoO_O`
	- 其中 `@harukiO_0` `@sakiko214` 登录在火狐
	- `@sakikoO_O` 登录在谷歌浏览器
- 或许必须使账号中有推文通知，tweetcord才会运作
	- 所以将 `@sakikoO_O` 登录在不同的浏览器（如果在一个浏览器可能不显示通知），并在浏览器开启推文推送通知
	- .env中的TWITTER_TOKEN设置为账号`@sakikoO_O`的
	- 现在在discord中使用指令添加 `@sakiko214`，再使用其发帖，`@sakikoO_O`（谷歌浏览器）有推文通知，tweetcord也正常运作
- 虽然在测试时将 `tweets` 更新相关的时间设置的很短，但在部署时还是长一点好

discord的包不想再重新打了，现在梳理一下 **部署流程：**
1. 运行容器
```sh
docker run -d \
    --name tweetcord_sakiko \
    -v /root/tweetcord_sakiko/data:/bot/data \
    --restart=unless-stopped \
    tweetcord_sakiko:0.4
```

2. 修改配置
```sh
# 进入容器
docker exec -it tweetcord_sakiko /bin/sh
vi configs.yml
vi .env
```
configs.yml
```yml
prefix: '.'
activity_name: '春日影'
tweets_check_period: 100
tweets_updater_retry_delay: 3000
tasks_monitor_check_period: 60
tasks_monitor_log_period: 14400
auto_turn_off_notification: true
auto_unfollow: true
```
.env
```
BOT_TOKEN=YourDiscordBotToken
TWITTER_TOKEN=sakikoO_OTwitterAccountAuthToken
DATA_PATH=./data/
```

```sh
# 退出容器
exit
# 重启容器
docker restart tweetcord_sakiko
# 查看日志
docker logs tweetcord_sakiko
```

参考
- https://github.com/Yuuzi261/Tweetcord
- https://www.youtube.com/watch?v=4JerLQkqPSo&t=5s
- https://home.gamer.com.tw/artwork.php?sn=5808333


