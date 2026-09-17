# Vue 响应式、运行时与编译器

> Sources: 本地笔记, Unknown
> Raw: [1、编程范式](<../../raw/Vue/1、编程范式.md>); [2、企业应用的开发与设计原则](<../../raw/Vue/2、企业应用的开发与设计原则.md>); [3、学习vue的概念](<../../raw/Vue/3、学习vue的概念.md>); [4、副作用](<../../raw/Vue/4、副作用.md>); [5、vue 架构搭建](<../../raw/Vue/5、vue 架构搭建.md>); [6、如何阅读源码](<../../raw/Vue/6、如何阅读源码.md>); [1、vue3 结构](<../../raw/Vue/Vue3/1、vue3 结构.md>); [2、vue 响应式系统](<../../raw/Vue/Vue3/2、vue 响应式系统.md>); [3、ref ](<../../raw/Vue/Vue3/3、ref .md>); [4、conputed 响应式](<../../raw/Vue/Vue3/4、conputed 响应式.md>); [5、watch 侦听器](<../../raw/Vue/Vue3/5、watch 侦听器.md>); [6、 scheduler 调度系统](<../../raw/Vue/Vue3/6、 scheduler 调度系统.md>); [7、runtime 运行时核心设计设计原则](<../../raw/Vue/Vue3/7、runtime 运行时核心设计设计原则.md>); [8、组件设计原理， 与渲染方案](<../../raw/Vue/Vue3/8、组件设计原理， 与渲染方案.md>); [9、runtime的 diff 算法](<../../raw/Vue/Vue3/9、runtime的 diff 算法.md>); [Vue3](<../../raw/Vue/Vue3/Vue3.md>); [1、watch  watchEffect](<../../raw/Vue/Vue3/API/1、watch  watchEffect.md>); [10、compiler 编译器](<../../raw/Vue/Vue3/10、compiler 编译器.md>); [11、 compiler AST](<../../raw/Vue/Vue3/11、 compiler AST.md>); [12、compile 转化成 js AST  流程](<../../raw/Vue/Vue3/12、compile 转化成 js AST  流程.md>); [13、render函数生成](<../../raw/Vue/Vue3/13、render函数生成.md>); [14、 compiler 对 响应式数据处理](<../../raw/Vue/Vue3/14、 compiler 对 响应式数据处理.md>); [15、compiler 对对个子节点处理](<../../raw/Vue/Vue3/15、compiler 对对个子节点处理.md>); [16、compiler 对 指令处理](<../../raw/Vue/Vue3/16、compiler 对 指令处理.md>)
> Updated: 2026-09-16

## Overview

这一页整理 Vue 的编程范式、核心概念、响应式系统、调度器、运行时、组件设计、diff、编译器和 Vue 3 整体结构。它描述模板如何转成渲染逻辑，以及状态变化如何驱动视图更新。

## 响应式系统

- 响应式系统负责追踪依赖并在数据变化时触发更新。
- `ref`、`computed`、`watch` 和 `watchEffect` 分别面向基本值包装、派生状态、显式侦听和自动追踪副作用。
- 副作用是响应式更新的核心概念，调度器决定副作用何时执行以及如何批处理。
- 阅读源码时应区分依赖收集、触发更新和调度执行三个阶段。

## 运行时与组件

- Vue 3 整体结构可以按响应式、运行时、编译器和平台适配拆分。
- 组件设计需要处理状态、属性、事件、插槽和渲染方案。
- diff 算法用于比较前后虚拟节点，并尽可能复用真实 DOM。
- 运行时核心设计强调渲染器与平台解耦。

## 编译器

- 编译器把模板解析为 AST，再经过转换生成 JavaScript AST 和 render 函数。
- 编译阶段可以处理静态提升、响应式数据和指令。
- 多层级模板和指令处理说明编译器需要在结构转换与运行时语义之间保持对应。

## 架构与源码阅读

- 编程范式和架构搭建笔记关注 Vue 的设计选择与阅读源码的方法。
- 阅读源码应从概念模型进入，再沿调用链定位响应式、调度和渲染边界。

## See Also

- [Vue 状态管理与 SSR](state-management-and-ssr.md)
- [前端构建与工具链](../frontend-engineering/build-and-toolchain.md)
- [JavaScript 模块化与设计模式](../javascript/modularity-and-patterns.md)