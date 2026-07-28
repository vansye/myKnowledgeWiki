---
type: 学习笔记
title: API接口封装
created: 2026-07-28
updated: 2026-07-28
tags:
  - Axios
  - 请求拦截器
  - 响应拦截器
  - API封装
  - 前后端交互
subject: JavaWeb
description: 详细讲解前端 API 封装的完整实现，包括 Axios 实例创建、请求/响应拦截器原理、分模块组织和统一导出机制。
---

> 本笔记以 Tlias 管理系统前端为例，系统讲解前端 API 接口封装的完整实现方案。从 Axios 实例创建到拦截器配置，从分模块组织到统一导出，每个细节都深入剖析。

## 目录

- [1. 为什么要封装 API](#1-为什么要封装-api)
- [2. Axios 实例创建](#2-axios-实例创建)
  - [2.1 基础配置](#21-基础配置)
  - [2.2 baseURL 设置](#22-baseurl-设置)
- [3. 请求拦截器](#3-请求拦截器)
  - [3.1 Token 自动注入](#31-token-自动注入)
  - [3.2 拦截器原理](#32-拦截器原理)
- [4. 响应拦截器](#4-响应拦截器)
  - [4.1 数据剥离](#41-数据剥离)
  - [4.2 错误处理](#42-错误处理)
  - [4.3 401 自动处理](#43-401-自动处理)
- [5. 分模块 API 组织](#5-分模块-api-组织)
  - [5.1 模块划分原则](#51-模块划分原则)
  - [5.2 各模块示例](#52-各模块示例)
- [6. 统一导出入口](#6-统一导出入口)
- [7. 在组件中使用](#7-在组件中使用)
- [8. 前后端数据约定](#8-前后端数据约定)
- [9. 完整工作流程](#9-完整工作流程)
- [10. 常见问题](#10-常见问题)
- [小结](小结)

---

## 1. 为什么要封装 API

在前端项目中直接使用 `axios` 进行请求会带来以下问题：

| 问题 | 说明 |
|------|------|
| **重复配置** | 每个文件都要 baseURL、超时设置、拦截器 |
| **Token 重复添加** | 每个请求都要手动添加 token 到 header |
| **错误处理分散** | 每个 catch 块都要写错误提示 |
| **代码冗余** | 相同的请求逻辑重复编写 |
| **维护困难** | 修改 baseURL 或拦截器需要改多处 |

**封装方案的好处**：

- ✅ **集中配置**：Axios 实例统一配置，修改一处全局生效
- ✅ **自动注入**：请求拦截器自动添加 Token，无需手动设置
- ✅ **统一错误处理**：响应拦截器统一处理错误提示和 401 跳转
- ✅ **代码复用**：业务层只需关注接口方法，无需关心请求细节
- ✅ **易于维护**：清晰的模块划分，职责单一

## 2. Axios 实例创建

`src/api/http.js` 是 API 封装的核心，创建 Axios 实例并配置拦截器。

```javascript
// src/api/http.js
import axios from 'axios'
import { ElMessage, ElMessageBox } from 'element-plus'

# 创建 Axios 实例
const api = axios.create({
  baseURL: '/api',   # 所有请求自动拼接此前缀
  timeout: 10000,    # 请求超时 10 秒
  withCredentials: false  # 默认不发送 Cookie
})
```

### 2.1 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `baseURL` | `/api` | 后端 API 的根路径，所有请求自动拼接 |
| `timeout` | `10000` | 毫秒数，请求超过 10 秒自动取消 |
| `withCredentials` | `false` | 是否携带 Cookie，后端认证通常不用 |

**为什么 baseURL 设为 `/api`？**
- 后端接口通常以 `/api` 开头（如 `/api/login`、`/api/dept/list`）
- 前端设置 baseURL 后，发起 `http.get('/login')` 实际请求 `/api/login`
- 开发期配合 Vite 代理可避免跨域问题

### 2.2 baseURL 设置与跨域

开发环境中，`baseURL: '/api'` 需要配合 Vite 的代理配置，将 `/api` 转发到后端端口：

```javascript
// vite.config.js (如有)
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',  # 后端端口
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
}
```

这样在开发时，前端访问 `http://localhost:5173/api/login` 会被代理到 `http://localhost:8080/api/login`，避免跨域问题。

## 3. 请求拦截器

请求拦截器在请求发出前执行，可用于统一添加请求头、修改配置等。

```javascript
# 请求拦截器
api.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')   # 从本地存储获取 Token
    if (token) {
      config.headers.token = token                # 添加到请求头
      # 或者：config.headers.Authorization = 'Bearer ' + token
    }
    return config
  },
  error => {
    # 请求失败时（如网络错误）的处理
    console.error('请求拦截器错误:', error)
    return Promise.reject(error)
  }
)
```

### 3.1 Token 自动注入

**工作流程**：

```
前端发起 http.post('/login', { username, password })
  ↓
[请求拦截器] → 检查 localStorage.getItem('token')
  ↓
如果没有 token（如登录页），不添加 Header
  ↓
如果有 token，config.headers.token = 'xxx'
  ↓
HTTP 请求发出，带上 token Header
  ↓
后端验证 token，返回结果
```

**为什么用 localStorage 存储 Token？**
- 本地存储持久化，刷新页面后仍有效
- 简单易用，无需复杂管理
- 配合请求拦截器自动注入
- 登出时手动清除

### 3.2 拦截器原理

Axios 拦截器本质上是一个**中间件链**：

```
请求 → [请求拦截器 1] → [请求拦截器 2] → ... → 发送请求
        ↓                      ↓
    修改 config         修改 config
        ↓                      ↓
响应 ← [响应拦截器 1] ← [响应拦截器 2] ← ... ← 返回响应
        ↓                     ↓
   处理 response         处理 error
```

**请求拦截器**：在请求发送前 intercept，可以修改 `config`（如添加 header、修改 URL）。

**响应拦截器**：在响应返回后 intercept，可以处理 `response` 或 `error`。

## 4. 响应拦截器

响应拦截器在请求返回后执行，统一处理响应数据、错误和特殊状态码。

```javascript
# 响应拦截器
api.interceptors.response.use(
  response => {
    const data = response.data
    # 后端返回格式：{code: 1, msg: '成功', data: 业务数据}
    
    if (data.code === 1) {
      # 成功：直接返回业务数据，调用方无需取 data.data
      return data.data
    } else {
      # 失败：弹出错误消息，抛出异常
      ElMessage.error(data.msg || '操作失败')
      return Promise.reject(new Error(data.msg || '操作失败'))
    }
  },
  error => {
    # 网络错误或非 2xx 状态码的处理
    
    # 401 未认证：跳转到登录页
    if (error.response && error.response.status === 401) {
      ElMessage.warning('请先登录')
      setTimeout(() => {
        window.location.href = '/login'   # 直接跳转，不经过路由
      }, 1500)
      return Promise.reject()   # 阻止错误传播
    }

    # 其他错误
    ElMessage.error(error.message || '系统异常')
    return Promise.reject(error)
  }
)
```

### 4.1 数据剥离

**问题**：后端返回 `{code: 1, msg: '成功', data: {userId: 1, name: '张三'}}`，如果不用拦截器，前端每次都要写 `result.data.data`。

**解决方案**：响应拦截器剥离一层，直接返回业务数据：

```javascript
# 拦截器前：result = await http.get('/user/1') → {code:1, msg:'成功', data:{...}}
# 拦截器后：result = await http.get('/user/1') → {userId: 1, name: '张三'} (直接返回 data.data)
```

这样在业务代码中可以直接使用返回的数据，不需要再取 `.data` 层。

### 4.2 错误处理

**错误处理策略**：

1. **业务错误**（code ≠ 1）：通过 `ElMessage.error()` 提示用户，并抛出异常供 catch 捕获
2. **网络错误**：error.response 可能为 undefined，需要安全访问
3. **超时错误**：网络断开或后端无响应，提示"请求超时"

### 4.3 401 自动处理

**场景**：Token 过期或无效，后端返回 401 状态码。

**处理流程**：
1. 响应拦截器捕获 401 错误
2. 弹出"请先登录"提示
3. 1.5 秒后自动跳转到登录页 (`/login`)
4. 阻止错误继续传播（`return Promise.reject()`），避免页面 catch 重复处理

**为什么用 `window.location.href` 而不是 `router.push`？**
- 401 可能在任何请求中触发，包括非 Vue 路由控制的场景
- 直接跳转确保即使页面未完全加载也能跳转到登录页
- 使用 Hash 模式时，`/login` 会自动带上 `#`

## 5. 分模块 API 组织

请求实例封装好后，按业务领域分模块封装具体接口：`src/api/` 目录下每个文件对应一个业务模块。

### 5.1 模块划分原则

| 原则 | 说明 | 示例 |
|------|------|------|
| **按业务域划分** | 每个模块对应一个后端 Controller | `auth.js` → 登录登出，`dept.js` → 部门管理 |
| **职责单一** | 一个文件只做一件事 | `emp.js` 只包含员工相关接口 |
| **命名清晰** | 文件名对应业务 | `student.js`、`clazz.js`、`log.js` |
| **统一导出** | `index.js` 一站式导入 | `import { empApi } from '@/api'` |

### 5.2 各模块示例

#### auth.js (登录/登出)

```javascript
// src/api/auth.js
import http from './http'

export const loginApi = {
  # POST /login
  login: (data) => http.post('/login', data),

  # GET /logout
  logout: () => {
    # 清除本地存储
    localStorage.removeItem('token')
    localStorage.removeItem('userInfo')
    # 调用登出接口
    return http.get('/logout')
  }
}
```

#### dept.js (部门管理)

```javascript
// src/api/dept.js
import http from './http'

export const deptApi = {
  # GET /department/list
  list: () => http.get('/department/list'),

  # POST /department/add
  add: (data) => http.post('/department/add', data),

  # PUT /department/update
  update: (data) => http.put('/department/update', data),

  # DELETE /department/delete?ids=1,2,3
  delete: (ids) => http.delete('/department/delete', { params: { ids } })
}
```

#### emp.js (员工管理) - 含分页

```javascript
// src/api/emp.js
import http from './http'

export const empApi = {
  # GET /employee/list?page=1&pageSize=10&name=张三
  list: (params) => http.get('/employee/list', { params }),

  # GET /employee/1
  getById: (id) => http.get(`/employee/${id}`),

  # POST /employee/add
  add: (data) => http.post('/employee/add', data),

  # PUT /employee/update
  update: (data) => http.put('/employee/update', data),

  # DELETE /employee/delete?ids=1,2
  delete: (ids) => http.delete('/employee/delete', { params: { ids } })
}
```

其他模块（clazz.js、student.js、log.js、report.js）结构类似，按后端接口规范封装。

## 6. 统一导出入口

为了方便全局导入，`src/api/index.js` 将所有模块的 API 统一导出：

```javascript
// src/api/index.js
# 从各模块导出接口对象
export { loginApi } from './auth'
export { deptApi } from './dept'
export { empApi } from './emp'
export { classApi } from './clazz'   # 注意：clazz 是保留字，导出时用 classApi
export { studentApi } from './student'
export { reportApi } from './report'
export { logApi } from './log'
```

**使用方式**：

```vue
<script setup>
# 方式1：按模块导入（推荐，按需要导入）
import { loginApi } from '@/api/auth'
import { deptApi } from '@/api/dept'

# 方式2：统一导入（全部导入，适合小项目）
import * as api from '@/api'
# 使用时：api.loginApi.login(...)，api.deptApi.list()
</script>
```

**推荐使用方式1**：按需导入可以减少打包体积，Vite 会自动分析依赖并拆分代码块。

## 7. 在组件中使用

组件中通过 `@/api` 别名导入 API，直接调用方法即可。

```vue
<script setup>
import { ref } from 'vue'
import { useUserStore } from '@/stores/user'
import { deptApi } from '@/api/dept'

const userStore = useUserStore()
const departments = ref([])

# 方法1：直接调用 API
async function loadDepartments() {
  departments.value = await deptApi.list()   # 拦截器已剥离数据层，直接返回数组
}

# 方法2：通过 Store 调用（推荐，集中管理）
await userStore.loadDepartments()   # Store 内部调用 deptApi.list()
</script>
```

**在 Store 中使用**：Store 是 API 调用的最佳位置，因为 Store 天然就是集中管理业务逻辑的地方。

```javascript
// stores/user.js
export const useUserStore = defineStore('user', {
  state: () => ({ departments: [] }),
  actions: {
    async loadDepartments() {
      this.departments = await deptApi.list()   # 直接调用 API
    }
  }
})
```

## 8. 前后端数据约定

前端 API 封装依赖后端返回的固定格式，约定如下：

**成功响应**：
```json
{
  "code": 1,
  "msg": "成功",
  "data": {   # 业务数据
    "id": 1,
    "name": "张三"
  }
}
```

**失败响应**：
```json
{
  "code": -1,
  "msg": "用户名或密码错误",
  "data": null
}
```

**分页响应**：
```json
{
  "code": 1,
  "msg": "成功",
  "data": {
    "rows": [    # 员工数组
      { "id": 1, "name": "张三" },
      { "id": 2, "name": "李四" }
    ],
    "total": 100,   # 总条数
    "page": 1,      # 当前页
    "pageSize": 10  # 每页条数
  }
}
```

**前端约定**：
- 响应拦截器判断 `code === 1` 为成功，直接返回 `data`
- `code ≠ 1` 时弹出错误消息并抛异常
- 分页数据中 `rows` 是员工列表，`total` 是总条数

## 9. 完整工作流程

```
用户操作（点击按钮）
  ↓
前端调用：await deptApi.list()
  ↓
[请求拦截器] → 检查是否有 Token → 添加到 config.headers
  ↓
HTTP GET /api/dept/list  (baseURL + 路径)
  ↓
后端处理，返回 {code:1, msg:"成功", data:[{...}]}
  ↓
[响应拦截器] → code=1 → 返回 data.data (直接返回数组)
  ↓
前端接收：const list = await deptApi.list()  # list 就是数组，无需取 .data
  ↓
前端处理数据（渲染表格等）
```

### 错误流程

```
用户操作 → 调用 API
  ↓
[请求拦截器] → 正常通过
  ↓
HTTP 请求失败 (网络中断/后端500)
  ↓
[响应拦截器 error] → 检查错误类型
  ├─ 401 → 提示"请先登录" → 1.5秒后跳转 /login
  └─ 其他 → ElMessage.error("系统异常") → 抛出异常
  ↓
前端 catch 块处理 (可选，拦截器已处理)
```

## 10. 常见问题

### Q1: 为什么响应拦截器要返回 `Promise.reject()`？

**原因**：如果只返回 `data.data` 而不 reject 错误，错误会被视为成功，catch 块不会触发，错误被吞掉。

```javascript
# 错误做法（错误被吞掉）：
return data.data   # code≠1时也返回，catch不触发

# 正确做法：
return Promise.reject(new Error(msg))   # 触发 catch
```

### Q2: 拦截器剥了几层，页面就要少取几层？

**重要原则**：拦截器和调用方要**保持一致**。如果拦截器剥了一层（返回 `data.data`），调用方就直接用结果，不能再写 `result.data`。

**典型 Bug**：改了拦截器剥层，但页面代码没改，出现"接口返回200但表格为空"的现象。

### Q3: 为什么登出时要同时清除 localStorage 和调用后端接口？

**原因**：
- `localStorage`：清除前端保存的 Token，下次请求拦截器不会加 Token
- 后端 `/logout` 接口：登出后端 Session/Token，防止 Token 被复用

两者配合确保前后端都退出。

### Q4: 拦截器中访问 `error.response` 为什么要用 optional chaining？

**原因**：网络错误时（如断网），`error.response` 为 `undefined`，直接取 `.status` 会报错。

```javascript
# 安全写法
if (error.response?.status === 401) { ... }

# 错误写法（断网时会崩溃）
if (error.response.status === 401) { ... }
```

---

## 小结

API 封装的核心思想是**集中处理重复逻辑，让业务代码更简洁**：

1. **Axios 实例**：创建带 baseURL 和超时配置的实例
2. **请求拦截器**：自动注入 Token，统一请求头设置
3. **响应拦截器**：剥离数据层、统一错误处理、401 自动跳登录
4. **分模块组织**：按业务域拆分接口，职责清晰
5. **统一导出**：`index.js` 一站式导入

掌握这套 API 封装模式，可以高效处理前后端交互，避免重复代码，提高可维护性。

关联知识点：
- [[学习/JavaWeb/知识条目/前端/Vue Router路由守卫]] 权限控制配合
- [[学习/JavaWeb/知识条目/前端/Pinia状态管理]] Store 中调用 API
- [[学习/JavaWeb/知识条目/前端/script setup语法]] 组件中使用
- [[响应式对象]] 响应式数据处理