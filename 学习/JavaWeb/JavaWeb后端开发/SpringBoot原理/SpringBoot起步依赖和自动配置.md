# SpringBoot 起步依赖和自动配置

## 目录

- [SpringBoot 解决了什么问题](#springboot-解决了什么问题)
- [起步依赖](#起步依赖)
- [自动配置是什么](#自动配置是什么)
- [引入第三方 Bean 的三种方案](#引入第三方-bean-的三种方案)
- [@SpringBootApplication 注解剖析](#springbootapplication-注解剖析)
- [自动配置的原理](#自动配置的原理)
- [@Conditional 条件装配](#conditional-条件装配)
- [自动配置全流程串一遍](#自动配置全流程串一遍)
- [自定义 starter](#自定义-starter)
- [小结](#小结)

---

## SpringBoot 解决了什么问题

回想一下传统 Spring 开发的两大痛点：

### 痛点一：依赖管理繁琐

想搭一个 Web 项目，要自己引入一大堆依赖：

- spring-core、spring-web、spring-webmvc
- tomcat、jackson、日志框架……

而且**版本之间还要互相兼容**，版本冲突了项目直接起不来。

### 痛点二：配置繁琐

要写大量的 XML 或配置类：

- 配置 DispatcherServlet
- 配置视图解析器
- 配置数据源、事务管理器……

这些配置在每个项目里几乎都长一样，纯粹是重复劳动。

### SpringBoot 的答案

| 痛点 | SpringBoot 的解决方案 |
|------|------|
| 依赖管理繁琐 | **起步依赖（starter）** |
| 配置繁琐 | **自动配置（Auto Configuration）** |

这两个特性就是 SpringBoot 的两大核心，也是这篇笔记的主角。

---

## 起步依赖

### 什么是起步依赖

起步依赖（starter）本质上是**一组 Maven 依赖的打包集合**。

你只需要引入一个 starter，它背后的一整套依赖会通过 **Maven 依赖传递** 自动带进来，而且版本都是官方测试过、互相兼容的。

### 例子：web 起步依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

只写了这一个依赖，但它会传递引入：

- spring-web / spring-webmvc（MVC 框架）
- 内嵌 Tomcat（不用自己装服务器）
- jackson（JSON 序列化）
- spring-boot-starter（日志、YAML 解析等基础能力）

> 注意：老教程里这个 starter 叫 `spring-boot-starter-web`。  
> 本项目用的是 **SpringBoot 4.x**，官方把它改名成了 `spring-boot-starter-webmvc`（见 `web-management/pom.xml`），作用完全一样。

### 为什么不用写版本号

注意上面的依赖**没有写 `<version>`**，因为版本由 parent 统一管理：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.6</version>
</parent>
```

`spring-boot-starter-parent` 内部继承了 `spring-boot-dependencies`，里面用 `<dependencyManagement>` **锁定了几百个常用依赖的版本号**。

所以：

- 官方 starter：不用写版本，parent 已锁定
- 第三方依赖（如 `mybatis-spring-boot-starter`）：parent 没管的才需要自己写版本

### 常见 starter

| starter | 作用 |
|------|------|
| `spring-boot-starter-webmvc` | Web 开发（MVC + 内嵌 Tomcat） |
| `spring-boot-starter-test` | 单元测试（JUnit 等） |
| `spring-boot-starter-aop` | AOP 切面编程 |
| `spring-boot-starter-data-redis` | Redis 操作 |
| `mybatis-spring-boot-starter` | MyBatis 整合 |
| `pagehelper-spring-boot-starter` | 分页插件 |

### 命名规范

从名字就能看出是谁提供的：

- **官方**提供：`spring-boot-starter-xxx`，如 `spring-boot-starter-aop`
- **第三方**提供：`xxx-spring-boot-starter`，如 `mybatis-spring-boot-starter`、`pagehelper-spring-boot-starter`

（对照本项目 pom.xml，这两种命名都出现了。）

### 一句话总结

**起步依赖 = 依赖打包 + 版本锁定，底层靠 Maven 依赖传递。**

---

## 自动配置是什么

自动配置指的是：**SpringBoot 启动时，自动把第三方 jar 包里的配置类和 Bean 加载进 Spring 容器，不需要我们手动配置。**

### 直观感受

我们从来没写过下面这些 Bean 的配置，但它们都能直接注入使用：

- `DispatcherServlet`（前端控制器）
- `DataSource`（数据源）
- `SqlSessionFactory`（MyBatis 核心对象）

比如只在 `application.yml` 里写了数据库连接信息，`Mapper` 就能直接用了——是谁把 `SqlSessionFactory` 放进容器的？

答案就是：**MyBatis 的 starter 里带了自动配置类，SpringBoot 启动时自动加载了它。**

### 要搞清楚的核心问题

自动配置的原理，本质上就是回答一个问题：

> **第三方 jar 包里的 Bean，是怎么进入 Spring 容器的？**

---

## 引入第三方 Bean 的三种方案

在看 SpringBoot 的做法之前，先看看如果**手动**引入第三方 Bean 有哪些方案，就能理解 SpringBoot 为什么这么设计。

### 方案一：@ComponentScan 扫描第三方包

```java
@SpringBootApplication
@ComponentScan({"luv.vansye", "com.example.thirdparty"})
public class WebManagementApplication { }
```

思路：把第三方的包也加进扫描范围。

**缺点：**

- 要知道第三方 Bean 在哪些包下，用一个框架加一个，**繁琐**
- 扫描范围变大，**启动性能下降**

所以 SpringBoot 默认只扫描**启动类所在包及其子包**，不推荐这种方式。

### 方案二：@Import 导入

`@Import` 可以强行把类塞进容器，有三种用法：

#### 1. 导入普通类

```java
@Import(TokenParser.class)
@SpringBootApplication
public class WebManagementApplication { }
```

#### 2. 导入配置类

```java
@Import(HeaderConfig.class)   // HeaderConfig 是 @Configuration 类，里面的 @Bean 全部生效
@SpringBootApplication
public class WebManagementApplication { }
```

#### 3. 导入 ImportSelector 实现类

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 返回要导入容器的类的全类名数组
        return new String[]{"com.example.HeaderConfig"};
    }
}
```

```java
@Import(MyImportSelector.class)
@SpringBootApplication
public class WebManagementApplication { }
```

这种方式最灵活：**要导入哪些类，可以写逻辑动态决定**（比如从配置文件里读）。

**缺点：** 使用者还是得知道该导入第三方的哪些类，心智负担在使用者这边。

### 方案三：第三方提供 @EnableXxx 注解（最优）

让**第三方框架自己**封装一个注解，内部包好 `@Import`：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyImportSelector.class)   // 该导什么，第三方自己最清楚
public @interface EnableHeaderConfig { }
```

使用者只需要：

```java
@EnableHeaderConfig
@SpringBootApplication
public class WebManagementApplication { }
```

**优点：** 使用者不需要关心第三方内部结构，加一个注解就完事。

这就是很多框架 `@EnableScheduling`、`@EnableCaching`、`@EnableAsync` 的套路——而 SpringBoot 的自动配置，正是把这个思想做到了极致。

---

## @SpringBootApplication 注解剖析

启动类上的 `@SpringBootApplication` 是一个组合注解，拆开是三个核心注解：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication { }
```

| 注解 | 作用 |
|------|:-----|
| `@SpringBootConfiguration` | 本质就是 `@Configuration`，声明启动类本身是一个配置类 |
| `@ComponentScan` | 组件扫描，默认扫描**启动类所在包及其子包** |
| `@EnableAutoConfiguration` | **自动配置的总开关**，核心中的核心 |

### 两个常见追问

**为什么启动类要放在最外层的包？**  
因为 `@ComponentScan` 默认只扫启动类所在包及子包，放里层会导致外面的 Bean 扫不到。

**为什么我们自己写的 `@Service`、`@Mapper` 能被扫到？**  
靠的是 `@ComponentScan`，和自动配置无关。**自动配置解决的是第三方 jar 包里的 Bean**。

---

## 自动配置的原理

现在拆开 `@EnableAutoConfiguration` 看它是怎么工作的。

### 第一层：还是 @Import 的套路

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration { }
```

看到了吗？就是上面**方案三**：`@EnableXxx` 注解 + `@Import(ImportSelector 实现类)`。

### 第二层：AutoConfigurationImportSelector 做了什么

`AutoConfigurationImportSelector` 实现了 `ImportSelector` 接口，它的 `selectImports()` 方法会去**读一个约定好的文件**，拿到所有自动配置类的全类名。

### 第三层：约定的文件在哪

**当前版本（SpringBoot 2.7+ / 3.x / 4.x，即本项目）：**

```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

文件内容就是一行一个自动配置类的全类名，比如 MyBatis starter 的 jar 包里：

```
org.mybatis.spring.boot.autoconfigure.MybatisLanguageDriverAutoConfiguration
org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
```

**旧版本（SpringBoot 2.6 及以前，面试常问）：**

读的是 `META-INF/spring.factories` 文件中 `EnableAutoConfiguration` 对应的值。

> 面试时如果只背了 spring.factories，记得补一句：**2.7 开始改为 `.imports` 文件，3.0 彻底移除了从 spring.factories 读自动配置的支持**，能体现你了解版本演进。

### 第四层：自动配置类里是什么

打开 `MybatisAutoConfiguration` 看（简化后）：

```java
@AutoConfiguration          // 2.7+ 专用于自动配置类的注解，本质带着 @Configuration
@ConditionalOnClass({SqlSessionFactory.class, SqlSessionFactoryBean.class})
@ConditionalOnSingleCandidate(DataSource.class)
public class MybatisAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) {
        // 创建并返回 SqlSessionFactory
    }
}
```

就是普通的配置类 + `@Bean`，只是多了一堆 `@ConditionalOnXxx`——这就引出了条件装配。

---

## @Conditional 条件装配

`.imports` 文件里登记了**全量**的自动配置类（一百多个），但**并不是全部都会生效**。

每个配置类上都有条件注解，**条件满足才注册 Bean**。`@Conditional` 是这一族注解的根，常用派生注解：

| 注解 | 生效条件 |
|------|------|
| `@ConditionalOnClass` | 环境中**存在指定的类**（jar 包引了）才生效 |
| `@ConditionalOnMissingClass` | 环境中不存在指定类才生效 |
| `@ConditionalOnBean` | 容器中存在指定 Bean 才生效 |
| `@ConditionalOnMissingBean` | 容器中**不存在**指定 Bean 才生效 |
| `@ConditionalOnProperty` | 配置文件中存在指定属性（且值匹配）才生效 |

### 两个最重要的注解

#### 1. @ConditionalOnClass —— 决定"这套自动配置要不要启动"

```java
@ConditionalOnClass(SqlSessionFactory.class)
```

含义：你项目里引入了 MyBatis 的 jar 包（classpath 中有这个类），这套 MyBatis 自动配置才生效。

**这就是为什么"引入 starter 就自动配好了"——引入依赖 = 满足条件 = 自动配置生效。**

#### 2. @ConditionalOnMissingBean —— 给用户留后门

```java
@Bean
@ConditionalOnMissingBean
public SqlSessionFactory sqlSessionFactory(...) { ... }
```

含义：容器中还没有这个 Bean 时，才用默认的。

**如果你自己配置了一个同类型的 Bean，自动配置就让位。**  
这就是 SpringBoot 的设计哲学：**约定大于配置，用户配置优先于默认配置。**

---

## 自动配置全流程串一遍

把上面的内容按启动顺序串起来：

1. 启动类标注 `@SpringBootApplication`
2. 其中的 `@EnableAutoConfiguration` 通过 `@Import` 导入了 `AutoConfigurationImportSelector`
3. `selectImports()` 读取所有 jar 包中的  
   `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件
4. 拿到**全量**自动配置类名单
5. 逐个检查配置类上的 `@ConditionalOnXxx` 条件，**过滤掉不满足条件的**
6. 满足条件的配置类生效，其中的 `@Bean` 方法执行，Bean 注册进 [[IOC与DI依赖注入\|IOC 容器]]
7. 我们就可以直接 `@Autowired` 使用了

一句话版本：

> **@SpringBootApplication → @EnableAutoConfiguration → @Import(AutoConfigurationImportSelector) → 读 .imports 文件 → 按条件筛选 → 注册 Bean**

---

## 自定义 starter

### 什么场景需要

公司内部的通用功能（比如阿里云 OSS 上传工具），十个项目都要用。  
如果每个项目都复制一遍工具类 + 配置，改一处就要改十处。

做成 starter 后：**引入依赖 + 写配置文件，直接注入使用**，和用官方 starter 一样爽。

### 模块结构（约定）

一个规范的 starter 拆成两个模块：

| 模块 | 职责 |
|------|------|
| `xxx-spring-boot-autoconfigure` | 干活的：自动配置类、核心功能代码 |
| `xxx-spring-boot-starter` | 空壳：只做依赖聚合，引入 autoconfigure 模块 |

使用者只需要引入 starter 模块。

### 实现步骤（以 aliyun-oss 为例）

**第一步：** 在 autoconfigure 模块中编写核心类和自动配置类

```java
@AutoConfiguration
@EnableConfigurationProperties(AliyunOSSProperties.class)   // 绑定 yml 中的配置项
public class AliyunOSSAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public AliyunOSSOperator aliyunOSSOperator(AliyunOSSProperties properties) {
        return new AliyunOSSOperator(properties);
    }
}
```

**第二步：** 在 autoconfigure 模块的 resources 下创建文件

```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

内容写上自动配置类的全类名：

```
com.aliyun.oss.AliyunOSSAutoConfiguration
```

**第三步：** starter 模块的 pom 中引入 autoconfigure 模块（依赖聚合）

**第四步：** 使用方引入 starter，在 `application.yml` 里写好 OSS 配置，直接注入 `AliyunOSSOperator` 使用。

### 为什么这样就生效了

回看[自动配置全流程](#自动配置全流程串一遍)：SpringBoot 启动时会扫**所有 jar 包**的 `.imports` 文件——你的 starter 也是个 jar 包，自然被扫到，配置类自然被加载。

**自定义 starter 没有任何黑魔法，就是照着官方的约定摆好文件。**

---

## 小结

这份笔记可以记成 4 句话：

1. **起步依赖**：依赖打包 + 版本锁定，底层是 Maven 依赖传递，解决"依赖管理繁琐"。
2. **自动配置**：SpringBoot 启动时自动加载第三方 jar 包里的配置类，解决"配置繁琐"。
3. **原理主线**：`@EnableAutoConfiguration` → `@Import(AutoConfigurationImportSelector)` → 读 `.imports` 文件 → `@Conditional` 条件筛选 → 注册 Bean。
4. **自定义 starter**：autoconfigure 写配置类 + `.imports` 文件登记，starter 做依赖聚合。

### 口诀

**starter 管依赖，Import 进容器，imports 文件是名单，Conditional 是门槛。**

### 最容易混淆的两个点

- **@ComponentScan 和自动配置的分工**：前者管**自己写的** Bean，后者管**第三方 jar 包里的** Bean
- **登记 ≠ 生效**：`.imports` 文件里是全量名单，真正生效的只有满足 `@Conditional` 条件的那部分
