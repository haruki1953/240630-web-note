## websocket概述

http请求只能由客户端发起
- 轮询（浪费带宽，实时性差，服务器压力大）（长轮询）

WebSocket，
- 2008年提出，2011年成为标准
- HTML5新增的协议
- 可以在浏览器和服务器之间建立一个全双工的通信通道

### 通信流程
1. 浏览器发起http请求，请求建立WebSocket连接
![](assets/Pasted%20image%2020240715153852.png)

2. 服务器响应同意协议更改
![](assets/Pasted%20image%2020240715153940.png)

3. 相互发送数据。discord就有ws
![](assets/Pasted%20image%2020240715154603.png)

![](assets/Pasted%20image%2020240715154737.png)


- WebSocket协议建立在tcp协议基础上的，所以服务器端也容易实现，不同的语言都有支持
- tcp协议是全双工协议，http协议基于它，但设计成了单向的
- WebSocket没有同源限制

### 实现websocket
websocket就是一些事件的问题，由这些事件以及响应的处理函数来实现功能
1. open
2. close
3. error
4. message
5. connection（后端，管理连接）

（客户端简单实现）
```js
let ws = new WebSocket("ws://localhost:8080/myWs1")
ws.onopen = function (){
	ws.send("hello")
}
ws.onmessage = function (message){
	console.log(message.data)
}
```

## js实现websocket
https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket

### 项目起步
安装vite用于构建项目，修改 [package.json](code/websocket-origin/chat/package.json)
```json
  "scripts": {
    "dev": "vite"
  },
```
pnpm dev 即可启动

1. entry.html - Input username
	- localStorage to save the username
	- click for enter the chatting room
2. index.html - websocket
	- List - show messages list
	- input - message
	- btn - send message

- [entry.html](code/websocket-origin/chat/entry.html)
- [index.html](code/websocket-origin/chat/index.html)


[**entry.html**](code/websocket-origin/chat/entry.html) ：在html中引入js
```html
<body>
  <input type="text" id="username" placeholder="请输入用户名" />
  <button id="enter">进入聊天室</button>
  
  <script src="js/entry.js"></script>
</body>
```
[js/entry.js](code/websocket-origin/chat/js/entry.js)
```js
;((doc, storage, location) => {

  const oUsername = doc.querySelector('#username');
  const oEnterBtn = doc.querySelector('#enter');

  const init = () => {
    bindEvent();
  }

  function bindEvent () {
    oEnterBtn.addEventListener('click', handleEnterBtnClick, false);
  }

  function handleEnterBtnClick () {
    const username = oUsername.value.trim();
    if (username.length < 6) {
      alert('用户名不小于6位');
      return;
    }
    // 保存用户名，跳转至index.html
    storage.setItem('username', username);
    location.href = 'index.html';
  }

  init();

})(document, localStorage, location);
```
[240716-在js文件中使用立即执行函数表达式](笔记/240716-在js文件中使用立即执行函数表达式.md)


### 实现websocket客户端
[index.html](code/websocket-origin/chat/index.html)
```html
<body>
  <ul id="list"></ul>
  <input type="text" id="message" placeholder="请输入消息" />
  <button id="send">发送</button>

  <script src="js/index.js"></script>
</body>
```

```js
;((doc, Socket, storage, location) => {
  
  const oList = doc.querySelector('#list');
  const oMsg = doc.querySelector('#message');
  const oSendBtn = doc.querySelector('#send');
  // 1.实例化WebSocket
  const ws = new Socket('ws:localhost:8000');
  let username = '';

  const init = () => {
    bindEvent();
  }

  function bindEvent () {
    oSendBtn.addEventListener('click', handleSendBtnClick, false);
    // 2.绑定事件
    ws.addEventListener('open', handleOpen, false);
    ws.addEventListener('close', handleClose, false);
    ws.addEventListener('error', handleError, false);
    ws.addEventListener('message', handleMessage, false);
  }

  function handleSendBtnClick () {
    const msg = oMsg.value;

    if (!msg.trim().length) {
      return;
    }
    // 发送数据
    ws.send(JSON.stringify({
      user: username,
      dateTime: new Date().getTime(),
      message: msg
    }));

    oMsg.value = '';
  }

  function handleOpen (e) {
    console.log('Websocket open', e);
    username = storage.getItem('username');

    if (!username) {
      location.href = 'entry.html';
      return;
    }
  }

  function handleClose (e) {
    console.log('Websocket close', e);
  }

  function handleError (e) {
    console.log('Websocket error', e);
  }

  // 接收数据
  function handleMessage (e) {
    console.log('Websocket message');
    const msgData = JSON.parse(e.data);
    oList.appendChild(createMsg(msgData));
  }

  function createMsg (data) {
    const { user, dateTime, message } = data;
    const oItem = doc.createElement('li');
    oItem.innerHTML = `
      <p>
        <span>${ user }</span>
        <i>${ new Date(dateTime) }</i>
      </p>
      <p>消息：${ message }</p>
    `;

    return oItem;
  }

  init();
})(document, WebSocket, localStorage, location);
```
[240716-html事件绑定](笔记/240716-html事件绑定.md)


### 服务端简单实现
https://github.com/websockets/ws
```
# 安装websocket包
pnpm i ws

# nodemon全局安装
pnpm i nodemon -g
```

修改 [package.json](code/websocket-origin/server/package.json) ，pnpm dev 运行后端
```json
  "scripts": {
    "dev": "nodemon index.js"
  },
```

[index.js](code/websocket-origin/server/index.js)
```js
const Ws = require('ws');

;((Ws) => {
  // ws:localhost:8000
  // 1.实例化WebSocketServer
  const server = new Ws.Server({ port: 8000 });
  
  const init = () => {
    bindEvent();
  }

  function bindEvent () {
    // 2.绑定事件
    server.on('open', handleOpen); // 看了下文档没有open，教程上应该写错了
    server.on('close', handleClose);
    server.on('error', handleError);
    server.on('connection', handleConnection);
  }

  function handleOpen () {
    console.log('Websocket open');
  }

  function handleClose () {
    console.log('Websocket close'); 
  }

  function handleError () {
    console.log('Websocket error');
  }

  function handleConnection (ws) {
    console.log('Websocket connected');
    // 绑定接收消息事件，在connection事件中
    ws.on('message', handleMessage);
  }

  function handleMessage (msg) {
    // 广播：向所有已连接的客户端发送消息
    server.clients.forEach(function (c) {
      c.send(msg);
    })
  }

  init();

})(Ws);
```








