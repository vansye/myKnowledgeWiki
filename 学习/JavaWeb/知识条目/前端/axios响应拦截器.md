---
type: knowledge
title: Axios 响应拦截器
created: 2026-07-28
updated: 2026-07-28
tags: [Axios, 拦截器, 前端, HTTP]
source: Tlias 管理系统前端
conclusion: Axios 响应拦截器用于在收到服务器响应后、被 .then 或 .catch 处理前，统一拦截并处理所有响应数据或错误，是前端项目中实现全局错误处理、数据转换及权限校验的核心机制。
---

## 详细

### 概念
Axios 响应拦截器（response interceptors）是一种全局拦截机制，它会在服务器响应返回后、业务代码中的 `then` 或 `catch` 回调执行之前，对响应进行统一预处理。与请求拦截器共同构成 Axios 的核心拦截体系。

### 重点
- **核心作用**：统一处理所有响应的成功与失败情况，包括提取数据、集中处理错误码、隐藏全局 Loading、权限失效跳转等
- **执行时机**：在响应到达 `then` 或 `catch` 之前执行。`axios.interceptors.response.use(成功回调, 失败回调)`
- **响应对象结构**：成功回调接收完整的 `response` 对象（包含 data、status、headers 等），通常在此提取 `response.data` 返回给业务层
- **错误处理**：失败回调接收 `error` 对象，通常在此统一处理 HTTP 状态码（401 跳转登录、500 提示服务器错误）及网络超时
- **链式执行**：多个响应拦截器按添加顺序依次执行，每个拦截器可对处理结果进行链式传递
- **实例隔离**：可为 `axios` 全局实例或 `axios.create()` 创建的独立实例分别添加拦截器，实现不同业务模块的差异化处理
