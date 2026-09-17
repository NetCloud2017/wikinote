# 前端构建与工具链

> Sources: 本地笔记, Unknown
> Raw: [1. 前端模块化](<../../raw/前端工程化/1. 前端模块化.md>); [工程化相关命令](<../../raw/前端工程化/工程化相关命令.md>); [AST](<../../raw/前端工程化/AST.md>); [commander](<../../raw/前端工程化/commander.md>); [04横向对比主流构建工具，了解构建工具的设计考量](<../../raw/前端工程化/lg前端基础建设与架构/04横向对比主流构建工具，了解构建工具的设计考量.md>); [05Vite实现：从源码分析出发，构建bundlele开发工程](<../../raw/前端工程化/lg前端基础建设与架构/05Vite实现：从源码分析出发，构建bundlele开发工程.md>); [06core-j及垫片理念：设计一个“最完美”的Polyfill方案](<../../raw/前端工程化/lg前端基础建设与架构/06core-j及垫片理念：设计一个“最完美”的Polyfill方案.md>); [07梳理混乱的Babel，不再被编译报错困扰](<../../raw/前端工程化/lg前端基础建设与架构/07梳理混乱的Babel，不再被编译报错困扰.md>); [08探索前端工具链生态，制定一个统一标准化babel-preet](<../../raw/前端工程化/lg前端基础建设与架构/08探索前端工具链生态，制定一个统一标准化babel-preet.md>); [10代码拆分和按需加载：缩减bundleize，把性能做到极致](<../../raw/前端工程化/lg前端基础建设与架构/10代码拆分和按需加载：缩减bundleize，把性能做到极致.md>); [11TreeShaking：移除JavaScript上下文中的未引用代码](<../../raw/前端工程化/lg前端基础建设与架构/11TreeShaking：移除JavaScript上下文中的未引用代码.md>); [12如何理解AST实现和编译原理？](<../../raw/前端工程化/lg前端基础建设与架构/12如何理解AST实现和编译原理？.md>); [14解析Webpack源码，实现自己的构建工具](<../../raw/前端工程化/lg前端基础建设与架构/14解析Webpack源码，实现自己的构建工具.md>); [1、源码分析](<../../raw/前端工程化/vite/1、源码分析.md>); [vite 项目创建](<../../raw/前端工程化/vite/vite 项目创建.md>); [1.  webpack 编译一行代码](<../../raw/前端工程化/Webpack/1.  webpack 编译一行代码.md>); [源码分析 1](<../../raw/前端工程化/Webpack/源码分析 1.md>); [devServer配置](<../../raw/前端工程化/Webpack/devServer配置.md>); [loader](<../../raw/前端工程化/Webpack/loader.md>); [Plugin](<../../raw/前端工程化/Webpack/Plugin.md>); [readme](<../../raw/前端工程化/Webpack/readme.md>); [webpack](<../../raw/前端工程化/Webpack/webpack.md>)
> Updated: 2026-09-16

## Overview

这一页整理前端模块化、AST、构建工具、Webpack、Vite、Babel、polyfill、代码拆分和 tree shaking。核心问题是：源码如何经过分析、转换、依赖打包和运行加载，最终形成可部署资产。

## 模块与 AST

- 前端模块化关注依赖声明、静态分析、加载时机和作用域隔离，是构建工具工作的前提。
- AST 是编译流程中的结构化中间表示；解析、转换和代码生成都围绕 AST 完成。
- 理解 AST 后，可以进一步理解 Babel 插件、构建工具转换器以及自定义代码处理。

## 构建工具

- 主流构建工具在设计目标上各有取舍，常见维度包括开发启动、生产打包、插件机制和配置复杂度。
- Webpack 以入口、依赖图、loader、plugin 和输出为核心，源码分析笔记关注编译流程。
- Vite 利用开发期原生加载和生产构建的差异，提供更快的开发反馈。
- 工程化命令和 `commander` 笔记涉及命令行工具的基础组织方式。

## 编译、兼容与产物优化

- Babel 处理语法转换与目标环境兼容，polyfill 则补足运行时能力。
- core-js 与垫片方案强调按目标环境控制代码体积，避免无条件注入全部兼容代码。
- 代码拆分和按需加载把产物划分为可延迟加载的 chunk。
- tree shaking 通过静态依赖分析移除未引用代码，效果依赖模块格式和副作用声明。

## 使用边界

工具链配置与版本相关性强。这里的笔记适合建立概念地图，落地时仍需结合项目实际版本、构建目标、插件兼容性和产物分析结果。

## See Also

- [包管理与发布流程](package-management-and-release.md)
- [前端架构与数据结构](architecture-and-data-structures.md)
- [JavaScript 模块化与设计模式](../javascript/modularity-and-patterns.md)