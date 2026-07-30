---
type: knowledge
title: Filter 过滤器
created: 2026-07-24T00:00:00
updated: 2026-08-30T00:00:00
tags: [JavaWeb, 过滤器, 入门]
source:
conclusion:
---

## 详细

### 概念

Filter 是 Java Web 中对请求和响应进行统一处理的组件，常见于登录校验、编码处理、日志记录、权限检查等场景。它在请求到达 Servlet 之前或响应返回之前执行，适合做全局性的统一处理。[[Interceptor拦截器]] 是 Spring 框架中类似的机制，但更贴近 Controller 层面，两者常被对比学习。


### 重点

- 可以拦截一批请求，不是只处理单个 Servlet
- 常用于统一前置处理和后置处理
- 需要在 web.xml 或注解中配置映射关系
- 详细讲解见 Filter过滤器技术（含登录校验代码示例、FilterChain、Filter 与 Interceptor 对比）