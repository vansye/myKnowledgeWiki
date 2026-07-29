# Spring IOC 与 DI 依赖注入

## 目录

- [什么是 IOC](#什么是-ioc)
- [为什么需要 IOC](#为什么需要-ioc)
- [IOC 的核心概念](#ioc-的核心概念)
- [Bean 的创建方式](#bean-的创建方式)
- [什么是 DI](#什么是-di)
- [DI 的注入方式](#di-的注入方式)
- [IOC 和 DI 的关系](#ioc-和-di-的关系)
- [Bean 的作用域](#bean-的作用域)
- [Bean 的生命周期](#bean-的生命周期)
- [常见问题与解决方案](#常见问题与解决方案)
- [小结](#小结)

---

## 什么是 IOC

**IOC（Inversion of Control，控制反转）** 是 Spring 框架的核心思想。

它的含义是：**将对象的创建和管理权从程序员手中交给 Spring 容器**。速查版见 IoC和DI，Beans 概念见 Bean。

### 传统方式 vs IOC

```java
// 传统方式：程序员自己控制对象的创建
public class UserController {
    private UserService userService = new UserServiceImpl();  // 自己 new

    public void getUser() {
        userService.getUser();
    }
}
```

```java
// IOC 方式：Spring 容器控制对象的创建
@RestController
public class UserController {

    @Autowired  // Spring 自动注入
    private UserService userService;

    public void getUser() {
        userService.getUser();
    }
}
```

### 生活中的类比

**传统方式** 就像你自己做饭：

- 自己去买菜 → 自己洗菜 → 自己炒菜 → 自己洗碗
- 每个步骤都要自己控制

**IOC 方式** 就像去餐厅吃饭：

- 你只负责点菜（声明需要什么）
- 餐厅负责买菜、洗菜、炒菜、上菜（Spring 容器负责创建和管理对象）
- 你不需要关心菜是怎么做的

**"控制反转"反转了什么？**

- 传统：程序员控制对象的创建（`new`）
- IOC：Spring 容器控制对象的创建
- 控制权从程序员"反转"到了容器

---

## 为什么需要 IOC

### 没有 IOC 的问题

```java
public class UserController {
    private UserService userService = new UserServiceImpl();
}

public class UserServiceImpl implements UserService {
    private UserMapper userMapper = new UserMapperImpl();
}
```

| 问题 | 说明 |
|------|------|
| **耦合度高** | Controller 直接依赖具体的实现类，改实现类要改所有引用的地方 |
| **难以测试** | 无法用 Mock 对象替换真实依赖 |
| **代码冗余** | 每个地方都要 `new` 对象，重复代码多 |
| **管理混乱** | 对象的生命周期（创建、销毁）由程序员手动管理 |

### IOC 的好处

| 好处 | 说明 |
|------|------|
| **降低耦合** | 依赖接口而非实现类，改实现不影响使用方 |
| **便于测试** | 可以轻松注入 Mock 对象进行单元测试 |
| **统一管理** | 所有对象由容器统一管理，生命周期清晰 |
| **代码简洁** | 不需要手动 `new` 对象，用注解声明即可 |

---

## IOC 的核心概念

### Bean

**Bean 就是被 Spring 容器管理的对象。**

- 传统方式：对象由 `new` 创建
- IOC 方式：对象由 Spring 容器创建和管理，这个对象就叫 Bean

### Spring 容器

Spring 容器是 IOC 的载体，负责：

- 创建 Bean
- 管理 Bean 的生命周期
- 注入 Bean 的依赖

在 SpringBoot 中，容器是自动启动的，不需要手动创建。

### 组件扫描

Spring 启动时会自动扫描指定包下的类，找到带注解的类并创建 Bean。

```java
@SpringBootApplication  // 默认扫描启动类所在包及其子包
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## Bean 的创建方式

### 方式一：组件扫描（最常用）

在类上加注解，Spring 自动扫描并创建 Bean。

| 注解 | 使用场景 | 说明 |
|------|----------|------|
| `@Component` | 通用组件 | 所有组件都可以用 |
| `@Service` | 业务层 | 用于 Service 类 |
| `@Controller` | 控制层 | 用于 Controller 类 |
| `@RestController` | 控制层（返回 JSON） | `@Controller` + `@ResponseBody` |
| `@Repository` | 数据访问层 | 用于 Dao/Mapper 类 |

```java
@Service  // Spring 自动创建这个 Bean
public class UserServiceImpl implements UserService {
    // ...
}
```

**要点：**

- `@Service`、`@Controller`、`@Repository` 本质上都是 `@Component`
- 用不同的注解是为了**语义清晰**，让人一看就知道这个类属于哪一层
- SpringBoot 默认扫描启动类所在包及其子包

### 方式二：`@Bean` 注解（用于第三方类）

当类不是自己写的（如第三方库的类），无法加 `@Component` 时，用 `@Bean` 在配置类中创建。

```java
@Configuration  // 标记为配置类
public class MyConfig {

    @Bean  // 方法的返回值会被注册为 Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));
        return mapper;
    }
}
```

**要点：**

- `@Bean` 用在方法上，方法名默认就是 Bean 的名称
- 可以用 `@Bean("自定义名称")` 指定名称
- 适合创建第三方对象或需要复杂初始化的对象

### 方式三：XML 配置（传统方式，了解即可）

```xml
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>
```

SpringBoot 项目中基本不用 XML 配置了。

---

## 什么是 DI

**DI（Dependency Injection，依赖注入）** 是 IOC 的具体实现方式。

IOC 是思想（把对象管理权交给容器），DI 是实现（容器自动把依赖的对象注入进来）。

```java
@RestController
public class UserController {

    @Autowired  // DI：Spring 自动把 UserService 的 Bean 注入到这里
    private UserService userService;
}
```

**过程：**

1. Spring 容器启动，扫描到 `UserServiceImpl` 上的 `@Service`，创建 Bean
2. 扫描到 `UserController` 上的 `@RestController`，创建 Bean
3. 发现 `UserController` 中有 `@Autowired` 的 `UserService` 字段
4. 从容器中找到 `UserService` 类型的 Bean，注入到 `UserController` 中

---

## DI 的注入方式

### 1. 字段注入（最常用）

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;
}
```

**优点：** 写法简单
**缺点：** 不利于单元测试（无法通过构造器传入 Mock 对象）

### 2. 构造器注入（推荐）

```java
@RestController
public class UserController {

    private final UserService userService;

    // Spring 自动通过构造器注入
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

**简化写法**（只有一个构造器时，`@Autowired` 可以省略）：

```java
@RestController
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

**优点：**

- 依赖关系清晰，一眼就能看出这个类依赖了什么
- 可以用 `final` 修饰，保证不可变
- 便于单元测试

### 3. Setter 注入

```java
@RestController
public class UserController {

    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

**优点：** 可以在运行时动态修改依赖
**缺点：** 对象可能处于不完整状态（依赖还没注入就被使用）

### 三种注入方式对比

| 方式 | 写法 | 优点 | 缺点 | 推荐度 |
|------|------|------|------|--------|
| 字段注入 | `@Autowired` 在字段上 | 简单 | 不利于测试，不能 final | 一般 |
| 构造器注入 | `@Autowired` 在构造器上 | 清晰、可 final、便于测试 | 依赖多时构造器参数长 | **推荐** |
| Setter 注入 | `@Autowired` 在 setter 上 | 可动态修改 | 对象可能不完整 | 少用 |

### `@Autowired` vs `@Resource`

| 特性 | `@Autowired` | `@Resource` |
|------|-------------|-------------|
| 来源 | Spring 框架 | Java 标准（JSR-250） |
| 注入方式 | 默认按**类型**注入 | 默认按**名称**注入 |
| 多个实现时 | 需要 `@Qualifier` 指定 | 直接按字段名匹配 Bean 名称 |
| 推荐场景 | Spring 项目中 | 需要按名称注入时 |

```java
// @Autowired：按类型找，如果有多个实现需要 @Qualifier
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;

// @Resource：按名称找，字段名就是 Bean 名称
@Resource
private UserService userService;  // 找名为 "userService" 的 Bean
```

---

## IOC 和 DI 的关系

```
IOC（控制反转）
  │
  ├── 思想层面：对象的创建和管理权交给容器
  │
  └── 实现层面：DI（依赖注入）
        │
        ├── 字段注入
        ├── 构造器注入
        └── Setter 注入
```

**一句话总结：**

- **IOC** 是目标：让容器管理对象
- **DI** 是手段：容器把对象注入到需要的地方
- 两者是同一件事的不同角度

---

## Bean 的作用域

作用域决定了 Bean 在容器中创建几个实例。

| 作用域 | 说明 | 使用场景 |
|--------|------|----------|
| `singleton`（默认） | 容器中只有一个实例 | Service、Controller、Mapper |
| `prototype` | 每次获取都创建新实例 | 有状态的对象 |
| `request` | 每个 HTTP 请求一个实例 | Web 请求相关 |
| `session` | 每个 HTTP Session 一个实例 | 用户会话相关 |

### singleton（单例）

```java
@Service  // 默认 singleton
public class UserService {
    // 整个应用中只有一个 UserService 实例
}
```

**要点：**

- 默认作用域，容器启动时或第一次使用时创建
- 所有地方注入的都是同一个对象
- 适合无状态的对象（如 Service、Controller）

### prototype（多例）

```java
@Service
@Scope("prototype")  // 每次获取都创建新实例
public class ShoppingCart {
    // 每次注入都是新的 ShoppingCart 对象
}
```

**要点：**

- 每次从容器获取都创建新对象
- 适合有状态的对象（如购物车）

---

## Bean 的生命周期

Bean 从创建到销毁经历以下阶段：

```
1. 实例化（Spring 调用构造器创建对象）
        │
2. 属性注入（Spring 注入依赖，如 @Autowired）
        │
3. 初始化前处理（BeanPostProcessor.postProcessBeforeInitialization）
        │
4. 初始化（@PostConstruct / InitializingBean.afterPropertiesSet）
        │
5. 使用（Bean 可以被使用了）
        │
6. 销毁前处理（@PreDestroy / DisposableBean.destroy）
        │
7. 销毁（Bean 被移除）
```

### 常用生命周期注解

```java
@Service
public class MyService {

    @PostConstruct  // 初始化完成后执行（只执行一次）
    public void init() {
        System.out.println("Bean 初始化完成");
    }

    @PreDestroy  // 容器关闭前执行（只执行一次）
    public void destroy() {
        System.out.println("Bean 即将销毁");
    }
}
```

---

## 常见问题与解决方案

### 问题 1：`NoSuchBeanDefinitionException`

**错误信息：**

```
No qualifying bean of type 'com.itheima.service.UserService' available
```

**原因：**

- 类上没有加 `@Service`/`@Component` 等注解
- 类不在 Spring 的包扫描范围内

**解决方案：**

- 检查类上是否有组件注解
- 检查包路径是否在启动类的子包下

### 问题 2：`NoUniqueBeanDefinitionException`

**错误信息：**

```
No unique bean of type 'com.itheima.service.UserService' available:
expected single matching bean but found 2: userServiceImpl, userServiceImpl2
```

**原因：**

接口有多个实现类，Spring 不知道注入哪个。

**解决方案：**

```java
// 方式一：用 @Qualifier 指定
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;

// 方式二：用 @Resource 按名称注入
@Resource
private UserService userServiceImpl;  // 按字段名匹配
```

### 问题 3：循环依赖

**错误信息：**

```
The dependencies of some of the beans in the application context form a cycle
```

**原因：**

A 依赖 B，B 又依赖 A，形成循环。

```java
@Service
public class A {
    @Autowired
    private B b;  // A 依赖 B
}

@Service
public class B {
    @Autowired
    private A a;  // B 依赖 A → 循环！
}
```

**解决方案：**

- 重新设计，打破循环依赖
- 用 `@Lazy` 延迟加载其中一个依赖

```java
@Service
public class B {
    @Autowired
    @Lazy  // 延迟加载，先不注入，等真正使用时再注入
    private A a;
}
```

### 问题 4：`@Autowired` 注入为 null

**原因：**

- 对象不是由 Spring 管理的（自己 `new` 出来的）
- 静态字段不能用 `@Autowired` 注入

**解决方案：**

- 确保对象由 Spring 容器管理（加注解）
- 静态字段用 setter 方式注入

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **IOC** | 控制反转，对象创建权交给 Spring 容器 |
| **DI** | 依赖注入，容器自动注入依赖对象 |
| **Bean** | 被 Spring 容器管理的对象 |
| **组件注解** | `@Component`、`@Service`、`@Controller`、`@Repository` |
| **`@Bean`** | 在配置类中创建第三方 Bean |
| **注入方式** | 字段注入、构造器注入（推荐）、Setter 注入 |
| **作用域** | singleton（默认单例）、prototype（多例） |
| **生命周期** | 实例化 → 注入 → 初始化 → 使用 → 销毁 |

### 口诀

**IOC 反转控制权，DI 注入靠容器；  
Component 扫组件，Bean 注解建第三方；  
Autowired 按类型，Resource 按名称；  
构造注入最推荐，单例多例看场景。**

### 最容易混淆的三个点

- **IOC vs DI**：IOC 是思想（控制反转），DI 是实现（依赖注入），本质是同一件事
- **`@Component` vs `@Bean`**：`@Component` 加在类上（自己写的类），`@Bean` 加在方法上（第三方类）
- **`@Autowired` vs `@Resource`**：前者按类型注入（Spring 的），后者按名称注入（Java 标准的）

---

IOC 和 DI 是 Spring 框架的基石，理解了它们，就理解了 Spring 的核心思想：**把对象的创建和管理交给容器，程序员只关心业务逻辑。**
