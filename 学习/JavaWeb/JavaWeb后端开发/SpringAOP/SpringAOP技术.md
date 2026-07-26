# Spring AOP 技术笔记

## 目录

- [什么是 AOP](#什么是-aop)
- [AOP 的核心概念](#aop-的核心概念)
- [通知类型](#通知类型)
- [通知顺序](#通知顺序)
- [JoinPoint 和 ProceedingJoinPoint](#joinpoint-和-proceedingjoinpoint)
- [切入点表达式](#切入点表达式)
- [常用注解](#常用注解)
- [日志切面能做什么](#日志切面能做什么)
- [小结](#小结)

---

## 什么是 AOP

AOP（Aspect Oriented Programming，面向切面编程）是一种补充 OOP 的编程思想。

它把**与业务无关、但又会反复出现的功能**单独抽出来统一处理，比如：

- 日志记录
- 权限校验
- [[事务管理]]（AOP 的典型应用）
- 性能统计

这样做的好处是：

- 业务代码更干净
- 公共逻辑更集中
- 维护成本更低

---

## AOP 的核心概念

---

## 2. Spring AOP 的核心概念

### 2.1 切面（Aspect）

切面就是**公共功能的模块**。  
例如：日志切面、权限切面。

### 2.2 连接点（JoinPoint）

连接点是程序执行过程中**可以被拦截的位置**。  
在 Spring AOP 里，通常指**方法执行**。

### 2.3 切入点（Pointcut）

切入点是**告诉框架要拦截哪些连接点**的规则。

比如：

- 拦截某个包下的所有方法
- 拦截带有某个注解的方法
- 拦截某种命名规则的方法

### 2.4 通知（Advice）

通知就是**在目标方法前后执行的增强逻辑**。

### 2.5 目标对象（Target）

被 AOP 增强的原始对象。

### 2.6 代理对象（Proxy）

Spring 生成的代理对象。  
外部调用目标方法时，实际先经过代理对象，再进入通知逻辑。

### 2.7 织入（Weaving）

把切面逻辑应用到目标对象的过程。

---

## 通知类型

| 通知类型 | 作用时机 | 说明 |
|---|---|---|
| `@Before` | 目标方法执行前 | 适合做参数校验、日志开始记录 |
| `@After` | 目标方法执行后 | 不管是否异常都会执行，类似 finally |
| `@AfterReturning` | 目标方法正常返回后 | 适合记录返回值 |
| `@AfterThrowing` | 目标方法抛异常后 | 适合记录异常信息 |
| `@Around` | 包裹目标方法前后 | 最强，能控制是否执行目标方法，还能拿到执行耗时 |

### 最常用的是 `@Around`

因为它可以：

- 在方法前后都做处理
- 获取返回值
- 统计执行时长
- 决定是否放行

---

## 通知顺序

### 4.1 单个切面内的执行顺序

如果一个方法同时匹配多个通知，执行顺序通常是：

1. `@Around` 前半段
2. `@Before`
3. 目标方法
4. `@AfterReturning` / `@AfterThrowing`
5. `@After`
6. `@Around` 后半段

如果目标方法抛异常：

1. `@Around` 前半段
2. `@Before`
3. 目标方法
4. `@AfterThrowing`
5. `@After`
6. `@Around` 后半段（如果异常未处理，后半段不会正常执行）

### 4.2 多个切面之间的顺序

多个切面同时生效时，顺序由 `@Order` 控制：

- 数字越小，优先级越高
- 优先级高的切面，通常更靠外层

举例：

- `@Order(1)` 先进入
- `@Order(2)` 后进入

---

## JoinPoint 和 ProceedingJoinPoint

### 5.1 JoinPoint

`JoinPoint` 表示当前被拦截的连接点对象。  
常用来获取：

- 方法名
- 目标类
- 参数列表
- 目标对象

常用方法：

- `getArgs()`：获取参数
- `getSignature()`：获取方法签名
- `getTarget()`：获取目标对象

### 5.2 ProceedingJoinPoint

`ProceedingJoinPoint` 是 `JoinPoint` 的子接口，**只在 `@Around` 中使用**。

它比 `JoinPoint` 多一个关键方法：

- `proceed()`：放行目标方法

也就是说：

- `@Before`、`@AfterReturning` 里通常用 `JoinPoint`
- `@Around` 里必须用 `ProceedingJoinPoint`

---

## 切入点表达式

Spring AOP 里常用的是 `execution(...)`。

### 6.1 语法格式

```java
execution(返回值类型 包名.类名.方法名(参数))
```

### 6.2 常见写法

#### 拦截某个包下所有方法

```java
execution(* com.itheima.controller..*(..))
```

含义：

- `*`：返回值任意
- `com.itheima.controller..*`：controller 包及其子包下任意类
- `(..)`：任意参数

#### 拦截某个类的某个方法

```java
execution(* com.itheima.controller.EmpController.add(..))
```

#### 拦截所有以 add 开头的方法

```java
execution(* com.itheima.controller..*.add*(..))
```

#### 拦截 POST / PUT / DELETE 风格的方法

一般可以按方法名约定来写：

```java
execution(* com.itheima.controller..*.add*(..)) ||
execution(* com.itheima.controller..*.update*(..)) ||
execution(* com.itheima.controller..*.delete*(..))
```

### 6.3 常见通配符

- `*`：任意一个
- `..`：任意多个包或任意多个参数
- `+`：子类类型匹配

---

## 常用注解

- `@Aspect`：声明这是一个切面
- `@Pointcut`：声明切入点表达式
- `@Before`：前置通知
- `@After`：后置通知
- `@AfterReturning`：返回通知
- `@AfterThrowing`：异常通知
- `@Around`：环绕通知
- `@Order`：控制切面执行优先级
- `@Component`：把切面交给 Spring 管理

---

## 日志切面能做什么

一个常见的操作日志切面会记录：

- 操作人
- 操作时间
- 类名
- 方法名
- 方法参数
- 返回值
- 执行耗时

一般用 `@Around` 完成，因为它最适合同时拿到：

- 方法入参
- 返回结果
- 执行时间

---

## 小结

### 口诀

**切面是功能，切点是范围，通知是动作，连接点是位置。**

### 最容易混淆的两个点

- **JoinPoint**：能被拦截的方法位置
- **Pointcut**：拦截哪些 JoinPoint 的规则

---

Spring AOP 的本质是：**通过代理对象，在不修改业务代码的前提下，把公共逻辑织入到目标方法执行前后。**
