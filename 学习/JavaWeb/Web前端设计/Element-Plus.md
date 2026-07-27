---
type: 学习笔记
title: Element Plus 使用指南
created: 2026-07-26
updated: 2026-07-26
tags:
  - Vue
  - UI组件
  - 前端
subject: JavaWeb
---

> Element Plus 是 Vue 3 生态最主流的 UI 组件库。它把按钮、表格、表单、弹窗这些常见的界面元素做成了"预制件"，你只需要像搭积木一样拼装，不用从零写 CSS。

## 目录

- [1. 什么是 Element Plus](#1-什么是-element-plus)
- [2. 如何安装和引入](#2-如何安装和引入)
- [3. 核心组件介绍](#3-核心组件介绍)
  - [3.1 按钮 Button](#31-按钮-button)
  - [3.2 表格 Table](#32-表格-table)
  - [3.3 表单 Form](#33-表单-form)
  - [3.4 对话框 Dialog](#34-对话框-dialog)
  - [3.5 分页 Pagination](#35-分页-pagination)
  - [3.6 消息提示](#36-消息提示)
- [4. 完整示例：用户管理页面](#4-完整示例用户管理页面)
- [小结](#小结)

---

## 1. 什么是 Element Plus

Element Plus 是饿了么前端团队做的一套 **Vue 3 UI 组件库**。通俗理解：它把网页里常用的界面元素（按钮、输入框、表格、弹窗）提前封装好了，你用的时候只需要写一句 `<el-button>保存</el-button>`，就得到一个样式完整、交互到位的按钮。

**如果你只学过 HTML + CSS + JavaScript：**
- 以前做一个表格：手写 `<table>` + `<tr>` + `<td>`，还要自己写分页按钮、搜索框、模态窗的 CSS
- 现在用 Element Plus：写 `<el-table :data="list">` 就够了，样式和交互全帮你做了

| 特性 | 说明 |
|------|------|
| **Vue 3 专属** | 基于 Vue 3 的 `<script setup>` 语法，需要先学 Vue 基础 |
| **70+ 组件** | 按钮、表格、表单、弹窗、日期选择器、导航菜单……覆盖后台管理所有场景 |
| **中文友好** | 饿了么团队出品，文档有中文版，默认语言就是中文 |
| **主题定制** | 在线改主题色，一键下载 CSS |

### Element UI vs Element Plus

| | Element UI | Element Plus |
|--|-----------|-------------|
| 适用版本 | Vue 2 | Vue 3（现在的主流） |
| 状态 | 已停更 | 活跃维护 |
| 新项目选哪一个 | 不选，别用 | **必选** |

---

## 2. 如何安装和引入

### 方式一：npm 安装（正式项目用）

在你已经用 Vite 创建好的 Vue 3 项目里：

```bash
npm install element-plus
```

然后在入口文件 `src/main.js` 里引入：

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'   // ← 这行必须加，否则组件没有样式

const app = createApp(App)
app.use(ElementPlus)                    // 全局注册，之后所有组件都能直接用
app.mount('#app')
```

搞定之后，在任意 `.vue` 文件里直接写 `<el-button>` 就能用了，不用每次 import 组件。

你可以直接在 `<template>` 里写 `<el-button>测试</el-button>` 试试，按钮应该已经出现。

### 方式二：CDN 引入（快速体验用）

如果你不想配 npm，新建一个 `.html` 文件复制下面代码就能跑：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Element Plus 体验</title>
  <!-- 1. 引入 Element Plus 的 CSS -->
  <link rel="stylesheet"
        href="https://unpkg.com/element-plus/dist/index.css" />
</head>
<body>
  <div id="app">
    <el-button type="primary">点我试试</el-button>
  </div>

  <!-- 2. 引入 Vue 3 -->
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
  <!-- 3. 引入 Element Plus -->
  <script src="https://unpkg.com/element-plus"></script>

  <script>
    const { createApp } = Vue

    const app = createApp({
      data() {
        return { }
      }
    })
    // 4. 注册 Element Plus
    app.use(ElementPlus)
    app.mount('#app')
  </script>
</body>
</html>
```

用浏览器打开这个文件，点击按钮试一下，你应该能看到弹出一个蓝色的按钮。

---

## 3. 核心组件介绍

Element Plus 有 70+ 个组件，但你不需要全学。下面这 6 个覆盖了 80% 的中后台页面需求。

### 3.1 按钮 Button

按钮虽然简单，但 Element Plus 给它准备了丰富的样式，覆盖了日常所有场景。

```vue
<template>
  <!-- 颜色类型 -->
  <el-button>默认按钮</el-button>
  <el-button type="primary">主要按钮</el-button>
  <el-button type="success">成功按钮</el-button>
  <el-button type="warning">警告按钮</el-button>
  <el-button type="danger">危险按钮</el-button>

  <!-- 大小 -->
  <el-button size="large">大</el-button>
  <el-button>中（默认）</el-button>
  <el-button size="small">小</el-button>
</template>
```

几个常用属性：
- `type`：颜色类型（primary, success, warning, danger, info）
- `size`：大小（large, default, small）
- `disabled`：禁用按钮（`<el-button disabled>`）
- `circle`：圆形按钮
- `@click`：点击事件，用法和原生 JS 一样

### 3.2 表格 Table

这是后台管理系统里出现频率最高的组件，你可以把数组数据直接绑到表格上。

```vue
<template>
  <el-table :data="users" border stripe style="width: 100%">
    <el-table-column prop="id"   label="编号" width="80" />
    <el-table-column prop="name" label="姓名" width="120" />
    <el-table-column prop="age"  label="年龄" width="80" />
    <el-table-column prop="city" label="城市" />
  </el-table>
</template>

<script setup>
import { ref } from 'vue'

const users = ref([
  { id: 1, name: '张三', age: 25, city: '北京' },
  { id: 2, name: '李四', age: 30, city: '上海' },
  { id: 3, name: '王五', age: 28, city: '广州' },
])
</script>
```

效果：上面这 15 行代码会渲染出一个带边框、斑马纹的表格。一行数据 = 表格里的一行。

**常用属性：**

| 属性 | 作用 |
|------|------|
| `data` | 绑定的数组，数组里每个对象对应表格一行 |
| `border` | 加边框 |
| `stripe` | 隔行变背景色（斑马纹） |
| `height` | 固定表格高度，超出显示纵向滚动条 |

**表格里嵌按钮（操作列）：**

```vue
<el-table-column label="操作" width="150">
  <template #default="scope">
    <el-button size="small" @click="edit(scope.row)">编辑</el-button>
    <el-button size="small" type="danger" @click="del(scope.row.id)">删除</el-button>
  </template>
</el-table-column>
```

`scope.row` 就是当前行的整条数据对象。`scope.row.name` 拿姓名，`scope.row.id` 拿 ID。

### 3.3 表单 Form

```vue
<template>
  <el-form :model="form" label-width="80px">
    <el-form-item label="姓名">
      <el-input v-model="form.name" placeholder="请输入姓名" />
    </el-form-item>
    <el-form-item label="年龄">
      <el-input v-model="form.age" type="number" />
    </el-form-item>
    <el-form-item label="城市">
      <el-select v-model="form.city" placeholder="请选择">
        <el-option label="北京" value="beijing" />
        <el-option label="上海" value="shanghai" />
        <el-option label="广州" value="guangzhou" />
      </el-select>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submit">提交</el-button>
      <el-button @click="reset">重置</el-button>
    </el-form-item>
  </el-form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({
  name: '',
  age: 0,
  city: ''
})

function submit() {
  console.log('提交的数据：', form.value)
}
function reset() {
  form.value = { name: '', age: 0, city: '' }
}
</script>
```

**核心概念：**

- `model`：表单绑定的数据对象，表单里所有输入框的值都和这个对象的属性**双向绑定**（改输入框 → 对象值自动变，改对象值 → 输入框内容自动变）
- `el-form-item`：表单项容器，`label` 是左侧的标签文字
- `v-model`：Vue 的双向绑定指令，用在 `<el-input>`、`<el-select>` 上
- `el-select` + `el-option`：下拉选择框，相当于 HTML 的 `<select>` + `<option>`，但样式好看多了

### 3.4 对话框 Dialog

对话框就是"弹窗"——点击按钮弹出一个浮层，用户可以确认或取消。

```vue
<template>
  <el-button type="primary" @click="show = true">打开对话框</el-button>

  <el-dialog v-model="show" title="确认操作" width="400px">
    <p>确定要删除这条记录吗？</p>
    <template #footer>
      <el-button @click="show = false">取消</el-button>
      <el-button type="primary" @click="confirm">确定</el-button>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref } from 'vue'

const show = ref(false)

function confirm() {
  console.log('用户点了确定')
  show.value = false   // 关弹窗
}
</script>
```

**关键点：**
- `v-model="show"` 控制弹窗显示/隐藏，`show = true` 弹出，`show = false` 关闭
- `title` 是弹窗标题
- `<template #footer>` 是底部按钮区域

### 3.5 分页 Pagination

数据多了不可能一页全显示，分页组件就是这个作用。

```vue
<template>
  <el-pagination
    v-model:current-page="currentPage"
    :page-size="pageSize"
    :total="total"
    layout="prev, pager, next"
    @current-change="loadPage"
  />
</template>

<script setup>
import { ref } from 'vue'

const currentPage = ref(1)   // 当前在第几页
const pageSize = 10          // 每页 10 条
const total = 86             // 总共 86 条数据

function loadPage(page) {
  console.log('加载第', page, '页的数据')
  // 这里发请求获取对应页的数据
}
</script>
```

**参数说明：**

| 参数 | 说明 |
|------|------|
| `v-model:current-page` | 当前页码（双向绑定，用户翻页后值自动变） |
| `page-size` | 每页多少条 |
| `total` | 总条数（分页组件根据它自动算出总页数） |
| `layout` | 显示哪些按钮（prev=上一页, pager=页码, next=下一页） |
| `@current-change` | 用户翻页时触发的事件 |

### 3.6 消息提示

操作完成之后需要给用户反馈（"保存成功""删除失败"），消息提示就是干这个的。它不需要在模板里写组件，**直接在 JS 里调用函数**就行。

```vue
<script setup>
import { ElMessage, ElMessageBox } from 'element-plus'

// 成功 / 警告 / 错误 / 普通消息
ElMessage.success('保存成功！')
ElMessage.warning('请先填写必填项')
ElMessage.error('网络请求失败')
ElMessage.info('这是一条提示')

// 确认弹窗（带确定/取消按钮）
ElMessageBox.confirm('确定要删除吗？', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  type: 'warning'
}).then(() => {
  ElMessage.success('已删除')
}).catch(() => {
  ElMessage.info('已取消')
})
</script>
```

---

## 4. 完整示例：用户管理页面

下面是一个完整页面，把表格、表单、对话框、分页、消息提示串在一起用。

```vue
<template>
  <div class="page">
    <!-- 顶部操作栏 -->
    <div class="toolbar">
      <el-button type="primary" @click="openAdd">+ 添加用户</el-button>
      <el-input v-model="keyword" placeholder="搜索姓名" style="width: 200px" />
      <el-button @click="loadData">刷新</el-button>
    </div>

    <!-- 表格 -->
    <el-table :data="list" border stripe>
      <el-table-column prop="id"   label="ID"   width="60" />
      <el-table-column prop="name" label="姓名" />
      <el-table-column prop="role" label="角色" />
      <el-table-column prop="status" label="状态">
        <template #default="{ row }">
          <el-tag :type="row.status === '启用' ? 'success' : 'danger'">
            {{ row.status }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="120">
        <template #default="{ row }">
          <el-button link @click="edit(row)">编辑</el-button>
          <el-button link type="danger" @click="remove(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <el-pagination
      v-model:current-page="page"
      :page-size="10"
      :total="total"
      layout="prev, pager, next"
      style="margin-top: 20px; justify-content: center"
      @current-change="loadData"
    />

    <!-- 添加/编辑对话框 -->
    <el-dialog v-model="dialogShow" :title="isEdit ? '编辑用户' : '添加用户'" width="400px">
      <el-form :model="form" label-width="60px">
        <el-form-item label="姓名">
          <el-input v-model="form.name" />
        </el-form-item>
        <el-form-item label="角色">
          <el-select v-model="form.role">
            <el-option label="管理员" value="管理员" />
            <el-option label="普通用户" value="普通用户" />
          </el-select>
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="form.status">
            <el-option label="启用" value="启用" />
            <el-option label="禁用" value="禁用" />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogShow = false">取消</el-button>
        <el-button type="primary" @click="save">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

// ===== 数据 =====
const page = ref(1)
const total = ref(0)
const keyword = ref('')
const list = ref([])

// ===== 对话框 =====
const dialogShow = ref(false)
const isEdit = ref(false)
const editId = ref(null)
const form = ref({ name: '', role: '普通用户', status: '启用' })

// ===== 模拟数据源 =====
const allUsers = [
  { id: 1,  name: '张三', role: '管理员',   status: '启用' },
  { id: 2,  name: '李四', role: '普通用户', status: '启用' },
  { id: 3,  name: '王五', role: '普通用户', status: '禁用' },
  { id: 4,  name: '赵六', role: '管理员',   status: '启用' },
  // ……其余数据省略
]

// ===== 加载数据 =====
function loadData() {
  let data = allUsers
  if (keyword.value) {
    data = data.filter(u => u.name.includes(keyword.value))
  }
  total.value = data.length
  const start = (page.value - 1) * 10
  list.value = data.slice(start, start + 10)
}

// ===== 添加 =====
function openAdd() {
  isEdit.value = false
  form.value = { name: '', role: '普通用户', status: '启用' }
  dialogShow.value = true
}

// ===== 编辑 =====
function edit(row) {
  isEdit.value = true
  editId.value = row.id
  form.value = { ...row }            // 把行数据拷贝到表单
  dialogShow.value = true
}

// ===== 删除 =====
function remove(row) {
  ElMessageBox.confirm(`确定删除 ${row.name} 吗？`, '提示', { type: 'warning' })
    .then(() => {
      const idx = allUsers.findIndex(u => u.id === row.id)
      if (idx > -1) allUsers.splice(idx, 1)
      loadData()
      ElMessage.success('已删除')
    })
    .catch(() => {})
}

// ===== 保存 =====
function save() {
  if (isEdit.value) {
    const user = allUsers.find(u => u.id === editId.value)
    if (user) Object.assign(user, form.value)
    ElMessage.success('修改成功')
  } else {
    const newId = allUsers.length ? Math.max(...allUsers.map(u => u.id)) + 1 : 1
    allUsers.push({ id: newId, ...form.value })
    ElMessage.success('添加成功')
  }
  dialogShow.value = false
  loadData()
}

onMounted(() => loadData())
</script>

<style>
.page { padding: 20px; }
.toolbar { display: flex; gap: 10px; margin-bottom: 20px; }
</style>
```

**这个示例里用到的 Element Plus 组件：**

| 组件 | 标签 | 在本示例里的作用 |
|------|------|-----------------|
| 按钮 | `<el-button>` | 添加用户、刷新、保存、取消、删除 |
| 输入框 | `<el-input>` | 搜索框 |
| 标签 | `<el-tag>` | 显示"启用"/"禁用"状态 |
| 表格 | `<el-table>` | 展示用户列表 |
| 分页 | `<el-pagination>` | 翻页控制 |
| 对话框 | `<el-dialog>` | 添加和编辑用户的弹窗 |
| 表单 | `<el-form>` | 弹窗里的表单 |
| 下拉框 | `<el-select>` + `<el-option>` | 角色、状态下拉 |
| 消息提示 | `ElMessage` / `ElMessageBox` | 操作反馈、删除确认 |

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **是什么** | Vue 3 的 UI 组件库，"预制件"拼接界面，不用手写 CSS |
| **怎么装** | `npm install element-plus`，在 `main.js` 里 `app.use(ElementPlus)` |
| **怎么用** | 所有组件都是 `<el-xxx>` 标签，直接写在 `<template>` 里 |
| **必须引入 CSS** | `import 'element-plus/dist/index.css'`，忘了就没样式 |
| **重点组件** | Button(按钮)、Table(表格)、Form(表单)、Dialog(弹窗)、Pagination(分页)、Select(下拉) |

从零搭建 Vue 项目的过程见 [[Vue工程化]]。
