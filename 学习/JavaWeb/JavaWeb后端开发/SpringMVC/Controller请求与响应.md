# SpringMVC Controller 请求与响应

## 目录

- [什么是 Controller](#什么是-controller)
- [Controller 的核心职责](#controller-的核心职责)
- [常用注解](#常用注解)
- [接收请求参数](#接收请求参数)
- [返回响应数据](#返回响应数据)
- [RESTful 风格](#restful-风格)
- [请求参数校验](#请求参数校验)
- [常见问题与解决方案](#常见问题与解决方案)
- [小结](#小结)

---

## 什么是 Controller

Controller（控制器）是 SpringMVC 中负责**接收 HTTP 请求、调用业务逻辑、返回响应数据**的组件。

它是 Web 应用的入口层，位于请求处理链路的中间位置：

```
浏览器 → 过滤器 → 拦截器 → Controller → Service → Dao → MySQL
```

### 生活中的类比

把 Controller 想象成餐厅的**服务员**：

- 顾客（浏览器）点菜 → 服务员接收请求
- 服务员把菜单交给后厨（Service）
- 后厨做好菜 → 服务员端给顾客（返回响应）
- 服务员自己不炒菜，只负责传话

---

## Controller 的核心职责

| 职责 | 说明 |
|------|------|
| **接收请求** | 接收 GET、POST、PUT、DELETE 等 HTTP 请求 |
| **解析参数** | 从 URL、请求体、请求头中提取参数 |
| **调用业务** | 调用 Service 层处理业务逻辑 |
| **返回响应** | 将结果封装成 JSON 返回给前端 |

**Controller 不应该做的事：**

- 不写业务逻辑（那是 Service 的事）
- 不直接操作数据库（那是 Dao 的事）
- 不做复杂计算（保持 Controller 轻薄）

---

## 常用注解

### 类级别注解

| 注解 | 说明 |
|------|------|
| `@Controller` | 标记为控制器类，配合视图解析器使用 |
| `@RestController` | `@Controller` + `@ResponseBody`，直接返回 JSON |

**两者区别：**

```java
// @Controller：返回视图名（如 JSP 页面）
@Controller
public class PageController {
    @RequestMapping("/index")
    public String index() {
        return "index";  // 返回视图名
    }
}

// @RestController：直接返回数据（JSON）
@RestController
public class EmpController {
    @GetMapping("/emps")
    public Result list() {
        return Result.success(data);  // 返回 JSON 数据
    }
}
```

### 方法级别注解

| 注解 | 说明 |
|------|------|
| `@RequestMapping` | 映射请求路径（通用） |
| `@GetMapping` | 只接收 GET 请求 |
| `@PostMapping` | 只接收 POST 请求 |
| `@PutMapping` | 只接收 PUT 请求 |
| `@DeleteMapping` | 只接收 DELETE 请求 |
| `@PatchMapping` | 只接收 PATCH 请求 |

**`@RequestMapping` vs 快捷注解：**

```java
// 写法一：通用注解
@RequestMapping(value = "/emps", method = RequestMethod.GET)

// 写法二：快捷注解（推荐，更简洁）
@GetMapping("/emps")
```

### 参数级别注解

| 注解 | 说明 | 示例 |
|------|------|------|
| `@RequestParam` | 获取请求参数（URL 中的 `?key=value`） | `?name=admin` |
| `@RequestBody` | 获取请求体（JSON 数据） | `{"name":"admin"}` |
| `@PathVariable` | 获取 URL 路径参数 | `/emps/1` 中的 `1` |
| `@RequestHeader` | 获取请求头 | `Authorization: Bearer xxx` |
| `@CookieValue` | 获取 Cookie 值 | `JSESSIONID=xxx` |
| `@RequestParam(defaultValue)` | 设置默认值 | `?page=1` |

---

## 接收请求参数

### 1. 简单参数（`@RequestParam`）

用于接收 URL 中的查询参数（`?key=value`）。

```java
@GetMapping("/emps")
public Result page(
    @RequestParam(defaultValue = "1") Integer page,
    @RequestParam(defaultValue = "10") Integer pageSize
) {
    // page=1, pageSize=10
    return Result.success(empService.page(page, pageSize));
}
```

**要点：**

- 参数名和请求参数名一致时，可以省略 `@RequestParam`
- 参数名不一致时，用 `@RequestParam("前端参数名")` 指定
- `required = false` 表示参数可选
- `defaultValue` 设置默认值

### 2. 实体类参数（自动封装）

当请求参数较多时，可以用实体类接收，SpringMVC 会自动将参数封装到对象中。

```java
// 前端请求：POST /emps?name=admin&age=25&gender=1
@PostMapping("/emps")
public Result save(Emp emp) {
    // SpringMVC 自动将 name、age、gender 封装到 Emp 对象
    empService.save(emp);
    return Result.success();
}
```

**要点：**

- 请求参数名必须和实体类属性名一致
- 支持嵌套对象封装
- 适用于 GET 和 POST 请求

### 3. JSON 参数（`@RequestBody`）

用于接收请求体中的 JSON 数据（通常是 POST/PUT 请求）。

```java
// 前端请求：
// POST /emps
// Content-Type: application/json
// Body: {"name":"admin", "age":25, "gender":1}

@PostMapping("/emps")
public Result save(@RequestBody Emp emp) {
    // JSON 自动转换为 Emp 对象
    empService.save(emp);
    return Result.success();
}
```

**要点：**

- 必须配合 `Content-Type: application/json` 使用
- SpringMVC 使用 Jackson 将 JSON 转换为 Java 对象
- 一个方法只能有一个 `@RequestBody`

### 4. 路径参数（`@PathVariable`）

用于接收 URL 路径中的参数，是 RESTful 风格的核心。

```java
// 前端请求：DELETE /emps/1
@DeleteMapping("/emps/{id}")
public Result delete(@PathVariable Integer id) {
    // id = 1
    empService.delete(id);
    return Result.success();
}

// 多个路径参数
@GetMapping("/depts/{deptId}/emps/{empId}")
public Result getEmp(
    @PathVariable Integer deptId,
    @PathVariable Integer empId
) {
    // deptId 和 empId 自动获取
}
```

**要点：**

- URL 中用 `{参数名}` 占位
- 方法参数用 `@PathVariable` 接收
- 参数名必须和占位符名称一致

### 5. 请求头参数（`@RequestHeader`）

```java
@GetMapping("/info")
public Result info(@RequestHeader("Authorization") String token) {
    // token = "Bearer eyJhbG..."
    return Result.success();
}
```

### 参数接收方式对比

| 方式 | 注解 | 数据来源 | 适用场景 |
|------|------|----------|----------|
| 查询参数 | `@RequestParam` | URL `?key=value` | 分页、搜索 |
| 实体类 | 无注解 | URL 参数自动封装 | 表单提交（GET） |
| JSON 体 | `@RequestBody` | 请求体 JSON | 复杂数据提交（POST/PUT） |
| 路径参数 | `@PathVariable` | URL 路径 `/emps/1` | RESTful 资源定位 |
| 请求头 | `@RequestHeader` | HTTP Header | Token 认证 |

---

## 返回响应数据

### 统一响应格式

前后端分离项目中，通常定义统一的响应格式：

```java
public class Result {
    private Integer code;    // 状态码：1 成功，0 失败
    private String msg;      // 提示信息
    private Object data;     // 返回数据

    public static Result success() {
        return new Result(1, "success", null);
    }

    public static Result success(Object data) {
        return new Result(1, "success", data);
    }

    public static Result error(String msg) {
        return new Result(0, msg, null);
    }
}
```

### 返回方式

```java
@RestController
public class EmpController {

    // 1. 返回简单数据
    @GetMapping("/count")
    public Result count() {
        return Result.success(empService.count());
    }

    // 2. 返回对象
    @GetMapping("/emps/{id}")
    public Result getById(@PathVariable Integer id) {
        return Result.success(empService.getById(id));
    }

    // 3. 返回集合
    @GetMapping("/emps")
    public Result list() {
        return Result.success(empService.list());
    }

    // 4. 返回分页数据
    @GetMapping("/emps/page")
    public Result page(
        @RequestParam(defaultValue = "1") Integer page,
        @RequestParam(defaultValue = "10") Integer pageSize
    ) {
        return Result.success(empService.page(page, pageSize));
    }
}
```

### `@ResponseBody` 的作用

```java
// 没有 @ResponseBody：返回视图名
@Controller
public class MyController {
    @RequestMapping("/hello")
    public String hello() {
        return "hello";  // 去找 hello.jsp 或 hello.html
    }
}

// 有 @ResponseBody：直接返回数据
@Controller
public class MyController {
    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return "hello";  // 直接返回字符串 "hello"
    }
}

// @RestController = @Controller + @ResponseBody（所有方法都返回数据）
@RestController
public class MyController {
    @RequestMapping("/hello")
    public String hello() {
        return "hello";  // 直接返回字符串 "hello"
    }
}
```

---

## RESTful 风格

### 什么是 RESTful

RESTful 是一种**API 设计风格**，用 HTTP 方法表示对资源的操作。

### 传统风格 vs RESTful 风格

| 操作 | 传统风格 | RESTful 风格 |
|------|----------|-------------|
| 查询所有 | `GET /getEmps` | `GET /emps` |
| 查询单个 | `GET /getEmpById?id=1` | `GET /emps/1` |
| 新增 | `POST /addEmp` | `POST /emps` |
| 修改 | `POST /updateEmp` | `PUT /emps` |
| 删除 | `GET /deleteEmp?id=1` | `DELETE /emps/1` |

### RESTful 设计原则

| 原则 | 说明 |
|------|------|
| **用名词表示资源** | URL 用名词（`/emps`），不用动词（`/getEmps`） |
| **用 HTTP 方法表示操作** | GET 查、POST 增、PUT 改、DELETE 删 |
| **用路径参数定位资源** | `/emps/1` 表示 id 为 1 的员工 |
| **返回合适的状态码** | 200 成功、404 未找到、500 服务器错误 |

### 示例

```java
@RestController
@RequestMapping("/emps")
public class EmpController {

    @GetMapping                    // 查询所有员工
    public Result list() { ... }

    @GetMapping("/{id}")           // 根据 ID 查询
    public Result getById(@PathVariable Integer id) { ... }

    @PostMapping                   // 新增员工
    public Result save(@RequestBody Emp emp) { ... }

    @PutMapping                    // 修改员工
    public Result update(@RequestBody Emp emp) { ... }

    @DeleteMapping("/{id}")        // 删除员工
    public Result delete(@PathVariable Integer id) { ... }
}
```

---

## 请求参数校验

### 使用 `@Validated` + 注解校验

```java
// 1. 实体类上加校验注解
public class Emp {

    @NotBlank(message = "用户名不能为空")
    private String name;

    @NotNull(message = "年龄不能为空")
    @Min(value = 18, message = "年龄不能小于18")
    @Max(value = 60, message = "年龄不能大于60")
    private Integer age;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
}

// 2. Controller 中开启校验
@PostMapping("/emps")
public Result save(@Validated @RequestBody Emp emp) {
    // 如果校验不通过，会自动抛出 MethodArgumentNotValidException
    empService.save(emp);
    return Result.success();
}
```

### 常用校验注解

| 注解 | 说明 |
|------|------|
| `@NotNull` | 不能为 null |
| `@NotBlank` | 不能为 null 且不能为空字符串 |
| `@NotEmpty` | 不能为 null 且长度 > 0 |
| `@Min` / `@Max` | 数值范围 |
| `@Size` | 字符串/集合长度范围 |
| `@Pattern` | 正则表达式匹配 |
| `@Email` | 邮箱格式 |

---

## 常见问题与解决方案

### 问题 1：`@RequestBody` 接收不到参数

**原因：**

- 前端没有设置 `Content-Type: application/json`
- 前端传的是表单数据而不是 JSON

**解决方案：**

```javascript
// 前端必须设置 Content-Type
fetch('/emps', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: 'admin', age: 25 })
});
```

### 问题 2：`@PathVariable` 参数类型不匹配

**原因：**

URL 中的参数是字符串，但方法参数声明为 Integer，且传入的值不是数字。

**解决方案：**

- 确保 URL 中的参数值类型正确
- 或者用 String 接收后手动转换

### 问题 3：实体类参数接收不到值

**原因：**

- 请求参数名和实体类属性名不一致
- 实体类没有 getter/setter 方法

**解决方案：**

- 检查参数名是否一致
- 使用 Lombok 的 `@Data` 注解自动生成 getter/setter

### 问题 4：返回中文乱码

**原因：**

SpringMVC 默认编码可能不是 UTF-8。

**解决方案：**

```java
// 方式一：在 @RequestMapping 中指定
@RequestMapping(value = "/emps", produces = "application/json;charset=UTF-8")

// 方式二：在 application.yml 中配置
spring:
  http:
    encoding:
      charset: UTF-8
      enabled: true
```

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **`@RestController`** | 标记控制器，所有方法直接返回 JSON |
| **`@GetMapping`/`@PostMapping`** | 映射请求路径和方法 |
| **`@RequestParam`** | 接收 URL 查询参数 |
| **`@RequestBody`** | 接收 JSON 请求体 |
| **`@PathVariable`** | 接收 URL 路径参数 |
| **`@RequestHeader`** | 接收请求头 |
| **RESTful 风格** | 用 HTTP 方法表示操作，URL 用名词表示资源 |
| **统一响应格式** | 用 Result 对象封装 code、msg、data |

### 口诀

**Controller 是入口，接收请求做转发；  
GET 查 POST 增，PUT 改 DELETE 删；  
RequestParam 接查询，RequestBody 接 JSON；  
PathVariable 取路径，RESTful 风格最优雅。**

### 最容易混淆的三个点

- **`@Controller` vs `@RestController`**：前者返回视图，后者返回 JSON（等于前者 + `@ResponseBody`）
- **`@RequestParam` vs `@RequestBody`**：前者接 URL 参数，后者接 JSON 请求体
- **`@PathVariable` vs `@RequestParam`**：前者从 URL 路径取值（`/emps/1`），后者从查询字符串取值（`?id=1`）

---

Controller 是 Web 应用的门面，掌握好请求参数的接收和响应数据的返回，就能写出清晰、规范的接口！
