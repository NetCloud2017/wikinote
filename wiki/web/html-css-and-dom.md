# HTML、CSS、DOM 与 BOM

> Sources: 本地笔记, Unknown
> Raw: [盒模型](<../../raw/2、HTML 、CSS/盒模型.md>); [绝对路径和相对路径](<../../raw/2、HTML 、CSS/绝对路径和相对路径.md>); [BFC](<../../raw/2、HTML 、CSS/BFC.md>); [Canvas](<../../raw/2、HTML 、CSS/Canvas.md>); [CSS 垂直居中方案](<../../raw/2、HTML 、CSS/CSS 垂直居中方案.md>); [CSS 规范](<../../raw/2、HTML 、CSS/CSS 规范.md>); [CSS](<../../raw/2、HTML 、CSS/CSS.md>); [CSS3 新特性](<../../raw/2、HTML 、CSS/CSS3 新特性.md>); [Flex 布局](<../../raw/2、HTML 、CSS/Flex 布局.md>); [img标签](<../../raw/2、HTML 、CSS/img标签.md>); [meta 标签的作用](<../../raw/2、HTML 、CSS/meta 标签的作用.md>); [BOM](<../../raw/2022-10-05 browser/BOM.MD>); [DOM](<../../raw/2022-10-05 browser/DOM.md>)
> Updated: 2026-09-16

## Overview

这一页整理 HTML 标签与资源引用、CSS 布局和规范、DOM 与 BOM。重点是把页面结构、样式计算和浏览器对象模型放在同一套页面模型里理解。

## HTML 与资源

- 相对路径和绝对路径决定资源的解析基准。
- `img`、`meta` 和页面资源标签影响加载、SEO、响应式和设备行为。
- Iframe 原始文件为空，已作为无内容资料记录，不生成独立知识页。

## CSS 基础与布局

- 盒模型决定元素尺寸、内边距、边框和外边距的计算方式。
- BFC 用于理解外边距折叠、浮动包含和独立布局上下文。
- Flex 适合一维布局，垂直居中等方案应结合父容器尺寸和溢出处理。
- CSS 规范关注命名、作用域、复用和维护成本。
- CSS3 新特性和 Canvas 笔记分别覆盖样式能力和绘图能力。

## DOM 与 BOM

- DOM 把文档表示为可编程的节点树，用于查询、修改和监听页面结构。
- BOM 提供窗口、导航、位置、历史记录和浏览器环境相关能力。
- DOM 操作应关注批量更新、重排成本和事件委托。

## See Also

- [浏览器渲染与前端性能](../browser/performance.md)
- [浏览器架构与网络请求](../browser/architecture-and-network.md)