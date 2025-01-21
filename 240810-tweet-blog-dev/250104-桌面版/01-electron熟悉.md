https://www.electronjs.org/zh/docs/latest/


好像不能用pnpm

更新yarn
```
yarn -v
npm install -g yarn@latest
```

检查，配置npm代理
```
npm config get proxy
npm config get https-proxy

npm config set proxy http://127.0.0.1:10809
npm config set https-proxy http://127.0.0.1:10809
```

配置yarn代理
```
yarn config set proxy http://127.0.0.1:10809
yarn config set https-proxy http://127.0.0.1:10809
```

**electron安装的网络问题**
- https://www.electronjs.org/zh/docs/latest/tutorial/installation#%E4%BB%A3%E7%90%86
- https://github.com/gajus/global-agent/blob/v2.1.5/README.md#environment-variables
```sh
# cmd
set ELECTRON_GET_USE_PROXY=true
set GLOBAL_AGENT_ENVIRONMENT_VARIABLE_NAMESPACE=GLOBAL_AGENT_
set GLOBAL_AGENT_HTTP_PROXY=http://127.0.0.1:10809
set GLOBAL_AGENT_HTTPS_PROXY=http://127.0.0.1:10809
echo $ELECTRON_GET_USE_PROXY

# bash
export ELECTRON_GET_USE_PROXY=true
export GLOBAL_AGENT_ENVIRONMENT_VARIABLE_NAMESPACE=GLOBAL_AGENT_
export GLOBAL_AGENT_HTTP_PROXY=http://127.0.0.1:10809
export GLOBAL_AGENT_HTTPS_PROXY=http://127.0.0.1:10809
echo $ELECTRON_GET_USE_PROXY
```
好，成功了，现在node下载ELECTRON是走的代理。注意，每次安装时都要设置



## 开始入门
```
mkdir my-electron-app-250104 && cd my-electron-app-250104
yarn init
```

```
{
  "name": "my-electron-app-250104",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "author": "haruki",
  "license": "MIT"
}
```

```
yarn add --dev electron
```

electron和我想象中好像不太一样，不过还是勉强动起来了
![](assets/Pasted%20image%2020250105135856.png)

是让electron通过url直接获取后端所托管的前端
```js
const createWindow = () => {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // 加载 index.html
  // mainWindow.loadFile('index.html')
  mainWindow.loadURL('http://127.0.0.1:3000/admin/')

  // 打开开发工具
  // mainWindow.webContents.openDevTools()
}
```

哦哦哦，自己好像可以想象到该怎样将web应用改为桌面版了
```
在后端的基础上更改，安装electron，index中后端启动后创建窗口

可以在 router service system 等文件夹下新增 desktop 文件或文件夹，来编写用于控制桌面窗口的代码

创建窗口相关代码可以封装在 system/desktop

electron 主流的预加载脚本方式，IPC（进程间通信），是直接在前端控制桌面窗口，自己感觉有点混乱
自己想到一种比较好方式，在后端创建用于控制窗口的接口 router/desktop，前端来调用接口来控制窗口
```

250118，完善了想法
```
创建electron专用的前后端
tweblog-electron-hono
tweblog-electron-vue3

将前端生成在 tweblog-electron-hono/dist-vue3
pnpm build --outDir ../tweblog-electron-hono/dist-vue3 --emptyOutDir

后端在 tweblog-electron-hono/src 编写ts代码，
编译后在 tweblog-electron-hono/dist

后端对electron的控制，写在 tweblog-electron-hono/src/electron ，将在 index.ts 中调用
```
