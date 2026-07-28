---
type: 学习笔记
title: script setup 语法
created: 2026-07-28
updated: 2026-07-28
tags:
  - Vue3
  - Composition API
  - script setup
  - 单文件组件
subject: JavaWeb
description: 详细讲解 Vue3 的 <script setup> 语法糖，包括自动暴露、响应式 API、生命周期、组合式函数等核心特性。
---

> 本笔记以 Tlias 管理系统前端为例，系统讲解 Vue3 的 `<script setup>` 语法糖。从自动暴露机制到生命周期钩子，从 ref/reactive 到组合式函数，每个特性都深入剖析。

## 目录

- [1. script setup 概述](#1-script-setup概述)
- [2. 与传统 <script> 的对比](#2-与传统-script的对比)
  - [2.1 Options API](#21-options-api)
  - [2.2 <script setup> 优势](#22-script-setup-优势)
- [3. 自动暴露机制](#3-自动暴露机制)
  - [3.1 顶层变量自动可用](#31-顶层变量自动可用)
  - [3.2 导入的组件自动可用](#32-导入的组件自动可用)
- [4. 响应式数据](#4-响应式数据)
  - [4.1 ref](#41-ref)
  - [4.2 reactive](#42-reactive)
  - [4.3 模板中的自动解包](#43-模板中的自动解包)
- [5. 生命周期钩子](#5-生命周期钩子)
- [6. 组合式函数](#6-组合式函数)
  - [6.1 路由组合式函数](#61-路由组合式函数)
  - [6.2 Store 组合式函数](#62-store-组合式函数)
- [7. 事件处理](#7-事件处理)
- [8. 完整示例](#8-完整示例)
- [9. 注意事项](#9-注意事项)
- [小结](小结)

---

## 1. script setup 概述

`<script setup>` 是 Vue3 为单文件组件（.vue）提供的语法糖，让组件代码更简洁。它让所有顶层变量和方法自动在模板中可用，无需手动 export。

**基本用法**：

```vue
<script setup>
# 所有顶层声明自动暴露到模板
import { ref } from 'vue'

const count = ref(0)   # 顶层变量，模板中可直接用

function handleClick() { count.value++ }   # 顶层函数，模板中可直接用
</script>

<template>
  <button @click="handleClick">{{ count }}</button>  # 直接使用 count，无需 .value
</template>
```

**核心优势**：
- 无需 `export default`
- 无需手动返回变量（`return { count }`）
- 顶层变量自动在模板中可用
- 体积更小（编译优化）

## 2. 与传统 <script> 的对比

### 2.1 Options API

传统的 Vue2 风格（Options API）：

```vue
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    
    function increment() {
      count.value++
    }
    
    # 必须手动返回，模板才能访问
    return { count, increment }
  }
}
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

**缺点**：
- 需要 `export default`
- `setup()` 函数需要手动返回变量
- 逻辑分散在不同的选项（data、methods、computed 中）

### 2.2 <script setup> 优势

`<script setup>` 的简洁性体现在：

| 方面 | Options API | <script setup> |
|------|------------|---------------|
| **语法** | 需 `export default`、`setup()` | 直接写代码 |
| **暴露** | 手动 `return { ... }` | 自动暴露顶层变量 |
| **代码量** | 多行 | 少行 |
| **TypeScript** | 支持较弱 | 原生支持更好 |
| **性能** | 普通 | 编译优化，更小 |

## 3. 自动暴露机制

### 3.1 顶层变量自动可用

在 `<script setup>` 中声明的所有顶层变量，自动在模板中可用，无需额外操作：

```vue
<script setup>
import { ref } from 'vue'

# 声明 ref
const count = ref(0)

# 声明普通变量
const name = '张三'

# 声明函数
function handleClick() {
  count.value++
}

# 声明对象（需 reactive）
const state = reactive({ age: 25 })
</script>

<template>
  # 直接使用顶层变量
  <div>
    Count: {{ count }}    # 模板中自动解包，无需 .value
    Name: {{ name }}
    Age: {{ state.age }}
    <button @click="handleClick">+1</button>
  </div>
</template>
```

**自动暴露的规则**：
- 顶层声明的 `const/let/var` 变量 → 自动可用
- 顶层声明的函数 → 自动可用
- 顶层声明的对象（通过 `reactive`）→ 属性自动可用
- 导入的 Vue 组件 → 自动可用（如 `<ElButton>`）

### 3.2 导入的组件自动可用

直接导入的 Vue 组件在模板中可以直接使用，无需注册：

```vue
<script setup>
import { ElButton, ElInput } from 'element-plus'   # 导入组件
import ChildComponent from './ChildComponent.vue'   # 导入本地组件
</script>

<template>
  # 直接使用导入的组件
  <ElButton type="primary">保存</ElButton>
  <ElInput v-model="name" placeholder="请输入" />
  <ChildComponent />   # 本地组件直接使用
</template>
```

**原理**：`<script setup>` 编译器会自动分析导入的 Vue 组件，并注册为局部组件。

## 4. 响应式数据

### 4.1 ref

`ref()` 用于创建响应式包装，可以包装基本类型（string、number、boolean）或对象：

```javascript
import { ref } from 'vue'

# 包装基本类型
const count = ref(0)        # 包装数字
const name = ref('张三')    # 包装字符串
const loading = ref(false)  # 包装布尔值

# 访问和修改值（需要 .value）
count.value = 1             # 修改
console.log(count.value)    # 读取
```

**在模板中自动解包**：
- 模板中直接使用 `{{ count }}`，无需 `.value`
- 原因：模板会自动解包 ref，编译时转换为 `count.value`

```vue
<template>
  {{ count }}   # 等价于 {{ count.value }}
</template>
```

### 4.2 reactive

`reactive()` 用于包装对象，创建响应式代理：

```javascript
import { reactive } from 'vue'

const state = reactive({
  age: 25,
  city: '北京',
  hobbies: ['阅读', '游泳']
})

# 直接修改属性（无需 .value）
state.age = 26
state.city = '上海'
```

**特点**：
- 直接修改对象的属性即可触发响应式
- 不支持解构（解构会丢失响应性，需用 `toRefs`）
- 适合包装对象类型的数据

### 4.3 模板中的自动解包

`<script setup>` 的一个大优势是：**模板中直接使用 ref 变量时无需 .value**：

```vue
<script setup>
const count = ref(0)
</script>

<template>
  # 错误写法：{{ count.value }}
  # 正确写法：{{ count }}（Vue 3.2+ 自动解包）
  Count: {{ count }}
  
  # 在指令中
  v-model="count"   # 无需 .value
</template>
```

**例外情况**：在 `<script>` 代码块中（不是模板中）访问 ref 时，仍然需要 `.value`：

```javascript
function increment() {
  count.value++   # 需要 .value
}
```

## 5. 生命周期钩子

`<script setup>` 使用 Vue 3 的组合式生命周期钩子，这些钩子在 `setup()` 阶段执行：

```javascript
import { onMounted, onUpdated, onBeforeUnmount, onMounted } from 'vue'

# 组件挂载后执行（类似 Vue2 的 mounted）
onMounted(() => {
  console.log('组件已挂载')
  # 可在此发起 API 请求
})

# 组件更新后执行
onUpdated(() => {
  console.log('组件已更新')
})

# 组件卸载前执行（类似 Vue2 的 beforeUnmount）
onBeforeUnmount(() => {
  console.log('组件即将卸载')
  # 清理定时器、事件监听等
})

# 组件创建前执行
onBeforeMount(() => {
  console.log('组件即将挂载')
})
```

**生命周期顺序**：

```
onBeforeMount → onMounted → onBeforeUpdate → onUpdated → onBeforeUnmount → onUnmounted
```

**与传统 options 的对比**：

| Options API | Composition API |
|-------------|-----------------|
| `mounted` | `onMounted()` |
| `updated` | `onUpdated()` |
| `beforeUnmount` | `onBeforeUnmount()` |
| `created` | `onBeforeMount()` |

## 6. 组合式函数

组合式函数（Composition Functions）是封装可复用逻辑的函数，以 `use` 开头命名：

### 6.1 路由组合式函数

从 `vue-router` 导入：

```javascript
import { useRouter, useRoute, useLink } from 'vue-router'

# 获取路由实例（用于编程式导航）
const router = useRouter()
router.push('/dashboard')   # 跳转页面
router.replace('/login')    # 替换当前记录

# 获取当前路由信息（只读）
const route = useRoute()
console.log(route.path)     # 当前路径
console.log(route.name)     # 路由名称
console.log(route.query)    # 查询参数
console.log(route.params)   # 动态参数

# 创建路由链接对象
const link = useLink({ to: '/dashboard' })
# 返回 { href, route, navigate }，用于 <router-link> 替代方案
```

### 6.2 Store 组合式函数

从 Pinia Store 导入：

```javascript
import { useUserStore } from '@/stores/user'
import { useEmployeeStore } from '@/stores/employee'

# 获取 Store 实例
const userStore = useUserStore()
const employeeStore = useEmployeeStore()

# 访问 state
userStore.token
userStore.userInfo?.name

# 调用 actions
await userStore.login('admin', '123456')
employeeStore.list({ page: 1 })

# 访问 getters
userStore.departmentCount
```

**注意**：Store 组合式函数**只能**在 `<script setup>` 或 `setup()` 中使用，不能在普通函数中使用。

## 7. 事件处理

事件处理函数在 `<script setup>` 中直接定义：

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

# 定义方法
function handleClick() {
  count.value++
}

function handleChange(e) {
  console.log(e.target.value)
}
</script>

<template>
  # 方法绑定
  <button @click="handleClick">增加</button>
  
  # 事件修饰符
  <button @click.stop="handleClick">阻止冒泡</button>
  <input @keyup.enter="handleEnter">
  
  # 箭头函数（适合简单逻辑）
  <button @click="count++">+1</button>
</template>
```

### 7.1 常见修饰符

| 修饰符 | 作用 | 示例 |
|--------|------|------|
| `.stop` | 阻止冒泡 | `@click.stop` |
| `.prevent` | 阻止默认行为 | `@submit.prevent` |
| `.once` | 只触发一次 | `@click.once` |
| `.self` | 只处理自身事件 | `@click.self` |
| `.capture` | 捕获阶段触发 | `@click.capture` |
| `.native` | 监听组件原生事件（Vue 3 移除）| - |

## 8. 完整示例

以下是 Tlias 管理系统登录页面的完整实现：

```vue
<!-- src/views/login/Login.vue -->
<script setup>
import { reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessage } from 'element-plus'

# Store 实例
const userStore = useUserStore()

# 路由实例
const router = useRouter()

# 响应式表单（使用 reactive）
const form = reactive({
  username: '',
  password: '123456'   # 默认密码
})

# 加载状态
const loading = ref(false)

# 登录方法
async function handleLogin() {
  # 验证
  if (!form.username) {
    ElMessage.warning('请输入用户名')
    return
  }

  # 显示加载状态
  loading.value = true

  try {
    # 调用 Store 的 login 方法
    await userStore.login(form.username, form.password)
    
    # 成功提示
    ElMessage.success('登录成功')
    
    # 跳转到首页
    router.push('/dashboard')
  } catch (error) {
    # 错误提示
    ElMessage.error('登录失败，请检查用户名和密码')
  } finally {
    # 隐藏加载状态
    loading.value = false
  }
}
</script>

<template>
  <el-container class="login-container">
    <el-row type="flex" justify="center" align-middle style="height: 100vh; width: 100%;">
      <el-col :xs="24" :sm="20" :md="16" :lg="12" :xl="10">
        <el-card class="login-card">
          <template #header>
            <h3 style="margin: 0; text-align: center;">Tlias 管理系统</h3>
          </template>
          
          <el-form :model="form" label-width="80px" label-position="left">
            <el-form-item label="用户名">
              <el-input 
                v-model="form.username" 
                placeholder="请输入用户名" 
                @keyup.enter="handleLogin"
              />
            </el-form-item>
            
            <el-form-item label="密码">
              <el-input 
                v-model="form.password" 
                type="password" 
                placeholder="请输入密码" 
                @keyup.enter="handleLogin"
              />
            </el-form-item>
            
            <el-form-item>
              <el-button 
                type="primary" 
                @click="handleLogin" 
                :loading="loading"
                style="width: 100%"
              >
                登录
              </el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
  </el-container>
</template>

<style scoped>
.login-container {
  height: 100vh;
  background: url('background-image.jpg') no-repeat center center;
  background-size: cover;
}

.login-card {
  width: 400px;
  margin: 0 auto;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>
```

## 9. 注意事项

### 9.1 ref 的使用场景

| 场景 | 推荐方式 | 原因 |
|------|----------|------|
| 基本类型（string、number、boolean） | `ref()` | ref 可以包装任何类型 |
| 对象 | `reactive()` 或 `ref()` | reactive 更简洁，ref 更灵活 |
| 需要解构 | `ref()` | ref 解构后仍保持响应性 |
| 函数参数/返回值 | `ref()` | 更容易传递 |

### 9.2 reactive 的解构陷阱

```javascript
const state = reactive({ name: '张三', age: 25 })

# 错误：解构后失去响应性
const { name, age } = state  
name = '李四'   # 不会触发更新！

# 正确：使用 toRefs
import { toRefs } from 'vue'
const { name, age } = toRefs(state)   # 保持响应性
name.value = '李四'   # 会触发更新（需 .value）
```

### 9.3 Store 的使用限制

- Store 组合式函数只能在 `<script setup>` 或 `setup()` 中调用
- 不能在普通函数中调用（如事件处理函数内部）
- 解决方案：在 `<script setup>` 中获取 Store 实例，在方法中使用

```javascript
# 正确写法
<script setup>
const store = useUserStore()
function method() {
  store.login(...)   # 可以使用 store
}
</script>

# 错误写法（在普通函数中调用）
function outsideSetup() {
  useUserStore()   # 错误！只能在 setup 中使用
}
```

### 9.4 默认值设置

```javascript
# ref 设置默认值
const count = ref(0)
const name = ref('张三')

# reactive 设置默认值
const state = reactive({
  age: 25,
  city: '北京'
})
```

### 9.5 类型推断（配合 TypeScript）

使用 `<script setup lang="ts">` 可以获得更好的类型推断：

```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref<number>(0)   # 显式类型
const name = ref<string>('张三')

function add(amount: number): number {
  return count.value + amount
}
</script>
```

---

## 小结

`<script setup>` 是 Vue3 推荐的单文件组件写法，核心特性：

1. **自动暴露**：顶层变量和方法自动在模板中可用，无需 return
2. **响应式**：`ref()` 和 `reactive()` 创建响应式数据
3. **生命周期**：`onMounted`、`onUpdated` 等组合式生命周期钩子
4. **组合式函数**：`useRouter`、`useRoute`、`useStore()` 等可复用逻辑
5. **导入组件**：直接导入的组件在模板中自动可用
6. **语法简洁**：无需 `export default`，代码更紧凑

理解并熟练使用 `<script setup>`，是掌握 Vue3 开发的关键一步。

关联知识点：
- [[响应式对象]] ref/reactive 原理
- [[Vue Router路由配置]] 路由组合式函数
- [[Pinia状态管理]] Store 组合式函数
- [[Element Plus UI库]] UI 组件使用