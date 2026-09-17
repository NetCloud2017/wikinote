# JavaScript 语言核心

> Sources: 本地笔记, Unknown
> Raw: [1、非全等比较 == ](<../../raw/1、JavaScript/1、非全等比较 == .md>); [10、浅拷贝和深拷贝](<../../raw/1、JavaScript/10、浅拷贝和深拷贝.md>); [4、for 相关](<../../raw/1、JavaScript/4、for 相关.md>); [5、Proxy](<../../raw/1、JavaScript/5、Proxy.md>); [6、Reflect 和 Object.definedProperty](<../../raw/1、JavaScript/6、Reflect 和 Object.definedProperty.md>); [7、Array](<../../raw/1、JavaScript/7、Array.md>); [8、String](<../../raw/1、JavaScript/8、String.md>); [构造函数、 封装、继承、多态](<../../raw/1、JavaScript/构造函数、 封装、继承、多态.md>); [实现一个new 函数](<../../raw/1、JavaScript/实现一个new 函数.md>); [原型链](<../../raw/1、JavaScript/原型链.md>); [字符串String处理API](<../../raw/1、JavaScript/字符串String处理API.md>); [Class](<../../raw/1、JavaScript/Class.md>); [JavaScript  柯里化](<../../raw/1、JavaScript/JavaScript  柯里化.md>); [jsLoop](<../../raw/1、JavaScript/jsLoop.md>); [Map、Set、Weakmap、Weakset](<../../raw/1、JavaScript/Map、Set、Weakmap、Weakset.md>); [Object 常用API](<../../raw/1、JavaScript/Object 常用API.md>); [Set、Map、 weakSet、weakMap](<../../raw/1、JavaScript/Set、Map、 weakSet、weakMap.md>); [Smybol](<../../raw/1、JavaScript/Smybol.md>); [运算符](<../../raw/1、JavaScript/js运算/运算符.md>); [JavaScript 数值相关](<../../raw/1、JavaScript/js运算/JavaScript 数值相关.md>)
> Updated: 2026-09-16

## Overview

这一页汇总 JavaScript 的语言基础、对象模型、集合类型、运算规则和常用内置 API。原始笔记覆盖从值比较、深浅拷贝、循环和数值运算，到原型链、类、继承、`Proxy`、`Reflect` 以及数组、字符串、对象、`Map`、`Set`、`Symbol` 等主题。

## 核心模型

### 值、比较与拷贝

- `==`、`!=` 与严格比较涉及类型转换规则，应优先理解隐式转换发生的条件。
- 浅拷贝只复制第一层引用；深拷贝需要处理嵌套对象、循环引用和特殊对象类型。
- 数值笔记涵盖整数、浮点数、精度和常见运算差异。

### 控制流与函数

- `for`、数组遍历和对象遍历需要区分可枚举属性、原型属性以及中断循环的方式。
- 柯里化通过拆分参数，把多参数函数转化为一系列单参数函数。
- `new` 的实现需要建立原型关系、执行构造函数，并处理构造函数返回值。

### 对象模型

- 原型链决定属性查找和继承行为。
- 类语法是原型机制之上的抽象，封装、继承和多态需要结合构造函数一起理解。
- `Proxy` 可以拦截对象操作，`Reflect` 提供与默认对象操作对应的反射方法，两者常配合使用。

### 集合与常用 API

- `Map`、`Set`、`WeakMap`、`WeakSet` 在键类型、引用保持和遍历能力上不同。
- 数组、字符串和对象 API 是日常开发的基础工具，阅读时应结合返回值和是否修改原对象判断适用场景。
- `Symbol` 常用于唯一键和内置协议扩展。

## 使用边界

这些笔记以速查为主，部分 API 页面很短，适合作为索引，不应替代运行时行为验证和标准文档。涉及 `Object.defineProperty`、代理陷阱或类型转换时，应优先用具体示例确认浏览器行为。

## See Also

- [JavaScript 异步与浏览器 API](async-and-platform-apis.md)
- [JavaScript 正则与算法](regex-and-algorithms.md)
- [JavaScript 模块化与设计模式](modularity-and-patterns.md)