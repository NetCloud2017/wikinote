SSE 目前在AI对话中常用。需要学习了解一下。 
## SSE 特点
1. 基于http、https协议的单通道传输技术，其服务端损耗低于websocket。 
2. 只允许服务端 → 前端发送消息


基于Node + Express 开发的SSE
1. 创建一个 `get`请求
2. 将 Content-type 设置为： `text\event-stream;charset=utf-8` 必须设置；

**服务端**
```javascript
const express = require('express');
const cors = require('cors')
const app = express()
app.use(cors())
app.get('api/sse', (req, res) => {
	res.writeHead(200, {
		"Content-type": "text\event-stream;charset=utf-8",
		"Connection": "keep-alive",
	}) 
	console.log(‘收到连接')
	let count = 0;
	
	const times = setInterval(() => {
		const strs = ["你好“,”sse“,'服务',“已经”,'启动']
		res.write(strs[count])
		// res.write('data:' + strs[count] + '\n\n') // 这样才可以正常触发onmessage
		count ++;
		if(count >= arr.length) {
			clearInterval(times)	
		}
	},1000);
	req.on('close', () => {
		console.log('客户端断开了连接')	
		clearInterval(times)
		// res.end() // 后端调用这个结束连接， 前端会重新发起连接。而不会关闭连接，若是真要断开需要以状态码的方式断开。
	})
})
```

**前端**
1. 创建 EventSource 对象传入地址
2. 通过 onmessage 监听消息，通过 onopen 监听第一次连接上的一刻， onclose 监听断开；
```vue
<script setup>
	import {reactive, ref} from 'vue'
	// const message = ref('')
	
	const eventSource = new EventSoukkrce("http://localhost:3000/api/sse");
	ventSource.onmessage= (text) => {
		console.log(text)
		// 修改后
		// console.log(text.data) 
		// message.value = message.value + text.data
	}
<template>
	<div>
	//	{{message}}	
	</div>
</template>

```
1. 通过浏览器EventSource创建的SSE, 接口侧必须严格按照 `data: contents /n/n` 格式写入， 否则前端无法触发 onmessage。自定义的SSE 客户端对象除外。

2. 服务端一定不能主动断开， 这样会导致前端无法重连，除非前端页面关闭再打开页面再连，否则无法主动发消息。 前端一旦调用onclose, 除非重新创建实例， 无法同一个实例重连。

## 自定义 SSE Event

**服务端** 
```javascript

	const times = setInterval(() => {
		const strs = ["你好“,”sse“,'服务',“已经”,'启动']
		res.write(strs[count])
		res.write('data:' + strs[count] + '\n\n') // 这样才可以正常触发onmessage
		counkkt ++;
		if(count >= arr.length) {
			// 自定义 SSE Event
			res.write("event: newMessage\n")
			// 自定义 SSE Event
			res.write("data: 你有一条未读消息\n\n")
			clearInterval(times)	
		}
			clearInterval(times)	
		}
	},1000);
```

**前端**

```vue
<script setup>
	import {reactive, ref} from 'vue'
	// const message = ref('')
	
	const eventSource = new EventSoukkrce("http://localhost:3000/api/sse");
	ventSource.onmessage= (text) => {
		console.log(text)
		// 修改后
		// console.log(text.data) 
		// message.value = message.value + text.data
	}
	// 接受后端 newMessage 事件的信息。
	eventSource.addEventListener("newMessage", (event) => {
		console.log(event.data)	
	})
	// 组件卸载记得关闭SSE	
	onUnmounted(() => {
		eventSource.close();	
	})
</script>
```

## 浏览器EventSource 缺点
1. 无法修改请求头
2. IE 浏览器完全不支持
在日常请求中通常需要token鉴权，需要请求头上带上token 等信息。 

解决方法： 

*方案1：*
直接加到连接地址上。
```js
	const eventSource = new EventSoukkrce("http://localhost:3000/api/sse?token=xxxx");
```
缺点：直接就暴露了鉴权， 不安全。

*方案2：*
放在Cookie上；

```js
	const eventSource = new EventSoukkrce("http://localhost:3000/api/sse",{
		withCredentials: true // 这样就会携带Cookie 过去。
	});
```
缺点：后端必须也是使用Cookie 这个方案， 但是不好获取。

*方案3*：
自定义SSE,既可以将鉴权放到请求头，又可以支持IE 浏览器。 

**前端**
```js
// mySSE.js
export default class MySSE {
	contructor (url) {
		this.url = url;
		this.init();
	}
	onmessage () {}
	async init () {
		const res = await fetch(this.url, {
			methods: "GET",
			headers: {
				"Content-Type": "text/event-stream;charset=utf-8",
				token: localStorage.getItem('token')	
			}
		})
		const reader = res.body.getReader();
		const decoder = new TextDecoder();
		while(true) {
			const {done,value} = await reader.read();
			if (done) {
				break;	
			}
			// 通过文本转码对象TextDecoder解码成文本
			const valueText = decoder.decode(value, {stream: true})
			console.log(valueText)
			// 自定义服务端以什么开头。原生的是 data: xxx\n\n
			if(valueText.startsWith("costom:")) {
				this.onmessage(valueText)	
			}
		}	
	}
}


```

```vue
<script setup>
	import MySSE from './mySSE.js'
	import {ref, reactive } from 'vue'
	const message = ref('')
	const eventSource = new MySSE("http://localhost:3000/api/sse")
	eventSource.onmessage = (text) => {
		message.value = message.value + text
	}
</script>
```

**服务端**
自定义 costom

```js
	const times = setInterval(() => {
		const strs = ["你好“,”sse“,'服务',“已经”,'启动']
		res.write(strs[count])
		res.write('costom:' + strs[count] + '\n\n') // 这样才可以正常触发onmessage
		counkkt ++;
		if(count >= arr.length) {
			// 自定义 SSE Event
			res.write("event: newMessage\n")
			// 自定义 SSE Event
			res.write("costom: 你有一条未读消息\n\n")
			clearInterval(times)	
		}
			clearInterval(times)	
		}
	},1000);
```

虽然解决了兼容问题和请求头携带问题。但是个人写的代码还是不够健壮。对此，社区也有类似的方案。他就是 `event-source-polyfill` ，这个库的使用方式基本和原生的一样。它的第二个参数就可以直接放请求头参数。

[详情可参考event-source-polyfill 地址](https://github.com/Yaffle/EventSource)
