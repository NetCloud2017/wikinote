# Next.js 笔记

> Sources: 本地笔记, Unknown; 本地笔记, 2026-10-14
> Raw: [11](<../../raw/React/nextjs/11.md>); [27](<../../raw/React/nextjs/27.md>); [28](<../../raw/React/nextjs/28.md>); [3](<../../raw/React/nextjs/3.md>); [32](<../../raw/React/nextjs/32.md>); [40](<../../raw/React/nextjs/40.md>); [layout 和template总结](<../../raw/React/nextjs/layout 和template总结.md>)
> Updated: 2026-10-14

## Overview

这一页整理 Next.js App Router 的路由约定、组件渲染边界、运行时选择、静态导出、国际化，以及 Layout 与 Template 的核心区别和常见踩坑。

## 应用模型

- 路由决定页面与布局如何映射到文件结构。
- 组件渲染需要区分服务端与客户端边界。
- runtime 运行时选择影响代码运行环境和可用能力。
- 静态导出适合生成可静态托管的产物，但对动态能力有限制。
- 国际化需要处理路由、语言资源和内容切换。

## Layout 与 Template

App Router 中 `layout.tsx` 和 `template.tsx` 都是包裹子路由的共享 UI，但导航时行为截然不同。

### 相同点

两者都放在路由段目录下，接收 `children`，可多级嵌套，可在组件内直接 fetch 数据，默认为 Server Component（可加 `'use client'`），都能拿到动态路由的 `params`。

### 核心区别

| 维度 | Layout | Template |
|---|---|---|
| 导航行为 | 持久存在，不重新挂载 | 每次导航都重新挂载（新实例） |
| State | 保留，组件状态和滚动位置不丢 | 重置，全部重新初始化 |
| CSS 动画 | 只在首次挂载播放一次 | 每次导航重新播放 |
| DOM / Hydration | 复用同一节点，只 hydrate 一次 | 新建 DOM，每次重新 hydrate |
| 性能 | 更轻量 | 有额外开销 |
| 典型场景 | 导航栏、侧边栏、面包屑 | 需要每次进入重置状态或重放动画的 UI |

嵌套顺序为 `Layout > Template > Page`，两者可同时存在。

### 避坑要点

- 根 layout 必须包含 `<html>` 和 `<body>`，否则报错。
- 想在导航时播放进入动画用 template，不是 layout——layout 的 CSS animation 只跑一次。
- Layout 里的 `useState` 会跨路由保留，切面板后状态不丢；不希望这样就移到 page 或用 template。
- Template 会多一层 DOM 包裹，CSS 选择器依赖直接父子关系时要算上这层。
- Template 每次导航都重新 fetch 数据，数据不常变就用 layout。
- Layout 不能向 children 传响应式 props，需要传值用 Context 或 URL params。
- Parallel Routes / Intercepting Routes 下 layout 复用可能造成"页面没刷新"的错觉，实际是 layout 没换、page 已换。

## 使用边界

这些笔记较短，适合作为版本相关功能的索引。升级 Next.js 或调整渲染模式时，应重新核对官方约定和项目配置。

## See Also

- [React 基础与运行时](fundamentals.md)
- [Vue 状态管理与 SSR](../vue/state-management-and-ssr.md)
- [前端架构与数据结构](../frontend-engineering/architecture-and-data-structures.md)
