## 1.0.0
2025-01-31

### 🎉 Tweblog 1.0 发布啦
改进Web版的同时，使用 electron 制作了 Tweblog 的桌面版

✨ **新功能**
- 新增了 [桌面版](https://tweblog.com/guide/desktop)，拥有Web版的全部功能，可通过Web端口在手机等设备上远程控制

🧱 **改进**
- 得益于 Drizzle ，docker 镜像大小从 280M 减小到了 **199M**
- 关于页 markdown 支持直接编写 html
- 导航栏激活条件完善
- 推文转发页面中，添加了加载后的过渡
- 推文、图片、日志滚动时，底部的加载更多按钮进行了样式调整
- 图片无限滚动的初始值与增量进行了修改
- 图片预览按钮悬停时，添加了新的样式与过渡
- index.html 中添加了开放图谱元标签，现在在推特等平台分享时可以显示 Tweblog 的简介与图片了

🛠 **修复**
- 在关于页面使用 markdown 语法书写的链接，点击后不再是在当前页面打开，而是在新窗口打开
- 推文导入时，图片不再是按添加的顺序排序，而是根据推文的时间排序
- 前端的图标从 svg 换为了兼容性更好的 png 图标，并进行了放大

⚙ **重构**
- 将数据库框架从 Prisma 换为了 Drizzle

⚠ **注意**
- 1.0 版本和 0.0 版本的数据库不兼容，
	- 从 0.0 换为 1.0 版本后将使用新的数据库
	- 旧数据库路径 `data/sqlite.db`
	- 新数据库路径 `data/database-1_0_0.sqlite`
	- 暂时没有办法转移数据，请重新从推特等平台导入
	- 正常情况下 1.0 版本之后的更新都会兼容旧的数据库



## 0.0.2[​](https://tweblog.com/guide/changelog#_0-0-2)
2025-01-14

新增功能
- [批量导入](https://tweblog.com/guide/feature/tweet-import#%E6%89%B9%E9%87%8F%E5%AF%BC%E5%85%A5)
- [高级功能](https://tweblog.com/guide/feature/tweet-import#%E9%AB%98%E7%BA%A7%E5%8A%9F%E8%83%BD)：在导入的同时关联至转发记录
- 导入、转发任务，可以进行 [任务中止](https://tweblog.com/guide/feature/tweet-import#%E4%BB%BB%E5%8A%A1%E4%B8%AD%E6%AD%A2)
- [自动转发](https://tweblog.com/guide/feature/tweet-forward#%E8%87%AA%E5%8A%A8%E8%BD%AC%E5%8F%91)
- [转发记录设置](https://tweblog.com/guide/feature/tweet-forward#%E8%BD%AC%E5%8F%91%E8%AE%B0%E5%BD%95%E8%AE%BE%E7%BD%AE)

改进
- 解决图片预览的白边问题，优化动画
- 浏览器滚动条样式优化
- 修复分割线高度不一致的问题
- 推文卡片分割线样式优化
- 尽可能解决了页面切换时的卡顿问题
- 数字、秒数输入框动态步进
- 任务信息的轮询获取，增加了退避逻辑

## 0.0.1[​](https://tweblog.com/guide/changelog#_0-0-1)
2025-01-04

项目的初始版本
- 基本功能：发送推文、图片、回复……
- 支持对于 X / Twitter 的导入与转发
- web版，可以充当自己的博客

