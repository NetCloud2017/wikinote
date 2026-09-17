# TypeScript 语言基础

> Sources: 本地笔记, Unknown
> Raw: [11. 复习前的](<../../raw/TypeScript/11. 复习前的.md>); [class 类型](<../../raw/TypeScript/class 类型.md>); [data class](<../../raw/TypeScript/data class.md>); [OOP-继承](<../../raw/TypeScript/OOP-继承.md>); [tsconfig.json](<../../raw/TypeScript/tsconfig.json.md>); [TypeScript 简介](<../../raw/TypeScript/TypeScript 简介.md>); [TypeScript Q&A](<../../raw/TypeScript/TypeScript Q&A.md>); [TypeScript](<../../raw/2022-10-22 TS/TypeScript.md>)
> Updated: 2026-09-16

## Overview

这一页汇总 TypeScript 的基本定位、类型声明、类、配置文件、常见问答和函数式编程相关内容。目标是把类型系统如何描述值、函数、类和模块先建立起来。

## 类型系统定位

- TypeScript 在 JavaScript 之上提供静态类型检查，类型在编译后不会保留为运行时逻辑。
- 类型声明描述数据形状和约束，不应被当作运行时校验。
- `any` 会关闭大部分类型检查，`unknown` 要求使用前先缩小类型。
- `type` 与 `interface` 都能描述对象，但在扩展、合并和组合能力上存在差异。

## 类与配置

- 类可以描述字段、构造函数、方法和访问控制；继承笔记进一步覆盖抽象和面向对象组织。
- `data class`、单例等写法体现类型声明与运行时结构之间的对应关系。
- `tsconfig.json` 控制编译目标、模块系统、严格检查和输出行为，是类型行为的重要入口。

## 问答与使用边界

- 泛型、对象替代类和异步数据等问答适合作为具体问题的排查入口。
- 函数式编程笔记强调纯函数、组合和不可变数据，但类型系统本身不保证运行时纯度。
- 当类型断言掩盖了真实数据时，错误会从编译期转移到运行时。

## See Also

- [TypeScript 高级类型系统](advanced-type-system.md)
- [TypeScript 函数与构造函数重载](overloads.md)
- [TypeScript 装饰器与依赖注入](decorators-and-dependency-injection.md)