---
type: knowledge
title: API 接口封装
created: 2026-07-28
updated: 2026-07-28
tags: [Axios, 请求拦截器, 响应拦截器, 前端]
source: Tlias 管理系统前端
conclusion: 采用 Axios 实例 + 拦截器模式封装 API：请求拦截器自动加 Token，响应拦截器统一处理错误和 401 跳转，业务层按模块拆分（auth/dept/emp 等），最后统一导出。
---

## 详细

### 概念
`src/api/` 目录封装后端接口请求，采用分层模式：`http.js` 创建 Axios 实例并配置拦截器，各业务模块（auth/dept/emp 等）定义具体接口方法，`index.js` 统一导出。

### 重点
- **请求拦截器**：自动从 localStorage 获取 Token 添加到请求头，所有接口无需手动设置
- **响应拦截器**：统一剥离数据层级（直接返回业务数据）、处理错误消息、401 自动跳转登录
- **分模块组织**：按业务域拆分文件，每个文件导出接口对象，职责清晰
- **统一导出**：`api/index.js` 一站式导出，组件通过 `@/api/auth` 等路径导入
