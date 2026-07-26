# ThreadLocal 技术笔记

## 目录

- [什么是 ThreadLocal](#什么是-threadlocal)
- [为什么 Web 项目里常用它](#为什么-web-项目里常用它)
- [核心方法](#核心方法)
- [登录用户传递的典型用法](#登录用户传递的典型用法)
- [注意事项](#注意事项)
- [在本项目里的理解方式](#在本项目里的理解方式)
- [小结](#小结)

---

## 什么是 ThreadLocal

`ThreadLocal` 可以理解为**线程内部的专属变量存储区**。

同一个变量，放到不同线程里，彼此看到的值是隔离的。  
它的核心特点是：**线程存储，线程隔离**。

你可以把它想成：

- 每个线程都有一个自己的小柜子
- 往柜子里放什么，只能这个线程自己取出来
- 其他线程看不到，也不会互相影响

---

## 为什么 Web 项目里常用它

在 Web 项目里，请求通常会经过 [[Filter过滤器技术]] 或 [[Interceptor拦截器技术]]：

1. 过滤器或拦截器
2. 控制器
3. 业务层
4. 数据层

如果每一层都手动传递“当前登录用户”，方法参数会越来越长，也容易漏传。

`ThreadLocal` 的作用就是：  
**在请求开始时把当前用户放进去，后面的代码随取随用。**

这样做的好处是：

- 不需要层层传参
- 当前线程内任何地方都能拿到登录用户
- 代码更干净

---

## 核心方法

| 方法 | 作用 | 说明 |
|------|------|------|
| `set(T value)` | 存值 | 把值放进当前线程的 ThreadLocal 中 |
| `get()` | 取值 | 从当前线程中取出值 |
| `remove()` | 删值 | 删除当前线程中的值，请求结束后一定要清理 |

### `set(T value)`

```java
threadLocal.set(user);
```

### `get()`

```java
User user = threadLocal.get();
```

### `remove()`

```java
threadLocal.remove();
```

这个方法很重要，尤其是在 Web 项目里，请求结束后一定要清理。

---

## 登录用户传递的典型用法

在登录校验场景里，常见流程是这样的：

1. 用户请求进入过滤器或拦截器
2. 校验 Token 是否有效
3. 从 Token 中解析出用户信息
4. 把用户信息放进 ThreadLocal
5. 控制器或服务层直接从 ThreadLocal 里取当前用户
6. 请求结束时清理 ThreadLocal

### 示例思路

```java
public class LoginUserContext {
    private static final ThreadLocal<Integer> currentUser = new ThreadLocal<>();

    public static void setUserId(Integer userId) {
        currentUser.set(userId);
    }

    public static Integer getUserId() {
        return currentUser.get();
    }

    public static void remove() {
        currentUser.remove();
    }
}
```

### 在拦截器里保存用户

```java
Integer userId = parseUserIdFromToken(token);
LoginUserContext.setUserId(userId);
```

### 在业务代码里读取用户

```java
Integer userId = LoginUserContext.getUserId();
```

### 请求结束后清理

```java
LoginUserContext.remove();
```

---

## 注意事项

### 5.1 只适合线程内共享

`ThreadLocal` 不能跨线程传值。  
当前请求如果切到别的线程，值不会自动过去。

### 5.2 用完一定要清理

Web 容器里的线程会复用。  
如果不 `remove()`，旧数据可能被下一个请求拿到，造成脏数据或内存泄漏。

### 5.3 不适合存大对象

ThreadLocal 适合放：

- 用户 ID
- 用户名
- 登录信息

不适合放很大的对象或大量数据。

---

## 在本项目里的理解方式

可以把它理解成：

- Token 负责证明“你是谁”
- ThreadLocal 负责在**本次请求内部**保存“你是谁”

这样后面的控制器、服务层都能直接拿到当前登录用户，不需要每个方法都重复传参。

---

## 小结

`ThreadLocal` 是**线程隔离的共享变量容器**，在 Web 项目里最常见的用途就是：**把登录用户信息存起来，让同一次请求里的所有代码都能方便获取。**
