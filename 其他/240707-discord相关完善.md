## 手机推特重定向问题
自己的域名 discord.sakiko.top 会重定向至discord邀请链接，是用nginx配置的301重定向。

现在的问题是：在手机推特点开 discord.sakiko.top 时，会显示“其他应用打开”

解决方案：不返回301响应，而是返回静态页面，通过js重载页面转到discord邀请链接
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小祥の小窝</title>
</head>
<body>
    <script>
        window.location.href = 'https://discord.com/invite/EF7qGQQpxB';
    </script>
</body>
</html>
```


以下是可选的一些优化
```nginx
#nginx配置404为根路径，这样即使路径错误也没事
error_page 404 /;

#nginx配置重写，将所有路径重写为根路径
location / {
  rewrite ^ /index.html break;
}
#另一种，try_files指令
location / {
    try_files $uri $uri/ /index.html;
}
```


## TweetShift同步推特内容至Discord
TweetShift Bot：免费！自动！同步推特内容至Discord！
https://www.youtube.com/watch?v=I9MZZY06GIs

https://tweetshift.com/

将机器人加入服务器

在仪表盘配置 https://tweetshift.com/dashboard

好像不行。。。