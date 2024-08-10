https://sakiko.top/da-ti-zi

自己一般用的是电信，这几天发现在移动的热点下，好多节点都失效了（vless），试了试以前的vmess节点居然管用（但好像还是会丢包之类的），现在再试试其他协议的节点

试了 Trojan-WS-TLS 节点，节点刚创建的大概30分钟内是可行的，之后就不行了，状况也类似vless，好奇怪，是移动运营商对websocket有限制吗

VLESS-H2-TLS 试试吧，不行，好像nginx不支持h2

试试 VLESS-gRPC-TLS，好像也不行

看到了这个帖子 https://github.com/v2fly/v2ray-core/discussions/2036#discussioncomment-3835477
- 用的是233boy的脚本
- 半小时被阻断（和自己类似）
- 感觉是sniffing的配置上

> 这些封锁可能与翻墙软件客户端发出的[Clienthello指纹](https://tlsfingerprint.io/)相关

看了看帖子的分析，有点理解了，应该是移动的运行商可以识别由233boy自动脚本搭建的节点的特征了，自己手动改改配置或许有用

233boy自动脚本的配置存放路径为 `/etc/v2ray/conf`

好像也没什么用

```
整理一下现在的情况
现在用电信是完全没问题，而移动会经常出现“远程主机强迫关闭了一个现有的连接”（但也勉强能连上）
```
