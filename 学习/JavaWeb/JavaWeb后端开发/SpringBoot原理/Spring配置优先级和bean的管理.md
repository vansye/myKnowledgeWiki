# Spring 配置优先级和 Bean 管理

## 目录

- [配置优先级](#配置优先级)
- [Bean 的管理](#bean-的管理)
- [Bean 的作用域](#bean-的作用域)
- [Bean 的生命周期](#bean-的生命周期)
- [Spring 中的 Bean 是否线程安全](#spring-中的-bean-是否线程安全)
- [第三方 Bean](#第三方-bean)
- [小结](#小结)

---

## 配置优先级

Spring Boot 允许同一个配置项出现在多个位置。  
如果多个地方都配置了同一个 key，**优先级高的会覆盖优先级低的**。

### 常见配置来源

| 配置来源 | 说明 |
|------|------|
| 命令行参数 | 启动应用时直接传入 |
| Java 系统属性 | `-Dkey=value` |
| 操作系统环境变量 | 例如 `SERVER_PORT=8081` |
| 外部配置文件 | 项目外部的配置文件 |
| 项目内部配置文件 | `application.yml`、`application.properties` |
| 代码中的默认值 | 例如 `@Value("${name:admin}")` |

### 常见结论

- **越靠外部、越靠启动时传入的配置，优先级通常越高**
- 同一个配置项，后面的高优先级来源会覆盖前面的低优先级来源

### 例子

```yaml
# application.yml
server:
  port: 8080
```

如果启动时传入：

```bash
java -jar app.jar --server.port=8081
```

最终生效的是 `8081`。

---

## Bean 的管理

Bean 是被 Spring 容器管理的对象。  
常见的管理方式有两类：

### 1. 组件扫描

Spring 会扫描带注解的类并自动创建 Bean。

- `@Component`
- `@Service`
- `@Repository`
- `@Controller`

### 2. Java 配置

用 `@Configuration` + `@Bean` 显式注册对象。

### 3. 注入和获取

常用方式有：

- `@Autowired`
- `@Resource`
- `ApplicationContext.getBean(...)`

Bean 的管理核心就是：**对象交给 Spring 创建、保存、注入和销毁**。

---

## Bean 的作用域

作用域决定了 Bean 在容器中是“单例”还是“每次新建”。

### 常见作用域

| 作用域 | 说明 | 默认情况 |
|------|------|------|
| `singleton` | 单例，容器中只有一个实例 | 是 |
| `prototype` | 多例，每次获取都会创建新对象 | 否 |
| `request` | 同一次请求内共享一个对象 | 否 |
| `session` | 同一个会话内共享一个对象 | 否 |
| `globalSession` | 全局 Session 作用域 | 否 |

### 1. singleton

默认作用域。  
容器启动时创建，或者第一次使用时创建，之后全局只用这一份。

适合：

- 无状态工具类
- Service
- Mapper

### 2. prototype

每次从容器获取都会创建新对象。

适合：

- 有状态对象
- 需要临时保存数据的对象

### 3. request / session / globalSession

这几个作用域主要和 Web 请求有关。

- `request`：同一次请求共享
- `session`：同一个会话共享
- `globalSession`：Portlet 场景常用，普通 Web 项目里较少用

---

## Bean 的生命周期

Bean 从创建到销毁，通常会经历下面这些阶段：

1. 实例化
2. 属性注入
3. 初始化前处理
4. 初始化
5. 使用
6. 销毁

### 常见扩展点

- `@PostConstruct`：初始化完成后执行
- `@PreDestroy`：销毁前执行
- `InitializingBean.afterPropertiesSet()`：初始化回调
- `DisposableBean.destroy()`：销毁回调

### 简单理解

- **创建时**：Spring 负责 new 对象并注入依赖
- **使用前**：Spring 负责初始化
- **容器关闭时**：Spring 负责清理资源

---

## Spring 中的 Bean 是否线程安全

**Spring 的 Bean 默认不是线程安全的。**

原因不是 Spring 有问题，而是：

- 默认 Bean 多数是 `singleton`
- 单例 Bean 会被多个线程同时访问
- 如果 Bean 内部保存了可变成员变量，就可能出现线程安全问题

### 安全与不安全的区别

#### 相对安全

Bean 只提供无状态方法，不保存请求相关数据。

```java
@Service
public class UserService {
    public String getName() {
        return "admin";
    }
}
```

#### 不安全

Bean 中保存了会被多个请求同时修改的成员变量。

```java
@Service
public class CounterService {
    private int count;

    public void add() {
        count++;
    }
}
```

### 结论

- 单例 Bean 要尽量设计成**无状态**
- 如果确实要存状态，要自己处理并发问题
- 线程安全问题通常出在**成员变量**，不是出在方法局部变量

---

## 第三方 Bean

第三方 Bean 指的是：**不是你自己写的类，但你想交给 Spring 管理**。

### 为什么要注册第三方 Bean

这样就可以：

- 统一交给 Spring 管理
- 方便注入到其他类中
- 统一配置初始化参数

### 常见写法

```java
@Configuration
public class MyConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

如果要给第三方对象设置参数，也可以在创建时直接配置：

```java
@Bean
public DruidDataSource dataSource() {
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setUrl("jdbc:mysql://localhost:3306/test");
    dataSource.setUsername("root");
    dataSource.setPassword("123456");
    return dataSource;
}
```

### 使用场景

- JSON 工具类
- 数据库连接池
- 日期格式化工具
- 第三方 SDK 对象

---

## 小结

这份笔记可以记成 4 句话：

1. **配置优先级**：同一个配置项，优先级高的覆盖低的。  
2. **Bean 管理**：对象交给 Spring 容器创建、注入和销毁。  
3. **作用域和生命周期**：决定 Bean 是单例、多例，还是跟请求/会话绑定。  
4. **第三方 Bean**：用 `@Bean` 把外部类交给 Spring 管理。  

如果你先记住这 4 点，再去看 Spring 配置和 Bean 相关代码，就会清楚很多。
