# HTTP、HTTPS 与缓存

> Sources: 本地笔记, Unknown
> Raw: [1、强缓存和协商缓存](<../../raw/HTTP、及请求库/1、强缓存和协商缓存.md>); [Axios](<../../raw/HTTP、及请求库/Axios.md>); [HTTP 发展史](<../../raw/HTTP、及请求库/HTTP 发展史.md>); [HTTPS](<../../raw/HTTP、及请求库/HTTPS.md>)
> Updated: 2026-09-16

## Overview

这一页整理 HTTP 发展、HTTPS、强缓存与协商缓存，以及 Axios 请求库。

## HTTP 演进

- HTTP 不同版本在连接复用、头部压缩、多路复用和传输效率上持续演进。
- 请求方法、状态码、头部和消息体构成接口通信的基本语义。
- 缓存、连接管理和内容协商会直接影响性能与一致性。

## HTTPS

- HTTPS 通过 TLS 提供加密、完整性和身份认证。
- 证书验证和信任链是安全连接的基础。
- 部署 HTTPS 需要处理证书、协议版本、重定向和安全头部。

## 缓存

- 强缓存由响应头控制，在有效期内可以不向服务器重新验证。
- 协商缓存通过条件请求判断资源是否变化。
- 缓存策略需要区分静态资源、接口数据和用户私有数据。

## 请求库

- Axios 封装请求、响应、拦截器和错误处理。
- 封装请求库时应统一基础地址、认证、超时、取消和错误格式。
- 请求库不能替代接口契约、服务端校验和安全策略。

## See Also

- [浏览器架构与网络请求](../browser/architecture-and-network.md)
- [跨域与请求安全](cross-origin.md)
- [Web 攻击与防护](../security/web-security.md)