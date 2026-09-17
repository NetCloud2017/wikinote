# Vue 状态管理与 SSR

> Sources: 本地笔记, Unknown
> Raw: [pinia](<../../raw/Vue/pinia/pinia.md>); [1、 vue3 ssr 原理](<../../raw/Vue/vue-ssr/1、 vue3 ssr 原理.md>); [技术调研](<../../raw/Vue/vue-ssr/技术调研.md>); [nuxtjs](<../../raw/Vue/vue-ssr/nuxtjs.md>); [Vuex](<../../raw/Vue/Vuex/Vuex.md>)
> Updated: 2026-09-16

## Overview

这一页整理 Pinia、Vuex、Vue SSR 原理、技术调研和 Nuxt 相关笔记。重点在于全局状态如何组织，以及服务端渲染如何完成首屏输出和客户端接管。

## 状态管理

- Vuex 使用集中式 store 组织状态、变更和异步流程。
- Pinia 提供更轻量的 store 定义方式，适合按领域拆分状态。
- 状态管理应明确模块边界、派生状态、异步副作用和重置策略。

## SSR 与同构

- SSR 在服务端生成首屏 HTML，客户端随后完成激活和交互接管。
- 同构代码需要避免服务端与客户端运行环境差异带来的副作用。
- 数据预取、状态序列化和 hydration 是 SSR 的关键环节。
- 技术调研和 Nuxt 笔记用于比较应用结构和工程约束。

## See Also

- [Vue 响应式、运行时与编译器](reactivity-runtime-and-compiler.md)
- [前端架构与数据结构](../frontend-engineering/architecture-and-data-structures.md)