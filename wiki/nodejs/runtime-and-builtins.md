# Node.js 运行时与内置模块

> Sources: 本地笔记, Unknown
> Raw: [1、nodej框架](<../../raw/nodejs/1、nodej框架.md>); [2、 1_23 node笔记](<../../raw/nodejs/2、 1_23 node笔记.md>); [2.NPM](<../../raw/nodejs/2.NPM.md>); [3、1_24 node 笔记](<../../raw/nodejs/3、1_24 node 笔记.md>); [6、inquirer](<../../raw/nodejs/6、inquirer.md>); [常用的三方库](<../../raw/nodejs/常用的三方库.md>); [NPM 命令](<../../raw/nodejs/NPM 命令.md>); [package.json文件配置](<../../raw/nodejs/package.json文件配置.md>); [path 模块 ](<../../raw/nodejs/path 模块 .md>); [01  事件循环：高性能到底是如何做到的？](<../../raw/nodejs/nodejs 开发实战/01  事件循环：高性能到底是如何做到的？.md>); [Buffer](<../../raw/nodejs/内置包/Buffer.md>); [child_process](<../../raw/nodejs/内置包/child_process.md>); [Dgram数据报](<../../raw/nodejs/内置包/Dgram数据报.md>); [Events事件触发器](<../../raw/nodejs/内置包/Events事件触发器.md>); [FS 文件系统](<../../raw/nodejs/内置包/FS 文件系统.md>); [HTTP1、2模块](<../../raw/nodejs/内置包/HTTP1、2模块.md>)
> Updated: 2026-09-16

## Overview

这一页整理 Node.js 运行时基础、包管理、三方库、框架入口以及常用内置模块。内建模块部分覆盖文件系统、路径、Buffer、事件、HTTP、子进程和数据报。

## 运行时与包管理

- Node.js 在服务端运行 JavaScript，模块系统、包管理、命令行工具和异步 IO 是工程基础。
- npm 命令、`package.json` 和常用三方库决定依赖安装、脚本执行和项目元数据。
- `inquirer` 等库用于命令行交互，框架笔记用于建立服务入口。

## 核心内置模块

- `fs` 提供文件读写、权限、目录和流相关能力。
- `path` 处理跨平台路径拼接、解析和规范化。
- `Buffer` 表示二进制数据，用于文件、网络和编码转换。
- `events` 提供事件触发器，是许多异步模块的基础。
- `http` 与 `http2` 模块负责服务端和客户端网络通信。
- `child_process` 用于创建子进程，需要注意进程生命周期、错误和资源回收。
- `dgram` 面向数据报通信，与 TCP 流式通信的语义不同。

## 使用边界

Node.js API 往往同时提供回调和 Promise 形态，错误处理与资源释放是重点。文件权限、流背压、进程退出和事件监听泄漏需要结合实际场景验证。

## See Also

- [Node.js 服务开发](service-development.md)
- [Node.js 性能与稳定性](performance-and-stability.md)
- [包管理与发布流程](../frontend-engineering/package-management-and-release.md)