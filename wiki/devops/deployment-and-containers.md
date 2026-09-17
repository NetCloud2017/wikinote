# 部署、容器与反向代理

> Sources: 本地笔记, Unknown
> Raw: [1、发布程序](<../../raw/DevOps/1、发布程序.md>); [Docker](<../../raw/DevOps/Docker.md>); [Nginx实现反向代理](<../../raw/Nginx/Nginx实现反向代理.md>)
> Updated: 2026-09-16

## Overview

这一页整理发布程序、Docker 文件技巧和 Nginx 反向代理。重点是从构建产物到服务器运行之间的部署链路。

## 发布与部署

- 发布程序需要经过构建、上传、替换、启动和验证等步骤。
- Putty 与 Xftp 等组合用于远程连接和文件传输。
- 部署脚本应可重复执行，并明确回滚和失败处理方式。

## Docker

- Dockerfile 通过分层构建生成可运行镜像。
- 减少无效层、利用缓存、控制上下文和固定基础环境是常见优化方向。
- 容器化只解决运行环境封装，不自动解决配置、数据和安全问题。

## Nginx 反向代理

- 反向代理代表客户端访问后端服务，可用于统一入口、负载均衡和 TLS。
- 配置需要明确监听端口、域名、上游地址、请求头和超时。
- 代理层还常用于静态资源、缓存、压缩和访问日志。

## See Also

- [Linux 使用基础](../linux/linux.md)
- [Node.js 服务开发](../nodejs/service-development.md)
- [系统、DNS 与 Hosts](../general/system-and-network.md)