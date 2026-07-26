# Filter 过滤器技术

## 目录
- [什么是 Filter](#什么是-filter)
- [Filter 的工作原理](#filter-的工作原理)
- [Filter 的生命周期](#filter-的生命周期)
- [Filter 的使用步骤](#filter-的使用步骤)
- [Filter 的配置方式](#filter-的配置方式)
- [FilterChain 过滤器链](#filterchain-过滤器链)
- [常见应用场景](#常见应用场景)
- [代码示例：登录校验 Filter](#代码示例登录校验-filter)
- [Filter 与 Interceptor 的区别](#filter-与-interceptor-的区别)
- [小结](#小结)

---

## 什么是 Filter

> **Filter（过滤器）是 Java Servlet 规范中的核心组件之一，用于在请求到达 Servlet 之前或响应返回浏览器之前，对请求/响应进行预处理或后处理。**
> 
> 详细版见 [[Java Web 后端开发知识总结]]，速查版见 [[Filter过滤器]]。

它是一段可以**插入到请求处理流程中的代码**，位于 Web 服务器和目标资源（Servlet / JSP / 静态资源）之间。

### 核心特点

- 属于 **Java EE（Jakarta EE）规范**，不是 Spring 特有（虽然 Spring 也能整合）
- **拦截请求**：在请求到达目标资源之前执行
- **拦截响应**：在响应返回浏览器之前执行
- **可插拔**：通过配置决定哪些 Filter 生效，不影响业务代码
- **链式执行**：多个 Filter 按顺序组成 FilterChain

### 生活中的类比

`
┌─────────┐   ┌───────────────────┐   ┌───────────────────┐   ┌──────────┐
│  浏览器  │ → │  安检1：身份验证   │ → │  安检2：包裹检查   │ → │  服务器  │
│  请求    │ ← │  (放行/拦截)       │ ← │  (放行/拦截)       │ ← │  处理    │
└─────────┘   └───────────────────┘   └───────────────────┘   └──────────┘
              ↑ 每个 Filter 可以决定是否放行到下一个环节
              或直接拦截，返回错误响应
`

---

## Filter 的工作原理

### 单次请求的完整流程

`
① 浏览器发送 HTTP 请求
        │
        ▼
    到达 Web 服务器（Tomcat）
        │
        ▼
② 经过 Filter 链（FilterChain）
   ┌─────────────────────────────────────────────────────┐
   │  Filter1.doFilter()                                │
   │    ↓                                               │
   │  Filter2.doFilter()                                │
   │    ↓                                               │
   │  Filter3.doFilter()                                │
   │    ↓  chain.doFilter(request, response)  ← 关键！  │
   └─────────────────────────────────────────────────────┘
        │  放行
        ▼
③ 访问目标资源（Servlet / Controller / 静态页面）
   执行业务逻辑，生成响应
        │
        ▼
④ 反向经过 Filter 链（后处理）
   ┌─────────────────────────────────────────────────────┐
   │  Filter3.doFilter() 的后续代码（chain 之后的部分）   │
   │    ↑                                               │
   │  Filter2.doFilter() 的后续代码                      │
   │    ↑                                               │
   │  Filter1.doFilter() 的后续代码                      │
   └─────────────────────────────────────────────────────┘
        │
        ▼
⑤ 响应返回浏览器
`

### 关键方法：doFilter

`java
public void doFilter(ServletRequest request,
                     ServletResponse response,
                     FilterChain chain)
        throws IOException, ServletException {

    // ========== 前处理（请求到达资源之前） ==========
    System.out.println("请求到达 Filter 了！");

    // ===== 关键：放行到下一个 Filter / 目标资源 =====
    chain.doFilter(request, response);  // ← 不调用这行，请求就被拦截了！

    // ========== 后处理（响应返回浏览器之前） ==========
    System.out.println("响应返回 Filter 了！");
}
`

> **要点**：chain.doFilter() 是放行开关。
> - **调用** → 放行，继续后续流程
> - **不调用** → 拦截，直接在此结束请求（返回错误/重定向）

---

## Filter 的生命周期

Filter 有三个核心方法，对应生命周期的三个阶段：

| 方法 | 调用时机 | 执行次数 | 用途 |
|------|----------|----------|------|
| init(FilterConfig) | Filter 被创建时 | **一次** | 初始化资源、读取配置 |
| doFilter(req, res, chain) | 每次匹配请求时 | **每次请求** | 核心过滤逻辑 |
| destroy() | Filter 被销毁时（服务器关闭/项目卸载） | **一次** | 释放资源 |

### 生命周期流程图

`
服务器启动
    │
    ▼
Filter 实例被创建（单例）
    │
    ▼
init(FilterConfig)   ← 仅执行一次，初始化
    │
    ▼
┌───────────────────┐
│ doFilter() 被调用  │ ← 每次匹配的请求都执行，可能 N 次
│                   │
│  1. 前处理        │
│  2. chain.doFilter() ← 放行
│  3. 后处理        │
└───────────────────┘
    │
    ▼
服务器关闭 / 项目卸载
    │
    ▼
destroy()           ← 仅执行一次，释放资源
    │
    ▼
垃圾回收
`

### 生命周期代码示例

`java
package luv.vansye.filter;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter {

    /**
     * 初始化方法 —— Filter 被创建时调用，**只执行一次**
     * 可以用来：加载配置文件、建立数据库连接池、初始化缓存等
     */
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("=== DemoFilter init() 被执行，初始化完成 ===");
        // 可以通过 filterConfig 读取初始化参数
        // String encoding = filterConfig.getInitParameter("encoding");
    }

    /**
     * 核心过滤方法 —— 每次匹配的请求都会执行
     */
    @Override
    public void doFilter(ServletRequest request,
                         ServletResponse response,
                         FilterChain chain)
            throws IOException, ServletException {

        System.out.println("=== doFilter() 前处理：请求被拦截到 ===");

        // 放行（放行后会去执行目标资源，然后再回到这里执行后续代码）
        chain.doFilter(request, response);

        System.out.println("=== doFilter() 后处理：响应返回前 ===");
    }

    /**
     * 销毁方法 —— Filter 被销毁时调用，**只执行一次**
     * 可以用来：关闭流、断开连接、清理缓存等
     */
    @Override
    public void destroy() {
        System.out.println("=== DemoFilter destroy() 被执行，销毁完成 ===");
    }
}
`

> **注意**：Filter 是**单例**的，整个应用只有一个实例。所以 doFilter 中不要使用成员变量保存请求相关数据（线程不安全）。

---

## Filter 的使用步骤

### 以 Jakarta Servlet（Spring Boot 内置 Tomcat）为例

**步骤 1**：创建一个类，实现 jakarta.servlet.Filter 接口

**步骤 2**：重写 init、doFilter、destroy 三个方法（doFilter 是核心）

**步骤 3**：在 doFilter 中编写过滤逻辑，调用 chain.doFilter() 放行

**步骤 4**：配置 Filter 的拦截路径

---

## Filter 的配置方式

### 方式一：注解方式 @WebFilter（推荐，简单）

`java
package luv.vansye.filter;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import java.io.IOException;

// 拦截所有请求
@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter { ... }

// 只拦截 /emps 开头的请求
@WebFilter(urlPatterns = "/emps/*")
public class LoginCheckFilter implements Filter { ... }

// 只拦截 .jsp 页面
@WebFilter(urlPatterns = "*.jsp")
public class JspFilter implements Filter { ... }
`

> 使用 @WebFilter 时，需要在**启动类上加上 @ServletComponentScan**，让 Spring Boot 扫描到 @WebFilter 注解：
>
> `java
> @SpringBootApplication
> @ServletComponentScan(basePackages = "luv.vansye.filter")
> public class WebManagementApplication {
>     public static void main(String[] args) {
>         SpringApplication.run(WebManagementApplication.class, args);
>     }
> }
> `

### 方式二：web.xml 配置（传统方式）

`xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">

    <filter>
        <filter-name>demoFilter</filter-name>
        <filter-class>luv.vansye.filter.DemoFilter</filter-class>
        <!-- 可以配置初始化参数 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>demoFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
`

### 方式三：Spring Boot 的 FilterRegistrationBean（Spring 生态）

`java
package luv.vansye.config;

import luv.vansye.filter.DemoFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<DemoFilter> demoFilterRegistration() {
        FilterRegistrationBean<DemoFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new DemoFilter());
        registration.addUrlPatterns("/*");           // 拦截路径
        registration.setName("demoFilter");           // Filter 名称
        registration.setOrder(1);                     // 执行顺序（数字越小越先执行）
        return registration;
    }
}
`

### 拦截路径匹配规则

| 模式 | 含义 | 匹配示例 |
|------|------|----------|
| /* | 拦截所有请求 | /login、/emps、/index.html 全部匹配 |
| /emps/* | 拦截 /emps 下的所有子路径 | /emps/list、/emps/1，但不匹配 /emps |
| /emps | 只拦截 /emps 这一个路径 | 只匹配 /emps |
| *.jsp | 拦截所有 .jsp 结尾的请求 | /index.jsp、/pages/a.jsp |
| /login | 精确匹配 | 只匹配 /login |

---

## FilterChain 过滤器链

> 当一个请求匹配**多个 Filter** 时，它们会按**顺序**组成一条"链"执行。

### 多个 Filter 的执行顺序

`
浏览器
   │
   ▼
┌──────────────┐
│ FilterA      │
│ 前处理       │
│  chain.doFilter() ──┐
│  后处理       │     │
└──────────────┘     │
   │                 │
   ▼                 │
┌──────────────┐     │
│ FilterB      │     │
│ 前处理       │     │
│  chain.doFilter() ──┤
│  后处理       │     │
└──────────────┘     │
   │                 │
   ▼                 │
┌──────────────┐     │
│ 目标资源      │     │
│ (Servlet/    │     │
│  Controller) │◄────┘
│ 生成响应      │
└──────────────┘
   │
   ▼
 FilterB 后处理
   │
   ▼
 FilterA 后处理
   │
   ▼
返回浏览器
`

### 执行顺序总结

- **请求时**：FilterA → FilterB → 目标资源
- **响应时**：目标资源 → FilterB → FilterA
- 像一个 **"先进后出" 的栈**

### 执行顺序的确定方式

| 配置方式 | 优先级规则 |
|----------|-----------|
| @WebFilter | 按 **类名的字母顺序** 执行（A 在 B 前面） |
| web.xml | 按 <filter-mapping> 的**书写顺序**（先写的先执行） |
| FilterRegistrationBean | 按 setOrder() 的**数字大小**（越小越先） |

---

## 常见应用场景

### 场景 1：登录验证（最常用）
- 未登录用户访问受保护页面 → 拦截并跳转到登录页
- 已登录用户 → 放行

### 场景 2：字符编码统一处理
- 所有请求/响应都设置为 UTF-8，避免每个 Servlet 都写一遍

### 场景 3：敏感词过滤
- 对提交的参数做敏感词替换或拦截

### 场景 4：请求日志记录
- 记录每次请求的 URL、参数、耗时、IP 等

### 场景 5：跨域处理（CORS）
- 统一添加跨域响应头

### 场景 6：权限控制
- 检查用户是否有访问某资源的权限

---

## 代码示例：登录校验 Filter

### 背景（结合你的项目）

- 你的项目使用 **JWT 令牌**进行登录验证
- 前端登录后会拿到 Token，后续请求在请求头中携带：Authorization: Bearer <token>
- 需要一个 Filter 拦截需要登录的请求，校验 Token 是否合法
- 放行规则：
  - /login → 放行（登录接口不能拦截）
  - 其他路径 → 校验 Token，合法放行，非法返回 401

### 代码实现

`java
package luv.vansye.filter;

import com.alibaba.fastjson.JSONObject;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import luv.vansye.Result;
import luv.vansye.utils.JwtUtils;
import org.springframework.util.StringUtils;

import java.io.IOException;

/**
 * 登录校验 Filter
 * 拦截所有请求，校验 JWT Token
 */
@WebFilter(urlPatterns = "/*")
public class LoginCheckFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req,
                         ServletResponse res,
                         FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;

        // ========== 步骤1：获取请求路径 ==========
        String url = request.getRequestURL().toString();
        System.out.println("Filter 拦截到请求：" + url);

        // ========== 步骤2：判断是否是登录相关请求 ==========
        // 如果是登录请求，直接放行（登录接口不能拦截自己）
        if (url.contains("/login")) {
            System.out.println("登录请求，直接放行");
            chain.doFilter(request, response);
            return; // 放行后直接结束本方法，不执行后续校验逻辑
        }

        // ========== 步骤3：获取请求头中的 Token ==========
        // 前端约定的传递方式：Header 中的 Authorization，值为 "Bearer " + token
        String authHeader = request.getHeader("Authorization");

        // ========== 步骤4：判断 Token 是否存在 ==========
        if (!StringUtils.hasLength(authHeader) || !authHeader.startsWith("Bearer ")) {
            System.out.println("Token 为空或格式不对，返回未登录");
            Result result = Result.error("NOT_LOGIN");
            // 将 Result 对象转 JSON 写入响应体
            String json = JSONObject.toJSONString(result);
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().write(json);
            return; // 拦截，不放行
        }

        // 提取真正的 token 部分（去掉 "Bearer " 前缀）
        String token = authHeader.substring(7);

        // ========== 步骤5：校验 Token ==========
        try {
            // 解析成功说明 Token 合法
            JwtUtils.parseJwt(token);
            System.out.println("Token 校验通过，放行");

            // ========== 步骤6：校验通过，放行 ==========
            chain.doFilter(request, response);

        } catch (Exception e) {
            // 解析失败（签名错误、过期、格式非法等）
            System.out.println("Token 校验失败：" + e.getMessage());
            Result result = Result.error("NOT_LOGIN");
            String json = JSONObject.toJSONString(result);
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().write(json);
            // 不放行，方法结束
        }
    }
}
`

### 同时需要在启动类开启 Filter 扫描

`java
package luv.vansye;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
@ServletComponentScan(basePackages = "luv.vansye.filter")
public class WebManagementApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebManagementApplication.class, args);
    }
}
`

### 登录校验 Filter 的执行流程图

`
请求到达
   │
   ▼
 是否是 /login？ ──是──► 放行 ──► Controller
   │ 否
   ▼
 请求头是否有 Token？ ──否──► 返回 NOT_LOGIN
   │ 是
   ▼
 Token 是否合法？ ──否──► 返回 NOT_LOGIN
   │ 是
   ▼
 放行 ──► Controller
`

### 字符编码 Filter 示例（简单）

`java
package luv.vansye.filter;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 统一字符编码 Filter
 */
@WebFilter(urlPatterns = "/*")
public class EncodingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req,
                         ServletResponse res,
                         FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;

        // 前处理：统一设置编码
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        response.setCharacterEncoding("UTF-8");

        // 放行
        chain.doFilter(request, response);

        // 后处理（一般编码处理不需要后处理，留空即可）
    }
}
`

---

## Filter 与 Interceptor 的区别

> Spring 项目中常听到 **Interceptor（拦截器）**，它和 Filter 功能类似，但属于不同的技术体系。详见 [[Interceptor拦截器技术]]。

| 对比项 | Filter（过滤器） | Interceptor（拦截器） |
|--------|-----------------|---------------------|
| **规范归属** | Jakarta Servlet 规范（Java EE） | Spring 框架特有 |
| **拦截时机** | 请求进入 Servlet 之前（Tomcat 容器级别） | 请求进入 Controller 之前（Spring 容器级别） |
| **能获取的对象** | ServletRequest、ServletResponse（较底层） | HttpServletRequest、HttpServletResponse、HandlerMethod、ModelAndView（能拿到 Spring 上下文） |
| **依赖 Spring 容器** | 不依赖，可以脱离 Spring 使用 | 强依赖 Spring，必须在 Spring 容器中 |
| **执行顺序** | 在 Interceptor **之前** 执行 | 在 Filter **之后**、Controller **之前** 执行 |
| **访问路径匹配** | 基于 URL 模式匹配 | 基于 Spring MVC 路径匹配（支持更灵活的 Ant 风格） |
| **能拦截的资源** | 所有请求（Servlet、JSP、静态资源） | 仅拦截 Spring MVC 管理的请求（Controller 方法） |

### 执行顺序图解

`
┌─────────────────────────────────────────────────────┐
│                     Tomcat 容器                        │
│  ┌────────────────────────────────────────────────┐  │
│  │   Filter1 → Filter2 → ...                     │  │
│  │   （Servlet 规范，容器级别）                    │  │
│  └────────────────────────────────────────────────┘  │
│                           ↓                          │
│  ┌────────────────────────────────────────────────┐  │
│  │   DispatcherServlet                            │  │
│  │   ┌────────────────────────────────────────┐   │  │
│  │   │ Interceptor1 → Interceptor2 → ...      │   │  │
│  │   │ （Spring 特有，框架级别）               │   │  │
│  │   │           ↓                            │   │  │
│  │   │       Controller 执行业务逻辑           │   │  │
│  │   └────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
`

### 选择建议

| 场景 | 推荐使用 | 原因 |
|------|----------|------|
| 登录校验（JWT） | Filter 或 Interceptor 都可 | 逻辑简单，两种都能实现 |
| 字符编码 | **Filter** | 需要在所有请求最前端统一处理，Filter 更早执行 |
| 跨域 CORS | **Filter** | 需要在容器级别设置响应头 |
| Spring 权限控制（如 @PreAuthorize） | **Interceptor / AOP** | 需要 Spring 上下文支持 |
| 记录请求日志 | 都可 | Filter 更底层，Interceptor 能拿到 Controller 信息 |
| 敏感词过滤 | **Filter** | 在所有请求前端统一处理 |

> **本项目选择建议**：使用 **Filter** 做 JWT 登录校验，因为：
> 1. 不依赖 Spring 容器，更通用
> 2. 在请求最前端拦截，避免未登录请求进入 Controller 层
> 3. 实现简单，直接用 @WebFilter 注解

---

## 小结

### Filter 核心要点速记

| 要点 | 说明 |
|------|------|
| **接口** | jakarta.servlet.Filter（注意是 jakarta 不是 javax） |
| **三个方法** | init()（一次，初始化）、doFilter()（每次，核心）、destroy()（一次，销毁） |
| **放行开关** | chain.doFilter(request, response) —— **必须调用才能放行** |
| **配置方式** | @WebFilter（需 @ServletComponentScan）、web.xml、FilterRegistrationBean |
| **拦截路径** | /* 全部、/emps/* 部分、*.jsp 后缀 |
| **执行顺序** | 先进后出（栈结构），类名字母序 / 配置顺序 / order 值 |
| **与 Interceptor** | Filter 是 Servlet 规范，Interceptor 是 Spring 特有，Filter 更早执行 |

### 开发 Filter 的代码模板

`java
@WebFilter(urlPatterns = "/*")
public class XxxFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {

        // 1. 前处理：对请求做检查/预处理
        // ...

        // 2. 判断是否放行
        //    - 放行：chain.doFilter(req, res)
        //    - 拦截：返回错误响应，不调用 chain.doFilter()

        // 3. 后处理：对响应做加工
        // ...
    }
}
`

掌握 Filter 技术，你就能在项目中实现登录校验、编码统一、日志记录、权限控制等各种横切关注点，让业务代码更专注于核心逻辑！