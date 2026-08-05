---
type: 学习笔记
title: JavaWeb 开发全流程
created: 2026-08-05
updated: 2026-08-05
tags: [JavaWeb, 全流程, 前后端分离, SpringBoot, Vue3, 部署]
subject: JavaWeb
---

> JavaWeb 全流程 = 需求分析 → 接口设计 → 前后端并行开发 → 联调测试 → 部署上线。前端 Vue3 + Vite + Element Plus，后端 Spring Boot + MyBatis，数据库 MySQL，部署用 Docker Compose。掌握这个流程，就能独立完成一个完整的中后台管理系统。

## 目录

- [一、项目整体架构](#一项目整体架构)
- [二、开发流程总览](#二开发流程总览)
- [三、后端开发详解](#三后端开发详解)
- [四、前端开发详解](#四前端开发详解)
- [五、前后端联调](#五前后端联调)
- [六、项目部署](#六项目部署)
- [七、常见问题与排查](#七常见问题与排查)
- [小结](#小结)

---

## 一、项目整体架构

### 技术栈全景

```
┌─────────────────────────────────────────────────────────────┐
│                        浏览器（前端）                         │
│  Vue 3 + Vite + Element Plus + Vue Router + Pinia + Axios    │
└─────────────────────────────┬───────────────────────────────┘
                              │ HTTP / JSON
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Nginx（反向代理）                         │
│  静态资源托管 + /api/ 代理到后端                              │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Spring Boot 后端                         │
│  Controller → Service → Dao → MyBatis → MySQL               │
│  Filter（鉴权）→ Interceptor（权限）→ ThreadLocal（用户传递） │
└─────────────────────────────┬───────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              MySQL 8.0            Redis 7
           （持久化存储）          （缓存/会话）
└─────────────────────────────────────────────────────────────┘
```

### 项目目录结构

```
myproject/
├── backend/                          # 后端项目（Spring Boot）
│   ├── pom.xml                       # Maven 依赖管理
│   └── src/main/
│       ├── java/com/myapp/
│       │   ├── controller/           # 控制层（接收请求）
│       │   ├── service/              # 业务层（核心逻辑）
│       │   │   └── impl/             # 业务实现
│       │   ├── mapper/               # 数据访问层（MyBatis）
│       │   ├── entity/               # 实体类
│       │   ├── dto/                  # 数据传输对象
│       │   ├── common/               # 公共类（Result、异常、工具）
│       │   ├── filter/               # Filter 过滤器（JWT 解析）
│       │   ├── interceptor/          # Interceptor 拦截器（权限校验）
│       │   ├── util/                 # 工具类（JWT 生成/校验）
│       │   └── WebApplication.java   # 启动类
│       └── resources/
│           ├── application.yml       # 配置文件
│           └── mapper/               # MyBatis XML 映射文件
│
├── frontend/                         # 前端项目（Vue 3）
│   ├── package.json
│   ├── vite.config.js
│   └── src/
│       ├── api/                      # API 接口封装
│       │   ├── http.js               # Axios 实例 + 拦截器
│       │   ├── auth.js               # 登录/登出接口
│       │   └── *.js                  # 各模块接口
│       ├── router/                   # 路由配置
│       │   └── index.js
│       ├── stores/                   # Pinia 状态管理
│       │   ├── user.js
│       │   └── *.js
│       ├── views/                    # 页面组件
│       │   ├── login/
│       │   ├── layout/
│       │   └── ...
│       ├── App.vue
│       └── main.js
│
└── docker-compose.yml                # 容器编排
```

---

## 二、开发流程总览

### 六个阶段

```
1. 需求分析      →  确定功能模块、数据表结构
2. 接口设计      →  前后端约定 API 文档（路径、方法、参数、返回值）
3. 并行开发      →  前端用 Mock 数据，后端写真实逻辑
4. 接口联调      →  对接真实 API，处理跨域和异常
5. 测试修复      →  功能测试、边界测试、性能测试
6. 部署上线      →  Docker Compose 一键部署
```

### 阶段一：需求分析

**做什么**：确定项目有哪些功能模块，每张表有哪些字段。

**输出**：
- 功能清单（如：员工管理、部门管理、登录认证）
- 数据库表设计（ER 图 + 字段说明）
- 接口清单（每个功能对应哪些 API）

### 阶段二：接口设计（契约先行）

**做什么**：前后端一起定义 API 文档，这是联调的基础。

**RESTful 规范**：

| 操作 | HTTP 方法 | URL 示例 | 说明 |
|------|----------|---------|------|
| 查询所有 | GET | `/emps` | 返回员工列表 |
| 查询单个 | GET | `/emps/1` | 返回 id=1 的员工 |
| 新增 | POST | `/emps` | 请求体传员工数据 |
| 修改 | PUT | `/emps` | 请求体传完整员工数据 |
| 删除 | DELETE | `/emps/1` | 删除 id=1 的员工 |
| 分页查询 | GET | `/emps?page=1&pageSize=10` | 带分页参数 |
| 登录 | POST | `/auth/login` | 请求体传用户名密码 |
| 登出 | GET | `/auth/logout` | 清除 Token |

**统一响应格式**：

```json
// 成功
{ "code": 1, "msg": "success", "data": { ... } }

// 失败
{ "code": 0, "msg": "用户名或密码错误", "data": null }
```

### 阶段三：并行开发

**后端**：写 Entity → Mapper → Service → Controller，实现真实接口。

**前端**：用 Mock 数据模拟接口返回，先完成页面结构和交互逻辑。

### 阶段四：联调

- 前端把 Mock 数据换成真实 API 调用
- 处理跨域（Nginx 反向代理）
- 统一处理 401 未登录、500 服务器错误等异常

### 阶段五：测试

- 功能测试：每个按钮、每个表单是否正常
- 边界测试：空值、超长输入、重复提交
- 权限测试：未登录能否访问受保护页面

### 阶段六：部署

- 后端打包为 jar，前端构建为 dist
- Docker Compose 一键启动 MySQL + Redis + 后端 + 前端

---

## 三、后端开发详解

### 3.1 分层架构

```
浏览器请求
    ↓
Filter（全局预处理：编码设置、CORS）
    ↓
Interceptor（细粒度拦截：登录校验、权限检查）
    ↓
Controller（接收请求、解析参数、调用 Service）
    ↓
Service（业务逻辑处理、事务管理）
    ↓
Mapper/Dao（数据访问、SQL 执行）
    ↓
MySQL（数据持久化）
```

**各层职责**：

| 层级 | 职责 | 典型注解 |
|------|------|---------|
| **Controller** | 接收请求、返回响应 | `@RestController`、`@GetMapping`、`@PostMapping` |
| **Service** | 业务逻辑、事务控制 | `@Service`、`@Transactional` |
| **Mapper** | SQL 映射、数据操作 | `@Mapper`、`@Select`、`@Insert` |
| **Filter** | 全局请求预处理 | `@WebFilter`、实现 `Filter` 接口 |
| **Interceptor** | Controller 级拦截 | 实现 `HandlerInterceptor` 接口 |

### 3.2 Controller 层

```java
@RestController
@RequestMapping("/emps")
public class EmpController {

    @Autowired
    private EmpService empService;

    // 分页查询
    @GetMapping
    public Result page(@RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(defaultValue = "10") Integer pageSize,
                       Emp emp) {
        return Result.success(empService.page(page, pageSize, emp));
    }

    // 新增
    @PostMapping
    public Result save(@RequestBody Emp emp) {
        empService.save(emp);
        return Result.success();
    }

    // 修改
    @PutMapping
    public Result update(@RequestBody Emp emp) {
        empService.update(emp);
        return Result.success();
    }

    // 删除
    @DeleteMapping("/{id}")
    public Result delete(@PathVariable Integer id) {
        empService.delete(id);
        return Result.success();
    }

    // 根据 ID 查询
    @GetMapping("/{id}")
    public Result getById(@PathVariable Integer id) {
        return Result.success(empService.getById(id));
    }
}
```

### 3.3 Service 层

```java
@Service
public class EmpServiceImpl implements EmpService {

    @Autowired
    private EmpMapper empMapper;

    @Override
    public PageResult page(Integer page, Integer pageSize, Emp emp) {
        // 分页查询
        PageHelper.startPage(page, pageSize);
        List<Emp> list = empMapper.selectByCondition(emp);
        Page<Emp> p = (Page<Emp>) list;
        return new PageResult(p.getTotal(), p.getResult());
    }

    @Override
    @Transactional  // 开启事务，保证原子性
    public void save(Emp emp) {
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        empMapper.insert(emp);
    }

    @Override
    @Transactional
    public void update(Emp emp) {
        emp.setUpdateTime(LocalDateTime.now());
        empMapper.update(emp);
    }

    @Override
    @Transactional
    public void delete(Integer id) {
        empMapper.deleteById(id);
    }
}
```

### 3.4 Mapper 层（MyBatis）

```java
@Mapper
public interface EmpMapper {

    List<Emp> selectByCondition(Emp emp);

    Emp selectById(Integer id);

    void insert(Emp emp);

    void update(Emp emp);

    void deleteById(Integer id);
}
```

**XML 映射文件** (`resources/mapper/EmpMapper.xml`)：

```xml
<mapper namespace="com.myapp.mapper.EmpMapper">

    <select id="selectByCondition" resultType="Emp">
        SELECT * FROM emp
        <where>
            <if test="name != null and name != ''">
                AND name LIKE CONCAT('%', #{name}, '%')
            </if>
            <if test="gender != null">
                AND gender = #{gender}
            </if>
            <if test="begin != null and end != null">
                AND create_time BETWEEN #{begin} AND #{end}
            </if>
        </where>
        ORDER BY create_time DESC
    </select>

    <insert id="insert" parameterType="Emp" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO emp (name, gender, phone, job, entrydate, deptId, createTime, updateTime)
        VALUES (#{name}, #{gender}, #{phone}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})
    </insert>

</mapper>
```

### 3.5 认证与授权

**JWT（JSON Web Token）无状态认证**：

```
登录流程：
1. 用户提交用户名密码 → POST /auth/login
2. 后端验证通过 → 生成 JWT（含 userId、username、过期时间）
3. 返回 Token 给前端 → 前端存入 localStorage
4. 后续请求 → 前端在请求头携带 Token
5. Filter 解析 Token → ThreadLocal 存储当前用户信息
6. Interceptor 校验 Token 有效性 → 决定放行或 401
```

**JWT 工具类**：

```java
public class JwtUtils {
    private static final String SECRET = "mySecretKey2026!@#";

    // 生成 Token
    public static String generate(Integer userId, String username) {
        return Jwts.builder()
            .setSubject("USER")
            .claim("userId", userId)
            .claim("username", username)
            .setExpiration(new Date(System.currentTimeMillis() + 7 * 24 * 60 * 60 * 1000))
            .signWith(SignatureAlgorithm.HS256, SECRET)
            .compact();
    }

    // 解析 Token
    public static Claims parseToken(String token) {
        return Jwts.parser()
            .setSigningKey(SECRET)
            .parseClaimsJws(token)
            .getBody();
    }
}
```

**Filter（Token 解析 + 用户存入 ThreadLocal）**：

```java
@WebFilter("/api/*")
public class TokenParseFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        String token = request.getHeader("token");

        if (token != null && !token.isEmpty()) {
            try {
                Claims claims = JwtUtils.parseToken(token);
                // 存入 ThreadLocal，后续 Service 层可直接获取
                UserContext.set(claims.get("userId", Integer.class),
                               claims.get("username", String.class));
            } catch (Exception e) {
                // Token 无效
            }
        }
        chain.doFilter(req, res);
    }
}
```

**Interceptor（登录校验）**：

```java
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        // 白名单：登录、 Logout 接口不校验
        String url = req.getRequestURI();
        if (url.contains("/auth/login") || url.contains("/auth/logout")) {
            return true;
        }
        // 检查 ThreadLocal 中是否有用户信息
        if (UserContext.get() == null) {
            res.sendRedirect("/api/401");
            return false;
        }
        return true;
    }
}
```

**ThreadLocal 用户上下文**：

```java
public class UserContext {
    private static final ThreadLocal<Integer> userId = new ThreadLocal<>();
    private static final ThreadLocal<String> username = new ThreadLocal<>();

    public static void set(Integer id, String name) {
        userId.set(id);
        username.set(name);
    }

    public static Integer get() {
        return userId.get();
    }

    public static void remove() {
        userId.remove();
        username.remove();
    }
}
```

### 3.6 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 业务异常
    @ExceptionHandler(BusinessException.class)
    public Result handleBusinessException(BusinessException e) {
        log.error("业务异常：", e);
        return Result.error(e.getMessage());
    }

    // 系统异常
    @ExceptionHandler(SystemException.class)
    public Result handleSystemException(SystemException e) {
        log.error("系统异常：", e);
        return Result.error(e.getMessage());
    }

    // 其他异常
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        log.error("未知异常：", e);
        return Result.error("系统繁忙，请稍后再试");
    }
}
```

### 3.7 Maven 多模块项目

```
myproject-parent/              ← 父工程（packaging=pom）
├── myproject-pojo/            ← 实体类模块（jar）
├── myproject-common/          ← 公共工具模块（jar）
├── myproject-service/         ← 业务逻辑模块（jar）
└── myproject-web/             ← Web 控制层模块（jar，可执行）
```

**父工程 pom.xml 核心配置**：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<modules>
    <module>myproject-pojo</module>
    <module>myproject-common</module>
    <module>myproject-service</module>
    <module>myproject-web</module>
</modules>

<dependencyManagement>
    <dependencies>
        <!-- 统一版本管理 -->
        <dependency>
            <groupId>com.myproject</groupId>
            <artifactId>myproject-pojo</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## 四、前端开发详解

### 4.1 技术栈

| 层级 | 技术 | 作用 |
|------|------|------|
| 构建工具 | Vite | 快速开发服务器 + Rollup 生产构建 |
| 框架 | Vue 3 | 响应式组件化框架 |
| UI 库 | Element Plus | 企业级 UI 组件 |
| 路由 | Vue Router 4 | 单页面应用路由管理 |
| 状态管理 | Pinia | 全局共享状态 |
| HTTP | Axios | 发起 HTTP 请求 |

### 4.2 项目结构

```
frontend/src/
├── api/                    # API 接口封装
│   ├── http.js             # Axios 实例 + 拦截器
│   ├── auth.js             # 登录/登出接口
│   ├── emp.js              # 员工接口
│   └── dept.js             # 部门接口
├── router/                 # 路由配置
│   └── index.js
├── stores/                 # Pinia 状态管理
│   ├── user.js             # 用户状态（token、userInfo）
│   └── emp.js              # 员工列表状态
├── views/                  # 页面组件
│   ├── login/Login.vue     # 登录页
│   ├── layout/Layout.vue   # 布局组件（侧边栏+顶栏）
│   ├── emp/Emp.vue         # 员工管理页
│   └── dept/Dept.vue       # 部门管理页
├── App.vue                 # 根容器（仅 router-view）
└── main.js                 # 应用入口
```

### 4.3 入口文件

```javascript
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(createPinia())
app.use(router)
app.use(ElementPlus)
app.mount('#app')
```

### 4.4 Axios 封装与拦截器

```javascript
// api/http.js
import axios from 'axios'
import { ElMessage } from 'element-plus'

const api = axios.create({
  baseURL: '/api',      // 所有请求自动加 /api 前缀
  timeout: 10000
})

// 请求拦截器：自动注入 Token
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.token = token
  return config
})

// 响应拦截器：统一处理错误
api.interceptors.response.use(
  response => {
    const { code, msg, data } = response.data
    if (code === 1) return data       // 成功：返回业务数据
    ElMessage.error(msg || '操作失败')
    return Promise.reject(new Error(msg))
  },
  error => {
    if (error.response?.status === 401) {
      ElMessage.warning('请先登录')
      setTimeout(() => location.href = '/login', 1500)
    } else {
      ElMessage.error(error.message || '系统异常')
    }
    return Promise.reject(error)
  }
)

export default api
```

### 4.5 Pinia 状态管理

```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { loginApi, logoutApi } from '@/api/auth'

export const useUserStore = defineStore('user', {
  state: () => ({
    token: localStorage.getItem('token') || null,
    userInfo: JSON.parse(localStorage.getItem('userInfo') || 'null')
  }),
  actions: {
    async login(username, password) {
      const data = await loginApi({ username, password })
      this.token = data.token
      this.userInfo = { id: data.id, name: data.name, username: data.username }
      localStorage.setItem('token', this.token)
      localStorage.setItem('userInfo', JSON.stringify(this.userInfo))
    },
    logout() {
      this.token = null
      this.userInfo = null
      localStorage.removeItem('token')
      localStorage.removeItem('userInfo')
    }
  }
})
```

### 4.6 路由配置

```javascript
// router/index.js
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    { path: '/login', component: () => import('@/views/login/Login.vue') },
    {
      path: '/',
      component: () => import('@/views/layout/Layout.vue'),
      children: [
        { path: '', redirect: '/dashboard' },
        { path: 'dashboard', component: () => import('@/views/Dashboard.vue') },
        { path: 'emp', component: () => import('@/views/emp/Emp.vue') },
        { path: 'dept', component: () => import('@/views/dept/Dept.vue') }
      ]
    }
  ]
})

// 路由守卫：未登录跳转登录页
router.beforeEach((to, from, next) => {
  if (to.path !== '/login' && !localStorage.getItem('token')) {
    next('/login')
  } else {
    next()
  }
})

export default router
```

### 4.7 典型页面组件

```vue
<!-- views/emp/Emp.vue -->
<script setup>
import { ref, onMounted } from 'vue'
import { useUserStore } from '@/stores/user'
import { useEmpStore } from '@/stores/emp'
import { empApi } from '@/api/emp'

const userStore = useUserStore()
const empStore = useEmpStore()
const dialogVisible = ref(false)
const empForm = ref({ name: '', gender: '', phone: '', deptId: '' })

const loadData = async () => {
  empStore.list = await empApi.list(empStore.params)
}

const handleAdd = () => {
  empForm.value = { name: '', gender: '', phone: '', deptId: '' }
  dialogVisible.value = true
}

const handleSubmit = async () => {
  if (empForm.value.id) {
    await empApi.update(empForm.value)
  } else {
    await empApi.add(empForm.value)
  }
  dialogVisible.value = false
  loadData()
}

onMounted(loadData)
</script>

<template>
  <el-card>
    <template #header>
      <el-button type="primary" @click="handleAdd">新增员工</el-button>
    </template>
    <el-table :data="empStore.list" border>
      <el-table-column prop="name" label="姓名" />
      <el-table-column prop="gender" label="性别" />
      <el-table-column prop="phone" label="手机号" />
      <el-table-column label="操作">
        <template #default="{ row }">
          <el-button size="small" @click="handleEdit(row)">编辑</el-button>
          <el-button size="small" type="danger" @click="handleDelete(row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      v-model:current-page="empStore.params.page"
      :total="empStore.total"
      @current-change="loadData"
    />
  </el-card>
</template>
```

---

## 五、前后端联调

### 5.1 联调流程

```
1. 后端启动 → 访问 http://localhost:8080/actuator/health → {"status":"UP"}
2. 前端启动 → 访问 http://localhost:5173 → 看到登录页面
3. 登录测试 → 输入账号密码 → 后端返回 Token → 前端存入 localStorage
4. 接口测试 → 用 Postman 逐一验证每个接口
5. 前端对接 → 把 Mock 数据换成真实 API 调用
6. 跨域处理 → 前端请求 /api/* 走 Nginx 反向代理到后端 8080
```

### 5.2 跨域解决

**开发环境**：Vite 代理配置

```javascript
// vite.config.js
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  }
})
```

**生产环境**：Nginx 反向代理

```nginx
location /api/ {
    proxy_pass http://backend:8080/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### 5.3 联调问题排查

| 问题 | 排查方向 |
|------|---------|
| 401 Unauthorized | 检查 Token 是否传入、是否过期 |
| 404 Not Found | 检查 URL 路径是否正确、SpringMVC 路由配置 |
| 500 Internal Error | 查看后端日志 `journalctl -u myapp -f` |
| 跨域报错 | 检查 Nginx 代理配置、后端 CORS 配置 |
| 数据不更新 | 检查 Pinia Store 是否响应式、Vue 响应式绑定 |

---

## 六、项目部署

### 6.1 部署前准备

```bash
# 后端打包
cd backend && mvn clean package -DskipTests
# 产物：target/myproject.jar

# 前端打包
cd frontend && npm run build
# 产物：dist/ 目录
```

### 6.2 docker-compose.yml

```yaml
version: "3.8"

services:
  mysql:
    image: mysql:8.0
    container_name: myproject-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Root@123456
      MYSQL_DATABASE: myproject
    volumes:
      - mysql-data:/var/lib/mysql
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - app-net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: myproject-redis
    restart: always
    volumes:
      - redis-data:/data
    networks:
      - app-net

  backend:
    build: ./backend
    container_name: myproject-backend
    restart: always
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_HOST: mysql
      REDIS_HOST: redis
    volumes:
      - app-logs:/var/log/myproject
    networks:
      - app-net
    depends_on:
      mysql:
        condition: service_healthy

  frontend:
    image: nginx:1.24-alpine
    container_name: myproject-frontend
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./frontend/dist:/usr/share/nginx/html:ro
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - app-net
    depends_on:
      - backend

networks:
  app-net:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
  app-logs:
```

### 6.3 一键部署

```bash
# 构建并启动所有服务
docker compose up -d --build

# 查看状态
docker compose ps

# 查看日志
docker compose logs -f

# 停止所有服务
docker compose down

# 销毁容器和数据卷（⚠️ 数据会丢失）
docker compose down -v
```

### 6.4 Nginx 配置

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    # 前端静态资源
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 反向代理
    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|ico)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 七、常见问题与排查

### 7.1 后端问题

| 现象 | 原因 | 解决 |
|------|------|------|
| 启动报错 `Port 8080 already in use` | 端口被占用 | `sudo lsof -i :8080` 查进程并 kill |
| 数据库连接失败 | 账号密码错误 / MySQL 未启动 | 检查 `application.yml` 和 `docker compose ps` |
| 404 接口找不到 | 路径写错 / 注解写错 | 检查 `@RequestMapping` 路径，用 Postman 测试 |
| 中文乱码 | 字符集未配置 | `application.yml` 加 `spring:http:encoding:charset: UTF-8` |
| 事务不生效 | 同类方法调用 / 异常被 catch | 检查是否跨越方法调用、是否抛出异常 |

### 7.2 前端问题

| 现象 | 原因 | 解决 |
|------|------|------|
| 刷新页面 404 | Vue Router History 模式 | 改用 Hash 模式 或 Nginx 配 `try_files` |
| 登录后跳转失效 | 路由守卫逻辑问题 | 检查 `beforeEach` 中 token 判断 |
| 接口 401 | Token 未携带 / 已过期 | 检查请求拦截器是否注入 Token |
| 数据不更新 | Pinia Store 未响应式更新 | 检查 `this.xxx = data` 而非 `state.xxx = data` |
| 静态资源缓存 | 浏览器缓存旧版本 | Nginx 加版本号或清空缓存 |

### 7.3 部署问题

| 现象 | 原因 | 解决 |
|------|------|------|
| 容器启动后立即退出 | 应用报错 / 配置错误 | `docker logs <container>` 查看日志 |
| 前端能访问但 API 404 | Nginx 代理配置错误 | 检查 `proxy_pass` 路径和后端端口 |
| MySQL 数据丢失 | 未挂载数据卷 | 确保 `-v mysql-data:/var/lib/mysql` |
| 容器间无法通信 | 不在同一网络 | 确保所有服务在同一个 `networks` 下 |

---

## 小结

### 核心流程记忆

```
需求 → 接口设计 → 前后端并行开发 → 联调 → 测试 → Docker 部署
```

### 核心架构记忆

```
浏览器 → Nginx（静态+代理） → Spring Boot（Filter→Interceptor→Controller→Service→Mapper→MySQL）
                                                         ↓
                                                    ThreadLocal（用户上下文）
```

### 核心代码记忆

| 层级 | 核心代码 |
|------|---------|
| **Controller** | `@RestController` + `@GetMapping/@PostMapping` + 统一 `Result` 返回 |
| **Service** | `@Service` + `@Transactional` 事务 |
| **Mapper** | `@Mapper` + XML/注解 SQL |
| **认证** | JWT 生成/解析 + Filter 存 ThreadLocal + Interceptor 校验 |
| **前端 API** | Axios 实例 + 请求拦截器（加 Token） + 响应拦截器（统一错误） |
| **前端状态** | Pinia `defineStore` + `state` + `actions` |
| **前端路由** | Vue Router + `beforeEach` 守卫 |

### 一句总结

> **后端写 RESTful 接口，前端调 API 更新状态，Nginx 做反向代理解决跨域，Docker Compose 一键部署。** 这就是 JavaWeb 全流程的核心。
