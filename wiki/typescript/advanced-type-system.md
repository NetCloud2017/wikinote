# TypeScript 高级类型系统

> Sources: 本地笔记, Unknown
> Raw: [1.TS 泛型](<../../raw/TypeScript/1.TS 泛型.md>); [10. 高级类型 Omit](<../../raw/TypeScript/10. 高级类型 Omit.md>); [20、TS 型变、协变](<../../raw/TypeScript/20、TS 型变、协变.md>); [4、infer1](<../../raw/TypeScript/4、infer1.md>); [5、TS  高级 type  类型extract](<../../raw/TypeScript/5、TS  高级 type  类型extract.md>); [6、exclude 类型](<../../raw/TypeScript/6、exclude 类型.md>); [7、异步数据扁平化 Map](<../../raw/TypeScript/7、异步数据扁平化 Map.md>); [8、Pick ](<../../raw/TypeScript/8、Pick .md>); [9.Partial+Required+ReadOnly](<../../raw/TypeScript/9.Partial+Required+ReadOnly.md>); [常用工具类 Exclude Extract Parameters NonNullable ReturnTypeextends 操作符](<../../raw/TypeScript/常用工具类 Exclude Extract Parameters NonNullable ReturnTypeextends 操作符.md>); [交叉类型](<../../raw/TypeScript/交叉类型.md>); [类型断言1](<../../raw/TypeScript/类型断言1.md>); [类型守卫](<../../raw/TypeScript/类型守卫.md>); [模板字面量类型](<../../raw/TypeScript/模板字面量类型.md>); [索引签名](<../../raw/TypeScript/索引签名.md>); [易错的取值方式](<../../raw/TypeScript/易错的取值方式.md>); [映射类型](<../../raw/TypeScript/映射类型.md>); [any 类型和 unknown 类型](<../../raw/TypeScript/any 类型和 unknown 类型.md>); [infer ](<../../raw/TypeScript/infer .md>); [keyof 操作符](<../../raw/TypeScript/keyof 操作符.md>); [Omit 用法](<../../raw/TypeScript/Omit 用法.md>); [Pick 工具类型](<../../raw/TypeScript/Pick 工具类型.md>); [ts新特性](<../../raw/TypeScript/ts新特性.md>); [type 和 interface](<../../raw/TypeScript/type 和 interface.md>); [typeof  操作符](<../../raw/TypeScript/typeof  操作符.md>)
> Updated: 2026-09-16

## Overview

这一页整理泛型、条件类型、映射类型、模板字面量类型、类型守卫、工具类型和型变。核心思路是让类型从一个已知形状推导出新的约束，而不是重复手写类型。

## 泛型与类型操作

- 泛型让函数、类和类型别名复用同一套逻辑，同时保留输入与输出之间的关系。
- `keyof` 提取对象键，`typeof` 从值得到类型，二者常用于构建联动类型。
- 索引签名用于描述键类型相同但名称未知的对象。

## 条件类型与推导

- 条件类型根据类型关系选择分支，`infer` 在匹配过程中提取内部类型。
- `Exclude`、`Extract`、`NonNullable`、`Parameters`、`ReturnType` 等工具类型用于筛选、提取或转换现有类型。
- `Pick`、`Omit`、`Partial`、`Required`、`Readonly` 用于从已有对象类型派生新形状。
- 异步数据扁平化、映射类型和模板字面量类型展示了类型级编程的组合方式。

## 类型收窄与组合

- 类型守卫帮助运行时逻辑把联合类型缩小到具体分支。
- 类型断言只表达开发者判断，不会生成运行时转换。
- 交叉类型用于合并约束，但冲突属性可能形成不可表示的类型。
- 型变描述复杂类型内部对子类型关系的影响。

## 使用边界

高级类型应以可读性和错误提示质量为前提。过度推导会让类型难以维护，并增加编译成本。

## See Also

- [TypeScript 语言基础](language-foundations.md)
- [TypeScript 函数与构造函数重载](overloads.md)