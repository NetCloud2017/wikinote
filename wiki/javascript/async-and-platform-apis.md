# JavaScript 异步与浏览器 API

> Sources: 本地笔记, Unknown
> Raw: [2.XMLHttpRequest](<../../raw/1、JavaScript/2.XMLHttpRequest.md>); [3. Promise及 async、await](<../../raw/1、JavaScript/3. Promise及 async、await.md>); [9、indexDB 数据库](<../../raw/1、JavaScript/9、indexDB 数据库.md>); [离线缓存 mianfest文件](<../../raw/1、JavaScript/离线缓存 mianfest文件.md>); [自定义事件](<../../raw/1、JavaScript/自定义事件.md>); [defer 和 async的区别](<../../raw/1、JavaScript/defer 和 async的区别.md>); [drag API](<../../raw/1、JavaScript/drag API.md>); [Promise.js](<../../raw/1、JavaScript/Promise/Promise.js.md>); [1](<../../raw/1、JavaScript/websocket/1.md>); [2](<../../raw/1、JavaScript/websocket/2.md>); [3](<../../raw/1、JavaScript/websocket/3.md>); [4](<../../raw/1、JavaScript/websocket/4.md>)
> Updated: 2026-09-16

## Overview

这一页覆盖 JavaScript 异步流程和浏览器侧常用 API，包括 `XMLHttpRequest`、`Promise`、`async/await`、`IndexedDB`、WebSocket、脚本加载方式、拖拽、自定义事件和离线缓存。

## 异步模型

- `Promise` 把异步结果表示为可链式处理的状态对象，包含完成、拒绝和等待状态。
- `async/await` 是基于 Promise 的语法抽象，重点在于错误传播、串行与并行执行，以及执行顺序。
- 手写 Promise 的笔记用于理解状态迁移、回调队列和 thenable 解析。

## 浏览器通信

- `XMLHttpRequest` 展示了传统请求生命周期、状态码和事件回调。
- WebSocket 建立持久连接，笔记覆盖基本概念、约束、通讯流程和事件。
- 自定义事件适合在组件或模块之间建立低耦合通信。
- 跨域请求的解决方案另见 [跨域与请求安全](../web/cross-origin.md)。

## 浏览器存储与离线能力

- `IndexedDB` 适合结构化、大容量的浏览器端数据。
- 离线缓存和 manifest 笔记关注资源缓存与离线启动，但需要结合现代 Service Worker 方案核对。
- 深层存储、清理和版本迁移应由业务明确管理。

## 脚本加载与交互

- `defer` 与 `async` 改变脚本下载和执行时机，适用于不同依赖关系。
- 拖拽 API 需要区分拖拽源、放置目标和数据传输。
- 浏览器平台 API 往往涉及权限、兼容性和用户激活状态，不能只依赖语法存在性。

## See Also

- [JavaScript 语言核心](language-core.md)
- [浏览器 JavaScript 运行机制](../browser/javascript-runtime.md)
- [HTTP 协议与缓存](../web/http-protocols.md)