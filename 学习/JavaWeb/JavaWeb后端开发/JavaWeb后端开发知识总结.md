---
type: 学习笔记
title: Java Web 后端开发知识总结
created: 2026-07-23
updated: 2026-07-25
tags: [JavaWeb, 后端, SpringBoot, SpringMVC, MyBati[[学习/JavaWeb/Web前端设计/MyBatis|MyBatis]]
subject: JavaWeb
---

# Java Web 后端开发知识总结

## 目录

- [整体架构：请求处理流程](#整体架构请求处理流程)
- [JavaWeb 基础](#javaweb-基础)
- [SpringMV[[学习/JavaWeb/Web前端设计/IoC和DI|IO[[学习/JavaWeb/Web前端设计/全局异常处理|全局异常处理]]](#springmvc)
- [Spring Framework](#spring-framework)
- [MyBati[[学习/JavaWeb/Web前端设计/MyBatis|MyBatis]](#mybatis)
- [解决方案](#解决方案)
- [SpringBoo[[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]](#springboot)
- [知识体系全景图](#知识体系全景图)
- [各技术之间的关系总结](#各技术之间的关系总结)
- [学习路径建议](#学习路径建议)
- [小结](#小结)

---

## 整体架构：请求处理流程

> **图1：Web后端开发请求处理流程图**
>
> ![image-20260723144323272](C:\Users\VANSYE\AppData\Roaming\Typora\typora-user-images\image-20260723144323272.png)

从浏览器发起请求到返回响应，完整的请求链路如下：

```
浏览器 → 过滤器(Filter) → 拦截器(Interceptor) → Controller → Service → Dao → MySQL
浏览器 ← 过滤器(Filter) ← 拦截器(Interceptor) ← Controller ← Service ← Dao ← MySQL
```

**各层职责：**

| 层级 | 职责 | 对应技术 |
|------|------|----------|
| [[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤器]] | 请求的预处理和响应的后处理，对所有请求生效 | JavaWeb Filter |
| [[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]] | 请求的细粒度拦截，可针对特定路径 | SpringMVC Interceptor |
| **Controller** | 接收请求、解析参数、调用业务、返回响应 | SpringMVC |
| **Service** | 核心业务逻辑处理 | Spring Framework |
| **Dao** | 数据访问，与数据库交互 | MyBatis |
| **MySQL** | 数据持久化存储 | 关系型数据库 |

**上方的核心技术：**

| 技术 | 说明 |
|------|------|
| [[学习/JavaWeb/Web前端设计/IoC和DI|IOC]] | 控制反转，对象由 Spring 容器管理 |
| **DI** | 依赖注入，容器自动注入依赖对象 |
| [[学习/JavaWeb/Web前端设计/SpringAOP技术|AOP]] | 面向切面编程，分离横切关注点 |
| [[学习/JavaWeb/Web前端设计/事务管理|事务管理]] | 保证数据库操作的原子性 |
| [[学习/JavaWeb/Web前端设计/全局异常处理|全局异常处理]] | 统一处理 Controller 层异常 |

**下方的解决方案：**

| 技术 | 说明 |
|------|------|
| [[学习/JavaWeb/Web前端设计/Cookie 和 Session|Cookie、Session]] | 会话管理，跟踪用户状态 |
| [[学习/JavaWeb/Web前端设计/会话技术|JWT]] | 无状态认证方案 |
| [[学习/JavaWeb/JavaWeb后端开发/文件存储/阿里云OSS|阿里云OS[[学习/JavaWeb/Web前端设计/IoC和DI|IO[[学习/JavaWeb/Web前端设计/全局异常处理|全局异常处理]]]] | 对象存储，存放图片/文件 |
| [[学习/JavaWeb/Web前端设计/MyBatis持久层框架|MyBatis]] | 持久层框架，ORM 映射 |

---

## JavaWeb 基础

### 1. 过滤器（Filter）

**概念：** 过滤器是 JavaWeb 的三大组件之一（Servlet、Filter、Listener），工作在请求到达 Servlet 之前和响应返回客户端之前。

**核心要点：**

- 基于 Servlet 规范，所有请求都会经过过滤器
- 可以修改请求和响应（如设置编码、添加请求头）
- 通过 `FilterChain.doFilter()` 放行请求
- 常用于：统一编码处理、登录校验、敏感词过滤、CORS 跨域处理

**生命周期：**

```
init() → doFilter() → doFilter() → ... → destroy()
```

**执行顺序：** 多个过滤器按注册顺序形成"责任链"，请求时按顺序执行，响应时按逆序执行。

### 2. Cookie 和 Session

**Cookie：**

- 存储在**客户端（浏览器）**的键值对数据
- 每次请求自动携带在请求头中
- 大小限制约 4KB
- 不安全，容易被篡改
- 常用于：记住用户名、购物车、简单状态跟踪

**Session：**

- 存储在**服务器端**的数据
- 通过 Session ID（通常存在 Cookie 中）关联客户端
- 大小无严格限制
- 相对安全
- 常用于：登录状态、用户信息存储

**两者对比：**

| 特性 | Cookie | Session |
|------|--------|---------|
| 存储位置 | 客户端 | 服务器端 |
| 安全性 | 低 | 高 |
| 大小限制 | ~4KB | 无严格限制 |
| 生命周期 | 可设置过期时间 | 默认30分钟无操作失效 |
| 性能 | 不占服务器资源 | 占服务器内存 |

---

## SpringMVC

### 1. 接收请求与响应数据

**Controller 层的核心职责：**

- 接收 HTTP 请求（GET、POST、PUT、DELETE）
- 解析请求参数（`@RequestParam`、`@RequestBody`、`@PathVariable`）
- 调用 Service 层处理业务
- 返回响应数据（JSON、视图等）

**常用注解：**

| 注解 | 作用 |
|------|------|
| `@Controller` | 标记为控制器类 |
| `@RestController` | `@Controller` + `@ResponseBody`，直接返回 JSON |
| `@RequestMapping` | 映射请求路径 |
| `@GetMapping` / `@PostMapping` | 指定请求方式的快捷注解 |
| `@RequestParam` | 获取请求参数 |
| `@RequestBody` | 获取请求体（JSON 转对象） |
| `@PathVariable` | 获取 URL 路径参数 |
| `@RequestHeader` | 获取请求头 |

### 2. 拦截器（Interceptor）

**概念：** SpringMVC 提供的请求拦截机制，比过滤器更细粒度，可以针对特定 URL 路径进行拦截。

**核心方法：**

```java
public class LoginInterceptor implements HandlerInterceptor {
    // 请求到达 Controller 之前执行
    boolean preHandle(request, response, handler) {
        // 返回 true 放行，返回 false 拦截
    }
    
    // Controller 执行后、视图渲染前执行
    void postHandle(request, response, handler, modelAndView) {}
    
    // 请求完成后执行（无论是否异常）
    void afterCompletion(request, response, handler, ex) {}
}
```

**拦截器 vs 过滤器：**

| 特性 | 过滤器(Filter) | 拦截器(Interceptor) |
|------|---------------|-------------------|
| 所属规范 | Servlet 规范 | SpringMVC 框架 |
| 拦截范围 | 所有请求 | 可配置特定路径 |
| 执行时机 | Servlet 前后 | Controller 前后 |
| 依赖 | 不依赖 Spring | 依赖 Spring 容器 |
| 常用场景 | 编码设置、CORS | 登录校验、权限控制 |

### 3. 全局异常处理

**概念：** 统一处理 Controller 层抛出的异常，避免每个方法都写 try-catch。

**核心注解：**

| 注解 | 作用 |
|------|------|
| `@ControllerAdvice` | 标记全局异常处理类 |
| `@ExceptionHandler` | 指定处理的异常类型 |
| `@ResponseStatus` | 设置 HTTP 响应状态码 |

**示例：**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        return Result.error(e.getMessage());
    }
    
    @ExceptionHandler(BusinessException.class)
    public Result handleBusinessException(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }
}
```

---

## Spring Framework

### 1. IOC（控制反转）

**概念：** Inversion of Control，将对象的创建和管理权从程序员手中交给 Spring 容器。

**传统方式 vs IOC：**

```java
// 传统方式：程序员自己 new 对象
UserService service = new UserServiceImpl();

// IOC 方式：Spring 容器管理对象
@Autowired
private UserService service;  // Spring 自动注入
```

**核心要点：**

- 对象由 Spring 容器创建和管理，称为 **Bean**
- 程序员不再关心对象怎么创建，只关心怎么用
- 降低了代码耦合度

**常用注解：**

| 注解 | 作用 |
|------|------|
| `@Component` | 通用组件 |
| `@Service` | 业务层组件 |
| `@Controller` | 控制层组件 |
| `@Repository` | 数据访问层组件 |
| `@Bean` | 在配置类中定义 Bean |

### 2. DI（依赖注入）

**概念：** Dependency Injection，IOC 的具体实现方式。Spring 容器自动将依赖的对象注入到需要的地方。

**注入方式：**

```java
// 1. 字段注入（最常用）
@Autowired
private UserService userService;

// 2. 构造器注入（推荐，便于测试）
private final UserService userService;
@Autowired
public UserController(UserService userService) {
    this.userService = userService;
}

// 3. Setter 注入
@Autowired
public void setUserService(UserService userService) {
    this.userService = userService;
}
```

**IOC 和 DI 的关系：**

- **IOC** 是思想：控制反转，把对象管理权交给容器
- **DI** 是实现：依赖注入，容器自动把依赖的对象注入进来
- 两者是同一概念的不同角度描述

### 3. AOP（面向切面编程）

**概念：** Aspect-Oriented Programming，将横切关注点（如日志、事务、权限）从业务逻辑中分离出来，统一处理。

**核心概念：**

| 术语 | 含义 | 示例 |
|------|------|------|
| **切面（Aspect）** | 横切关注点的模块化 | 日志切面、事务切面 |
| **连接点（JoinPoint）** | 可以被拦截的点 | 方法执行前后 |
| **切入点（Pointcut）** | 实际被拦截的连接点 | `execution(* com.itheima.service.*.*(..))` |
| **通知（Advice）** | 在切入点执行的增强逻辑 | 前置通知、后置通知、环绕通知 |

**通知类型：**

```java
@Aspect
@Component
public class LogAspect {
    
    @Before("execution(* com.itheima.service.*.*(..))")
    public void before(JoinPoint jp) {
        // 方法执行前
    }
    
    @AfterReturning("execution(* com.itheima.service.*.*(..))")
    public void afterReturning(JoinPoint jp, Object result) {
        // 方法正常返回后
    }
    
    @AfterThrowing("execution(* com.itheima.service.*.*(..))")
    public void afterThrowing(JoinPoint jp, Exception ex) {
        // 方法抛出异常后
    }
    
    @Around("execution(* com.itheima.service.*.*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        // 环绕通知，可以控制方法是否执行
        Object result = pjp.proceed();  // 执行目标方法
        return result;
    }
}
```

**AOP 的应用场景：**

- 日志记录（方法调用前后记录日志）
- 事务管理（方法执行前开启事务，执行后提交/回滚）
- 权限校验（方法执行前检查权限）
- 性能监控（计算方法执行时间）

### 4. 事务管理

**概念：** 保证一组数据库操作要么全部成功，要么全部失败（原子性）。

**Spring 事务管理：**

```java
@Service
public class EmpService {
    
    @Transactional  // 开启事务
    public void deleteDept(Long id) {
        // 1. 删除部门下的员工
        empMapper.deleteByDeptId(id);
        // 2. 删除部门
        deptMapper.deleteById(id);
        // 如果第2步失败，第1步也会回滚
    }
}
```

**事务的传播行为：**

| 传播行为 | 含义 |
|----------|------|
| `REQUIRED`（默认） | 有事务就加入，没有就新建 |
| `REQUIRES_NEW` | 总是新建事务，挂起当前事务 |
| `NESTED` | 嵌套事务，外层回滚内层也回滚 |

**事务失效的常见场景：**

- 方法不是 public 的
- 同类内部方法调用（绕过了代理）
- 异常被 catch 了没有抛出
- 数据库引擎不支持事务（如 MyISAM）

---

## MyBatis

### 核心职责

| [[学习/JavaWeb/Web前端设计/MyBatis持久层框架|MyBatis]] | 持久层框架，ORM 映射 |

**核心要点：**

- 将 Java 对象映射到数据库表（ORM 的半自动实现）
- 通过 XML 或注解编写 SQL
- 自动处理参数绑定和结果集映射
- 支持动态 SQL（`<if>`、`<foreach>`、`<where>` 等）

**基本使用：**

```java
// Mapper 接口
@Mapper
public interface EmpMapper {
    @Select("SELECT * FROM emp WHERE id = #{id}")
    Emp findById(Long id);
    
    @Insert("INSERT INTO emp(name, age) VALUES(#{name}, #{age})")
    void insert(Emp emp);
    
    @Update("UPDATE emp SET name=#{name} WHERE id=#{id}")
    void update(Emp emp);
    
    @Delete("DELETE FROM emp WHERE id=#{id}")
    void delete(Long id);
}
```

**动态 SQL 示例：**

```xml
<select id="selectByCondition" resultType="Emp">
    SELECT * FROM emp
    <where>
        <if test="name != null">
            AND name LIKE CONCAT('%', #{name}, '%')
        </if>
        <if test="gender != null">
            AND gender = #{gender}
        </if>
    </where>
    ORDER BY create_time DESC
</select>
```

---

## 解决方案

### 1. JWT（JSON Web Token）

**概念：** 一种无状态的认证方案，用于替代传统的 Session 认证。

**工作原理：**

```
1. 用户登录 → 服务器验证账号密码
2. 验证通过 → 服务器生成 JWT Token（包含用户信息 + 签名）
3. 返回 Token → 客户端存储在本地（LocalStorage / Cookie）
4. 后续请求 → 客户端在请求头中携带 Token
5. 服务器验证 → 解析 Token，验证签名，获取用户信息
```

**JWT 的结构：**

```
Header.Payload.Signature
  ↓       ↓        ↓
算法    用户数据    签名（防篡改）
```

**JWT vs Session：**

| 特性 | Session | JWT |
|------|---------|-----|
| 存储位置 | 服务器 | 客户端 |
| 状态 | 有状态 | 无状态 |
| 扩展性 | 差（需要共享 Session） | 好（分布式友好） |
| 安全性 | 高（数据在服务器） | 中（数据在客户端，但签名防篡改） |
| 适用场景 | 单体应用 | 分布式/微服务/前后端分离 |

### 2. 阿里云 OSS

**概念：** Object Storage Service，对象存储服务，用于存储图片、视频、文件等非结构化数据。

**在 Web 开发中的应用：**

- 用户上传的文件（头像、商品图片等）存储到 OSS
- 不存储在应用服务器本地，避免服务器磁盘压力
- 通过 CDN 加速访问，提升用户体验

**基本流程：**

```
用户上传图片 → 后端接收 → 调用阿里云 OSS SDK → 上传到 OSS → 返回文件 URL → 前端展示
```

---

## SpringBoot

### 核心作用

SpringBoot 是整个技术栈的**底层基础**，负责整合所有框架。

> **图2：技术栈整合架构图**
>
> ![image-20260723144355326](C:\Users\VANSYE\AppData\Roaming\Typora\typora-user-images\image-20260723144355326.png)

**SpringBoot 的核心能力：**

| 能力 | 说明 |
|------|------|
| **自动配置** | 根据引入的依赖自动配置 SpringMVC、MyBatis 等 |
| **起步依赖** | 一个依赖引入一组相关依赖（如 `spring-boot-starter-web`） |
| **内嵌服务器** | 内置 Tomcat，不需要外部部署 |
| **配置文件** | `application.yml` / `application.properties` 统一管理配置 |
| **Actuator** | 提供健康检查、指标监控等运维功能 |

**技术栈整合关系：**

```
SpringBoot（底层基础）
    ├── SpringMVC（Web 层：Controller、拦截器、异常处理）
    ├── Spring Framework（核心：IOC/DI、AOP、事务）
    └── MyBatis（数据层：Dao、SQL 映射）
```

**左侧的补充技术：**

| 分类 | 技术 | 说明 |
|------|------|------|
| **JavaWeb** | 过滤器 | 请求的通用预处理 |
| [[学习/JavaWeb/Web前端设计/Cookie 和 Session|Cookie、Session]] | 会话管理，跟踪用户状态 |
| [[学习/JavaWeb/Web前端设计/会话技术|JWT]] | 无状态认证方案 |
| **解决方案** | 阿里云OSS | 文件存储 |

---

## 知识体系全景图

```
┌─────────────────────────────────────────────────────────┐
│                      SpringBoot                          │
│              （自动配置、起步依赖、内嵌服务器）              │
├──────────────┬──────────────────┬───────────────────────┤
│  SpringMVC   │ Spring Framework │       MyBatis          │
│              │                  │                       │
│  接收请求     │  IOC / DI        │  Mapper 接口           │
│  响应数据     │  AOP             │  XML/注解 SQL          │
│  拦截器       │  事务管理         │  动态 SQL              │
│  全局异常处理  │                  │  结果映射              │
├──────────────┴──────────────────┴───────────────────────┤
│                    JavaWeb 基础                           │
│              过滤器、Cookie、Session                      │
├─────────────────────────────────────────────────────────┤
│                    解决方案                                │
│              JWT（认证）、阿里云 OSS（文件存储）             │
└─────────────────────────────────────────────────────────┘
```

---

## 各技术之间的关系总结

| 技术 | 解决的问题 | 在架构中的位置 |
|------|-----------|---------------|
| [[学习/JavaWeb/JavaWeb后端开发/会话管理与登陆校验/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]]]** | 请求的通用预处理 | 最外层，所有请求必经 |
| [[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]] | 请求的细粒度拦截，可针对特定路径 | SpringMVC Interceptor |
| [[学习/JavaWeb/JavaWeb后端开发/SpringMVC/Controller请求与响应|Controlle[[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]]] | 接收请求、返回响应 | Web 层入口 |
| **Service** | 业务逻辑处理 | 核心业务层 |
| **Dao/Mapper** | 数据库操作 | 数据访问层 |
| [[学习/JavaWeb/JavaWeb后端开发/Spring核心/IoC和DI依赖注入|IOC/D[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]]** | 对象管理和依赖注入 | 贯穿所有层 |
| [[学习/JavaWeb/JavaWeb后端开发/SpringAOP/SpringAOP技术|AO[[学习/JavaWeb/Web前端设计/SpringAOP技术|AOP]]]** | 横切关注点分离 | 增强 Service 层 |
| [[学习/JavaWeb/JavaWeb后端开发/Spring核心/事务管理|事务管[[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]]]** | 数据一致性保证 | Service 层方法上 |
| [[学习/JavaWeb/JavaWeb后端开发/SpringMVC/全局异常处理|全局异常处[[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]]]** | 统一异常响应 | Controller 层之上 |
| [[学习/JavaWeb/Web前端设计/会话技术|JWT]] | 无状态认证方案 |
| [[学习/JavaWeb/JavaWeb后端开发/文件存储/阿里云OSS|阿里云 OS[[学习/JavaWeb/Web前端设计/IoC和DI|IO[[学习/JavaWeb/Web前端设计/全局异常处理|全局异常处理]]]]** | 文件存储 | 替代本地存储 |
| [[学习/JavaWeb/Web前端设计/MyBatis持久层框架|MyBatis]] | 持久层框架，ORM 映射 |
| [[学习/JavaWeb/JavaWeb后端开发/SpringBoot原理/SpringBoot起步依赖和自动配置|SpringBoo[[学习/JavaWeb/Web前端设计/Filter过滤器技术|过滤[[学习/JavaWeb/Web前端设计/Interceptor拦截器技术|拦截器]]]]** | 框架整合 | 底层基础 |

---

## 学习路径建议

```
1. JavaWeb 基础
   ── Servlet、Filter、Cookie/Session
   
2. SpringMVC
   └── Controller、拦截器、异常处理
   
3. Spring Framework
   └── IOC/DI → AOP → 事务管理
   
4. MyBatis
   └── Mapper、SQL、动态 SQL
   
5. SpringBoot
   └── 整合以上所有框架
   
6. 解决方案
   ── JWT 认证、OSS 文件存储
```

**核心思想：** 分层架构 + 控制反转 + 面向切面，通过 SpringBoot 统一整合，实现高内聚低耦合的 Web 后端开发。

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **分层架构** | Filter → Interceptor → Controller → Service → Dao → MySQL |
| **SpringMVC** | 接收请求、响应数据、拦截器、全局异常处理 |
| **Spring Framework** | IOC/DI（对象管理）、AOP（切面编程）、事务管理 |
| [[学习/JavaWeb/Web前端设计/MyBatis持久层框架|MyBatis]] | 持久层框架，ORM 映射 |
| **SpringBoot** | 底层基础，自动配置，整合所有框架 |
| **JavaWeb** | 过滤器、Cookie、Session |
| **解决方案** | JWT（认证）、阿里云 OSS（文件存储） |

### 口诀

**过滤器拦请求，拦截器细粒度；  
Controller 接数据，Service 做逻辑；  
Dao 层查数据库，MyBatis 来帮忙；  
IOC 管对象，AOP 切关注点；  
事务保一致，异常全局抓；  
SpringBoot 做底座，一切框架它整合。**

### 最容易混淆的三个点

- **过滤器 vs 拦截器**：过滤器是 Servlet 规范的，拦截所有请求；拦截器是 SpringMVC 的，可针对特定路径
- **IOC vs DI**：IOC 是思想（控制反转），DI 是实现（依赖注入），本质是同一件事
- **`<repositories>` vs `<distributionManagement>`**：前者配置从哪里下载依赖，后者配置发布到哪里（这是 Maven 的，但常一起学）

---

Java Web 后端开发的本质是：**通过分层架构实现职责分离，通过 Spring 的 IOC/DI 降低耦合，通过 AOP 分离横切关注点，通过 SpringBoot 统一整合所有框架，最终构建出高内聚低耦合的 Web 应用。**

掌握这套技术栈，你就能独立开发完整的 Java Web 后端项目！
