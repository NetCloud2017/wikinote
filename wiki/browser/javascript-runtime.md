# 浏览器 JavaScript 运行机制

> Sources: 本地笔记, Unknown
> Raw: [10. JavaScript 数据存储和回收](<../../raw/浏览器原理/10. JavaScript 数据存储和回收.md>); [11. JavaScript Event Loop](<../../raw/浏览器原理/11. JavaScript Event Loop.md>); [12. 宏任务和微任务](<../../raw/浏览器原理/12. 宏任务和微任务.md>); [5. JavaScript 解析流程和执行机制](<../../raw/浏览器原理/5. JavaScript 解析流程和执行机制.md>); [6. JavaScript 调用栈](<../../raw/浏览器原理/6. JavaScript 调用栈.md>); [7. JavaScript 作用域](<../../raw/浏览器原理/7. JavaScript 作用域.md>); [8. JavaScript 作用域链和闭包](<../../raw/浏览器原理/8. JavaScript 作用域链和闭包.md>); [9. JavaScript 中的 this](<../../raw/浏览器原理/9. JavaScript 中的 this.md>)
> Updated: 2026-09-16

## Overview

这一页整理 JavaScript 解析与执行、调用栈、作用域、闭包、`this`、数据存储与回收、事件循环以及宏任务和微任务。

## 解析与执行

- JavaScript 代码会经过解析、预编译和逐段执行，声明提升与作用域在进入执行上下文时确定。
- 调用栈记录函数调用关系，栈深度和同步任务时长会影响页面响应。
- 作用域决定变量可见性，作用域链决定向外层环境查找变量的路径。
- 闭包让函数保留对外部作用域变量的访问能力，也可能延长对象生命周期。

## `this` 与对象上下文

- `this` 的指向由调用方式决定，与函数定义位置无直接关系。
- 箭头函数捕获外层 `this`，普通函数的 `this` 则可在调用时绑定。

## 数据与内存

- JavaScript 数据分布在栈和堆中，引用类型需要关注共享与可达性。
- 垃圾回收基于可达性，闭包、全局引用、监听器和缓存都可能造成内存占用。

## 事件循环

- 事件循环协调调用栈、任务队列和渲染时机。
- 宏任务和微任务执行顺序不同，微任务通常在当前宏任务结束后、下一次渲染前清空。
- 需要区分同步执行、异步回调、Promise 微任务和浏览器渲染任务。

## See Also

- [浏览器架构与网络请求](architecture-and-network.md)
- [JavaScript 语言核心](../javascript/language-core.md)
- [JavaScript 异步与浏览器 API](../javascript/async-and-platform-apis.md)