# JavaScript 模块化与设计模式

> Sources: 本地笔记, Unknown
> Raw: [Commonjs、AMD、CMD、ESModule、UMD模块化](<../../raw/1、JavaScript/Commonjs、AMD、CMD、ESModule、UMD模块化.md>); [CSS Modules](<../../raw/1、JavaScript/CSS Modules.md>); [js砂箱](<../../raw/1、JavaScript/js砂箱.md>); [Commonjs](<../../raw/1、JavaScript/模块化/Commonjs.md>); [发布订阅模式和观察者模式](<../../raw/1、JavaScript/设计模式/发布订阅模式和观察者模式.md>); [设计模式与实践](<../../raw/1、JavaScript/设计模式/设计模式与实践.md>); [IoC and DI](<../../raw/1、JavaScript/设计模式/IoC and DI.md>); [模块化详解](<../../raw/2022-10-04 Module 重点/模块化详解.md>); [index](<../../raw/2022-10-29 ECMAJavaScript 规范/index.md>); [index](<../../raw/2022-10-30 ECMA 规范/index.md>)
> Updated: 2026-09-16

## Overview

这一页整理 JavaScript 模块化、常见设计模式、依赖倒置、沙箱和 CSS Modules。重点不是记住模式名称，而是理解模块边界、依赖方向、状态传播和隔离机制。

## 模块化

- CommonJS、AMD、CMD、ES Module 和 UMD 面向的运行环境、加载时机和静态分析能力不同。
- ES Module 使用静态导入导出，便于构建工具进行依赖分析和 tree shaking。
- 模块化专题强调把代码拆分、依赖管理、构建和运行加载视为同一套工程问题。
- ECMAScript 规范笔记可作为语法与标准行为的补充索引。

## 设计模式

- 发布订阅模式与观察者模式都能实现事件通知，但发布者和订阅者之间是否直接持有引用是重要区别。
- 常用设计模式笔记覆盖创建型、结构型和行为型模式的应用场景。
- IoC 与 DI 通过反转依赖控制方向，降低模块间耦合，并常用于框架与服务层组织。
- 设计模式应与具体问题匹配，避免为了模式而增加抽象层。

## 隔离与样式作用域

- JavaScript 沙箱通过作用域、代理或执行环境隔离不可信或独立的代码。
- CSS Modules 把类名映射到局部作用域，降低全局样式冲突。
- 样式与脚本的模块化都应服务于边界清晰和可维护性。

## See Also

- [前端构建与工具链](../frontend-engineering/build-and-toolchain.md)
- [TypeScript 装饰器与依赖注入](../typescript/decorators-and-dependency-injection.md)