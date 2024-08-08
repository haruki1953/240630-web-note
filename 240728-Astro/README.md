[240728-初步了解Astro](笔记/240728-初步了解Astro.md)

官网： https://astro.build/

教程： https://docs.astro.build/zh-cn/getting-started/

想要了解的
- [ ] astro的ts支持
- [ ] astro项目中使用eslint
	- https://docs.astro.build/zh-cn/editor-setup/#eslint
	- https://ota-meshi.github.io/eslint-plugin-astro/user-guide/
- [ ] astro使用vue组件，及pinia、vueuse的使用
- [ ] 组件库/模板 有哪些及其使用
- [ ] 服务端渲染SSR、静态站点生成SSG



### 计划通过三个 阶段/项目 完成astro学习

#### 编写类似公司官网的astro站点
- https://github.com/wowthemesnet/Anchor-Bootstrap-UI-Kit
- https://github.com/webpixels/boomerang-free-bootstrap-ui-kit
- 实现一下**预获取**与**视图过渡动画**
- 或许可以使用astro专用的模板/主题
	- https://astro.build/themes/

#### astro博客
- 可以学习推友的模板
	- https://www.tcdw.net/
	- https://github.com/tcdw/koi
	- https://x.com/tcdwww/status/1762045683715146206
- 将自己的xlog博客的内容用astro实现
- 将自己的所有笔记也加入博客
- 部署至GithubPage
	- 或许可以向推友请教 https://x.com/sirongzi/status/1820831388897247401
	- 同时要了解在部署至自己的服务器时如何优雅地更新
- 整理、规范一下自己写笔记的方法，试试更新几篇 博客/笔记
- 问题：如何优雅地使xlog与其保持同步

#### 项目：推文微博客
- 模仿在推特上看到的一个项目
	- https://x.com/ccbikai/status/1820082931089699091
	- https://github.com/ccbikai/BroadcastChannel/
	- 它的作用是 **将你的 Telegram Channel 转为微博客**
- 打算自己实现 **将自己的twitter转为微博客**
- 实现新的功能 **在博客中可选择将其转发至discord**
- 这个应该需要一个后端了