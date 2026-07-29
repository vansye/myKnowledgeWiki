---
type: knowledge
title: Cookie 和 Session
created: 2026-07-24T00:00:00
updated: 2026-07-24T00:00:00
tags: [JavaWeb, 会话, 入门]
source:
conclusion:
---

## 详细

### 概念

Cookie 和 Session 都用于保存会话状态。HTTP 本身是无状态协议，同一个用户的多次请求默认不会自动关联，因此需要借助会话技术保存登录状态、用户标识等信息。它们常用于登录、购物车和权限控制。


### 重点

- Cookie 保存在浏览器端，Session 保存在服务器端
- Cookie 适合存放少量数据，Session 更适合保存敏感状态
- 登录校验、购物车、验证码校验中很常见
- 三种会话技术的完整对比见 会话技术，JWT 登录校验实现见 Filter过滤器技术