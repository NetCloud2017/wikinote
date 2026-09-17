# 跨域与请求安全

> Sources: 本地笔记, Unknown
> Raw: [前端跨越的方式总结](<../../raw/1、JavaScript/跨域/前端跨越的方式总结.md>); [JSONP](<../../raw/1、JavaScript/跨域/JSONP.MD>)
> Updated: 2026-09-16

## Overview

这一页整理浏览器跨域来源限制、前端跨越方式总结和 JSONP。

## 同源策略

- 浏览器根据协议、域名和端口判断是否同源。
- 跨域限制保护的是浏览器中的资源和用户凭证，不等同于服务端之间的网络限制。
- 请求是否发送、响应是否可读、是否携带凭证由不同规则控制。

## 常见方案

- CORS 通过服务端响应头声明允许的来源、方法、头部和凭证。
- JSONP 利用脚本标签加载能力，但只适合特定请求方式，并存在安全风险。
- 开发代理可以把浏览器请求转发到目标服务，但生产环境仍需正确配置跨域策略。
- 方案选择应结合实际域名、认证方式和安全边界。

## See Also

- [HTTP、HTTPS 与缓存](http-protocols.md)
- [Web 攻击与防护](../security/web-security.md)
- [JavaScript 异步与浏览器 API](../javascript/async-and-platform-apis.md)