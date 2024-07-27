Tweetcord这个好像也可以实现推文同步功能
- https://home.gamer.com.tw/artwork.php?sn=5808333
- https://github.com/Yuuzi261/Tweetcord
- https://www.youtube.com/watch?v=4JerLQkqPSo&t=5s


创建应用

## 机器人设置
查看并保存token
> You are being rate limited.，嘶，没办法生成token，明天再看吧
> 好耶，拿到token了

设置
![](assets/Pasted%20image%2020240726180208.png)
![](assets/Pasted%20image%2020240726180248.png)
![](assets/Pasted%20image%2020240726180516.png)

复制底部链接，加入机器人


## 应用试运行

克隆项目至本地 https://github.com/Yuuzi261/Tweetcord.git

切换至分支至标签 `0.4-hotfix`

安裝必要的模組。
```
pip install -r requirements.txt
```

創建並配置.env文件
```
BOT_TOKEN=YourDiscordBotToken
TWITTER_TOKEN=YourTwitterAccountAuthToken
DATA_PATH=./data/
```
在twitter的cookies中获取auth_token

修改 `configs.yml`
所有與時間相關的配置都以秒為單位。
```yml
prefix: '.'
activity_name: 'Sakiko'
tweets_check_period: 100
tweets_updater_retry_delay: 3000
tasks_monitor_check_period: 60
tasks_monitor_log_period: 14400
auto_turn_off_notification: true
auto_unfollow: true
```

运行
```
python bot.py
```
好像失败了，换到vps上试试吧

## 在vps中试运行
上传至`/root/Tweetcord`

安裝必要的模組。
```
pip3 install -r requirements.txt
```

运行
```
python3 /root/Tweetcord/bot.py
```

失败了，可能py版本旧，老老实实docker打包吧

