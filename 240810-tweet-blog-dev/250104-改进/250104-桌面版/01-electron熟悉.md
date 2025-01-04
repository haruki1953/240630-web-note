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
