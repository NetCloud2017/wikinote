# Next.js 笔记

> Sources: 本地笔记, Unknown
> Raw: [11](<../../raw/React/nextjs/11.md>); [27](<../../raw/React/nextjs/27.md>); [28](<../../raw/React/nextjs/28.md>); [3](<../../raw/React/nextjs/3.md>); [32](<../../raw/React/nextjs/32.md>); [40](<../../raw/React/nextjs/40.md>)
> Updated: 2026-09-16

## Overview

这一页整理 Next.js 路由、组件渲染、运行时选择、静态导出、组件使用和国际化等片段。

## 应用模型

- 路由决定页面与布局如何映射到文件结构。
- 组件渲染需要区分服务端与客户端边界。
- runtime 运行时选择影响代码运行环境和可用能力。
- 静态导出适合生成可静态托管的产物，但对动态能力有限制。
- 国际化需要处理路由、语言资源和内容切换。

## 使用边界

这些笔记较短，适合作为版本相关功能的索引。升级 Next.js 或调整渲染模式时，应重新核对官方约定和项目配置。

## See Also

- [React 基础与运行时](fundamentals.md)
- [Vue 状态管理与 SSR](../vue/state-management-and-ssr.md)
- [前端架构与数据结构](../frontend-engineering/architecture-and-data-structures.md)