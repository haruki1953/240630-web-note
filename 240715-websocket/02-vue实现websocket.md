### vue2实现websocket
![README](code/websocket-vue2/vue2/README.md)

注意：vue2在给ws绑定methods中的事件时，要绑定this（唉，this真不是好东西）
```js
const ws = new WebSocket('ws://localhost:8000');
export default {
// ...
	mounted () {
		ws.addEventListener('message', this.handleWsMessage.bind(this), false);
	},
	methods: {
		handleWsMessage (e) {
			const msg = JSON.parse(e.data);
			this.msgList.push(msg);
		}
	}
}
```
教程里把ws放在了全局？感觉不太好，等到vue3时要好好研究一下究竟放在哪里合适


### vue3实现websocket
视频里的方法是这样的
[src/views/Home.vue](code/vue3-websocket/src/views/Home.vue)
```js
const ws = useWebSocket(handleMessage);
function handleMessage (e) {
  const _msgData = JSON.parse(e.data);
  state.msgList.push(_msgData);
}
```
[src/hooks/websocket.js](code/vue3-websocket/src/hooks/websocket.js)，
其封装了WebSocket
```js
import { WS_ADDRESS } from '../configs';

function useWebSocket (handleMessage) {
  const ws = new WebSocket(WS_ADDRESS);

  const init = () => {
    bindEvent();
  }

  function bindEvent () {
    ws.addEventListener('open', handleOpen, false);
    ws.addEventListener('close', handleClose, false);
    ws.addEventListener('error', handleError, false);
    ws.addEventListener('message', handleMessage, false);
  }

  function handleOpen (e) {
    console.log('WebSocket open', e);
  }

  function handleClose (e) {
    console.log('WebSocket close', e);
  }

  function handleError (e) {
    console.log('WebSocket error', e);
  }

  init();

  return ws;
}

export default useWebSocket;
```


### 自认为比较好的一种实现方式

**需求：**
- 自己的e5分享网站中有“通知”和“动态”功能，现在的缺点是只能在访问或刷新网站时获取，不能实时获取
- 以前就一直想解决这个，现在刚了解websocket，就一直在烦恼具体应该怎样实现，从http与ws协议一起获取数据时应该怎样协调

**解决方案：**
- 使用ws但并不通过其发送数据，只是让其给客户端发“更新消息”，客户端收到“更新消息”后再使用http获取数据。
- 这样前端和后端都不用改太多，很多已经写好的方法都可以复用，ws获取消息、http获取数据，这样的设计也感觉比较稳妥与优雅

在项目添加 src/services/ws.ts 文件
```ts
let ws: WebSocket | null = null

export const useWebSocketService () {
	if (ws === null) {
		init()
	}
	return {
		ws
		// 可添加其他如关闭ws、发送数据之类的操作
	}
}

const init = () => {
	ws = new WebSocket(WS_ADDRESS);
	ws.addEventListener('open', handleOpen, false);
	ws.addEventListener('close', handleClose, false);
	ws.addEventListener('error', handleError, false);
	ws.addEventListener('message', handleMessage, false);
}

// handle...
const handleMessage = (e) => {
	const msgData = JSON.parse(e.data);
	// 调用其他services或stores中的方法来操作数据
}
```


感觉自己对于前端（或后端）各个文件夹用途的理解还是有点混乱，现在来重新梳理于规划一下：
- 自己更偏向于根据用途划分文件结构
- 尽量让前端于后端保持一种方面的相似
```
api
前端用的多，用于保存一些接口，以前函数的命名都要加上Service，现在感觉不太好了（因为决定像后端一样加入services文件夹了），结尾的命名改为Api吧

stores
前端自然是用于管理数据状态，其中也可以有一些对于数据的处理操作（以前也是在这里调用api，现在觉得可能不太好了）
对于后端来说，或许也可以加上这个，以此实现一些状态管理。可以基于json持久化，以后研究研究规范一下。

services
后端的话主要的逻辑就是写在这里。前端的复杂度比较小，有时候可能只用stores就足够了，但在复杂时可以写在这里
像初始化数据、重新加载动态等等数据获取（管理请求），可以放在这里，然后再在合适的地方调用
其中可以调用stores用于存取数据，而在stores中不建议调用services。严格划分功能，避免循环调用。
（后端）基于数据库的基础封装可以放在data.ts（或base.ts）中
（前端）……

utils
这里就按照字面意义放一些工具吧，真不知道放在哪里的东西也可以考虑放在这里
还有自定义的组合式api可以放在这里，也许不放在components比较好（根据用途划分）


```


