# Redux 与 MobX 数据流管理对比

> Sources: 勾崽, 2021-03-11
> Raw: [数据流管理方案：Redux 和 MobX 哪个更好？](../../raw/React/2021-03-11-redux-vs-mobx-data-flow-management.md)
> Updated: 2026-09-16

## Overview

文章从 React 组件通信的局限出发，梳理了从 props、Context API 到 Redux 和 MobX 的数据流管理演进。核心结论是：随着跨层级状态共享和业务逻辑复杂度上升，状态管理需要从组件和 UI 中抽离；Redux 代表严格单向、显式且偏函数式的数据流方案，MobX 则代表基于响应式追踪、代码量更少且偏面向对象的方案。其对 Redux 的判断具有当时的生态背景。

## 为什么需要数据流管理

### 基于 props 的单向数据流

- 父组件可以直接通过 `this.props` 向子组件传递数据。
- 子组件可以通过父组件传入的回调函数，把数据作为参数交回父组件。
- 兄弟组件可以借助共同父组件，通过回调函数和 props 间接通信。
- 如果不是直接兄弟，跨组件通信会导致代码量持续增加，也无法自然做到状态同步和共享。

### Context API 的边界

Context API 是 React 官方提供的组件树全局通信方式，关键要素是 `React.createContext`、`Provider` 和 `Consumer`。从 React 16.3 开始，Context API 得到改进，可用性更强；Consumer 既能读取 Provider 下发的数据，也能接收后续更新。

但 Context API 没有把数据层和展示层分开。复杂组件的 UI 与状态逻辑仍会堆在一起，执行一个动作可能触发一系列 `setState`。因此还需要把数据流管理从 React 组件概念中抽离，让 React 更专注于 View 层。

## Redux：显式单向数据流

### 从 Flux 继承的思想

Flux 由 Facebook 在 2014 年的 F8 大会提出，针对 MVC 在大型前端项目中 Model 与 View 关系错综复杂、存在双向数据流的问题，提出单向数据流架构。Flux 的核心角色包括：

| 角色 | 作用 |
|------|------|
| View | 用户界面，可由 React、Vue、Angular 等形式实现 |
| Action | View 发出的消息，用于触发状态改变 |
| Dispatcher | 分发 Action |
| Store | 保存应用状态，并定义修改状态的逻辑 |

典型工作流是：用户与 View 交互，View 发起 Action，Dispatcher 将 Action 派发给 Store，Store 更新状态后通知 View 更新界面。Flux 本身在 Facebook 之外使用较少，但它奠定了 Redux 等后续方案的基础。

### Redux 的核心模型

Redux 是 JavaScript 状态容器，提供可预测化的状态管理，不限于 React，也可以在 Vue 或原生 JavaScript 中使用。它的核心是一个独立于组件的数据源：

- `store`：单一、只读的数据源。
- `action`：对数据变化的描述，其中 `type` 是必传字段。
- `reducer`：纯函数，根据 action 处理变化并返回新的 state。

Redux 的数据流严格单向。组件从 store 读取状态，修改状态只能通过派发 action；action 被 reducer 读取后生成新 state，再更新 store 并驱动视图变化。组件因此可以自由但有序地共享和修改全局状态。

编码流程通常从 `createStore(reducer, 初始状态, 中间件)` 开始；`createStore` 一般只有 reducer 必传。dispatch action 后，reducer 根据 `type` 进行定向更新，执行完成后返回新 state。文章给出的数字加减案例也强调了其模板代码较繁琐。

### 文章列出的优点

- 状态持久化：global store 可以在组件销毁后保留状态。
- 状态可回溯：action 可序列化，reducer 不修改原状态而是返回新状态。
- 函数式编程：使用纯函数，输出依赖输入，没有副作用。
- 中间件：通过类 Express 的中间件机制处理异步数据流，并可通过第三方插件细粒度控制数据流动。

### 文章列出的缺点

- 模板代码繁重，修改一个 state 可能涉及多个文件。
- 多组件共用 store 状态时，需要注意初始化和清空。
- 每次 dispatch action 都会遍历 reducer 并重新计算 `connect`，产生额外损耗。
- store 较大且频繁修改时，可能出现页面卡顿。
- 原文将“不支持 TypeScript”列为缺点；这一条属于文章的原始判断，使用时应结合当前 Redux 生态重新核对。
- 对大型多人团队而言，Redux 的约束有助于形成清晰规约；对快速迭代的小团队，模板负担可能较重。

## MobX：响应式数据流

文章以 MobX 5 为例。MobX 5 使用 ES6 的 Proxy 追踪属性，旧版本则使用 `Object.defineProperty`。它通过隐式订阅自动追踪被监听对象的变化并更新数据。

Redux 把更多工作交给用户，以保持自身纯净；MobX 则把大部分实现交给内部处理，用户主要操作 Observable data。其基本理念仍是 Model 与 View 完全分离，业务逻辑可以写在 action 中，Observer view 会自动响应，因此主打响应式设计，编程风格仍是传统的面向对象范式。

文章列出的 MobX 优点包括：

- 代码量少。
- 基于数据劫持实现精准定位，也就是真正意义上的局部更新。
- 通过多个 store 抽离业务逻辑，实现 Model 与 View 分离。
- 响应式性能良好，可以应对频繁交互。
- 可以替代 React 自身的状态管理。
- 支持 TypeScript。

## 方案对比

| 维度 | Redux | MobX |
|------|-------|------|
| 主要范式 | 单向数据流、函数式 | 响应式、面向对象 |
| 状态修改 | 派发 action，由 reducer 返回新 state | 修改 Observable data，由隐式订阅触发更新 |
| 更新范围 | dispatch 后遍历 reducer，并可能重新计算 `connect` | 基于数据劫持精准定位，局部更新 |
| 代码成本 | 模板代码多，改动可能涉及多个文件 | 代码量少 |
| 大型团队 | 约束清晰，容易形成规约 | 文章未评价 |
| 快速迭代小团队 | 模板负担较重 | 文章未评价 |
| 高频交互 | store 较大且频繁修改时可能卡顿 | 响应式性能可支撑频繁交互 |
| TypeScript | 原文称不支持 | 原文称支持 |

文章将当时主流方案概括为两大派：以 Redux 为代表的函数式库，以及以 MobX 为代表的响应式库。

## See Also

- [React 基础与运行时](fundamentals.md)
- [Next.js 笔记](nextjs.md)
