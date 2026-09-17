# React 基础与运行时

> Sources: 本地笔记, Unknown
> Raw: [1、运行一个原始的React](<../../raw/React/1、运行一个原始的React.md>); [2、useEffect的使用](<../../raw/React/2、useEffect的使用.md>); [react虚拟dom](<../../raw/React/react虚拟dom.md>)
> Updated: 2026-09-16

## Overview

这一页整理 React 项目运行、`useEffect` 和虚拟 DOM。重点是从组件渲染、副作用和视图更新三个角度理解 React 的运行模型。

## 项目与组件

- 原始 React 项目需要理解入口、JSX 转换、根节点挂载和开发构建。
- 组件根据 props 与 state 渲染视图，更新时应保持数据流可追踪。
- 虚拟 DOM 是运行时用于描述界面结构的 JavaScript 对象，真实 DOM 更新通过比较和提交完成。

## 副作用

- `useEffect` 用于在渲染后同步外部系统，不应在渲染阶段直接产生副作用。
- 依赖数组决定副作用何时重新执行。
- 清理函数用于取消订阅、释放资源和撤销上一次副作用。
- 副作用依赖必须完整表达，否则会出现旧值闭包或重复执行问题。

## See Also

- [Next.js 笔记](nextjs.md)
- [Redux 与 MobX 数据流管理对比](redux-vs-mobx-data-flow.md)
- [JavaScript 异步与浏览器 API](../javascript/async-and-platform-apis.md)