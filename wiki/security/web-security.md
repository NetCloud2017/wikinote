# Web 攻击与防护

> Sources: 本地笔记, Unknown
> Raw: [CSRF攻击](<../../raw/安全相关/CSRF攻击.md>); [XSS攻击](<../../raw/安全相关/XSS攻击.md>)
> Updated: 2026-09-16

## Overview

这一页整理 XSS 和 CSRF 两类 Web 安全风险及其防护思路。

## XSS

- XSS 把攻击者控制的脚本注入页面并在用户浏览器中执行。
- 常见防护包括输出编码、输入校验、富文本清理和内容安全策略。
- 不应直接拼接不可信内容到 HTML、脚本或属性上下文。

## CSRF

- CSRF 利用浏览器自动携带凭证的行为，让用户在不知情时发起请求。
- 常见防护包括 CSRF Token、SameSite Cookie、来源校验和关键操作二次确认。
- 防护需要结合认证方式、跨域策略和接口方法设计。

## 通用原则

- 安全边界应覆盖输入、存储、输出、传输和权限校验。
- 客户端校验改善体验，服务端校验才是最终防线。
- 敏感信息不应进入日志、错误页面或前端公开产物。

## See Also

- [HTTP、HTTPS 与缓存](../web/http-protocols.md)
- [跨域与请求安全](../web/cross-origin.md)
- [Node.js 服务开发](../nodejs/service-development.md)