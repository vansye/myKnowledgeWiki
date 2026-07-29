# MyBatis 持久层框架

## 目录

- [什么是 MyBatis](#什么是-mybatis)
- [为什么需要 MyBatis](#为什么需要-mybatis)
- [MyBatis 的核心概念](#mybatis-的核心概念)
- [MyBatis 的基本使用](#mybatis-的基本使用)
- [XML 映射文件](#xml-映射文件)
- [动态 SQL](#动态-sql)
- [MyBatis 与 SpringBoot 整合](#mybatis-与-springboot-整合)
- [MyBatis 的缓存机制](#mybatis-的缓存机制)
- [常见问题与解决方案](#常见问题与解决方案)
- [小结](#小结)

---

## 什么是 MyBatis

**MyBatis** 是一个**持久层框架**，负责 Java 程序与数据库之间的交互。速查见 MyBatis，常与 事务管理 配合使用保证数据一致性。

它的核心作用是：**将 Java 对象映射到数据库表，将 SQL 语句映射到 Java 方法**。

### 生活中的类比

把 MyBatis 想象成**翻译官**：

- Java 程序说"Java 语言"（对象、方法）
- 数据库说"SQL 语言"（表、字段）
- MyBatis 负责在两者之间翻译：
  - Java 对象 → SQL 语句（增删改查）
  - SQL 结果 → Java 对象

---

## 为什么需要 MyBatis

### 没有 MyBatis 的问题

**直接用 JDBC 操作数据库：**

```java
// 原始 JDBC 代码（繁琐）
public Emp findById(Long id) {
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = DriverManager.getConnection(url, user, password);
        ps = conn.prepareStatement("SELECT * FROM emp WHERE id = ?");
        ps.setLong(1, id);
        rs = ps.executeQuery();
        if (rs.next()) {
            Emp emp = new Emp();
            emp.setId(rs.getLong("id"));
            emp.setName(rs.getString("name"));
            emp.setAge(rs.getInt("age"));
            // ... 每个字段都要手动设置
            return emp;
        }
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        // 还要手动关闭连接...
    }
    return null;
}
```

**问题：**

- 代码冗长，大量重复的 JDBC 模板代码
- 手动处理结果集映射（ResultSet → Java 对象）
- 手动管理数据库连接（打开、关闭）
- SQL 和 Java 代码混在一起

### MyBatis 的解决方案

```java
// MyBatis 方式（简洁）
@Mapper
public interface EmpMapper {
    @Select("SELECT * FROM emp WHERE id = #{id}")
    Emp findById(Long id);
}

// 使用
Emp emp = empMapper.findById(1L);  // 一行搞定！
```

**MyBatis 的好处：**

| 好处 | 说明 |
|------|------|
| **代码简洁** | 不需要写 JDBC 模板代码 |
| **自动映射** | SQL 结果自动转换为 Java 对象 |
| **SQL 灵活** | 支持复杂 SQL 和动态 SQL |
| **解耦** | SQL 可以写在 XML 中，和 Java 代码分离 |

---

## MyBatis 的核心概念

| 概念 | 说明 |
|------|------|
| **Mapper 接口** | 定义数据库操作的方法（如 findById、insert） |
| **SQL 映射** | 将 SQL 语句和方法关联（注解或 XML） |
| **参数映射** | 将 Java 参数映射到 SQL 的 `#{}` 占位符 |
| **结果映射** | 将 SQL 查询结果映射为 Java 对象 |
| **动态 SQL** | 根据条件动态生成 SQL 语句 |

---

## MyBatis 的基本使用

### 1. Mapper 接口

```java
@Mapper  // 标记为 MyBatis 的 Mapper 接口
public interface EmpMapper {

    // 查询
    @Select("SELECT * FROM emp WHERE id = #{id}")
    Emp findById(Long id);

    @Select("SELECT * FROM emp")
    List<Emp> findAll();

    // 新增
    @Insert("INSERT INTO emp(name, age, gender) VALUES(#{name}, #{age}, #{gender})")
    void insert(Emp emp);

    // 修改
    @Update("UPDATE emp SET name=#{name}, age=#{age} WHERE id=#{id}")
    void update(Emp emp);

    // 删除
    @Delete("DELETE FROM emp WHERE id = #{id}")
    void delete(Long id);
}
```

### 2. 参数传递

| 方式 | 说明 | 示例 |
|------|------|------|
| `#{}` | 预编译参数（防 SQL 注入） | `WHERE id = #{id}` |
| `${}` | 字符串拼接（有 SQL 注入风险） | `ORDER BY ${columnName}` |

**要点：**

- **优先用 `#{}`**：预编译，安全
- **慎用 `${}`**：直接拼接，有 SQL 注入风险，只用于动态表名、列名等

### 3. 多个参数传递

```java
// 方式一：用 @Param 注解（推荐）
@Select("SELECT * FROM emp WHERE name = #{name} AND gender = #{gender}")
List<Emp> findByNameAndGender(@Param("name") String name, @Param("gender") Integer gender);

// 方式二：用实体类
@Insert("INSERT INTO emp(name, age) VALUES(#{name}, #{age})")
void insert(Emp emp);

// 方式三：用 Map
@Select("SELECT * FROM emp WHERE name = #{name}")
Emp findByName(Map<String, Object> params);
```

---

## XML 映射文件

当 SQL 比较复杂时，用 XML 文件比注解更清晰。

### 基本结构

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace 对应 Mapper 接口的全类名 -->
<mapper namespace="com.itheima.mapper.EmpMapper">

    <!-- 查询 -->
    <select id="findAll" resultType="com.itheima.pojo.Emp">
        SELECT * FROM emp
    </select>

    <select id="findById" resultType="com.itheima.pojo.Emp">
        SELECT * FROM emp WHERE id = #{id}
    </select>

    <!-- 新增 -->
    <insert id="insert">
        INSERT INTO emp(name, age, gender) VALUES(#{name}, #{age}, #{gender})
    </insert>

    <!-- 修改 -->
    <update id="update">
        UPDATE emp SET name=#{name}, age=#{age} WHERE id=#{id}
    </update>

    <!-- 删除 -->
    <delete id="delete">
        DELETE FROM emp WHERE id = #{id}
    </delete>

</mapper>
```

### 要点

- `namespace`：对应 Mapper 接口的全类名
- `id`：对应 Mapper 接口中的方法名
- `resultType`：返回结果的 Java 类型
- `parameterType`：参数类型（通常可以省略，MyBatis 自动推断）

---

## 动态 SQL

动态 SQL 是 MyBatis 的强大功能，可以根据条件动态生成 SQL 语句。

### 常用动态 SQL 标签

| 标签 | 说明 |
|------|------|
| `<if>` | 条件判断 |
| `<where>` | 自动处理 WHERE 关键字和多余的 AND/OR |
| `<set>` | 自动处理 SET 关键字和多余的逗号 |
| `<foreach>` | 循环遍历 |
| `<choose>/<when>/<otherwise>` | 多选一（类似 switch） |
| `<trim>` | 自定义字符串截取 |

### `<if>` 条件判断

```xml
<select id="selectByCondition" resultType="Emp">
    SELECT * FROM emp
    WHERE
    <if test="name != null">
        name LIKE CONCAT('%', #{name}, '%')
    </if>
    <if test="gender != null">
        AND gender = #{gender}
    </if>
    <if test="begin != null and end != null">
        AND entrydate BETWEEN #{begin} AND #{end}
    </if>
</select>
```

**问题：** 如果 `name` 为 null，SQL 会变成 `WHERE AND gender = ?`，语法错误。

### `<where>` 自动处理 WHERE

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
        <if test="begin != null and end != null">
            AND entrydate BETWEEN #{begin} AND #{end}
        </if>
    </where>
</select>
```

**要点：**

- `<where>` 会自动添加 WHERE 关键字
- 会自动去掉第一个条件前多余的 AND 或 OR
- 如果所有条件都不满足，不生成 WHERE 子句

### `<set>` 动态更新

```xml
<update id="update">
    UPDATE emp
    <set>
        <if test="name != null">name = #{name},</if>
        <if test="age != null">age = #{age},</if>
        <if test="gender != null">gender = #{gender},</if>
    </set>
    WHERE id = #{id}
</update>
```

**要点：**

- `<set>` 会自动添加 SET 关键字
- 会自动去掉最后一个字段后多余的逗号

### `<foreach>` 循环遍历

```xml
<!-- 批量删除 -->
<delete id="deleteBatch">
    DELETE FROM emp WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>
```

**属性说明：**

| 属性 | 说明 |
|------|------|
| `collection` | 要遍历的集合（参数名） |
| `item` | 当前元素的变量名 |
| `open` | 开始符号 |
| `close` | 结束符号 |
| `separator` | 分隔符 |

**生成的 SQL：**

```sql
DELETE FROM emp WHERE id IN (1, 2, 3, 4, 5)
```

### `<choose>` 多选一

```xml
<select id="selectByCondition" resultType="Emp">
    SELECT * FROM emp
    <where>
        <choose>
            <when test="name != null">
                AND name = #{name}
            </when>
            <when test="gender != null">
                AND gender = #{gender}
            </when>
            <otherwise>
                AND status = 1
            </otherwise>
        </choose>
    </where>
</select>
```

**要点：**

- 类似 Java 的 switch-case
- 只执行第一个满足条件的 `<when>`
- 都不满足时执行 `<otherwise>`

---

## MyBatis 与 SpringBoot 整合

### 1. 引入依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```

### 2. 配置数据源

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/tlias?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
```

### 3. 配置 MyBatis

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true  # 下划线转驼峰（数据库字段 → Java 属性）
  mapper-locations: classpath:mapper/*.xml  # XML 映射文件位置
```

### 4. 启动类加注解

```java
@SpringBootApplication
@MapperScan("com.itheima.mapper")  // 扫描 Mapper 接口
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## MyBatis 的缓存机制

### 一级缓存（默认开启）

- 作用域：**SqlSession**（一次数据库会话）
- 同一个 SqlSession 中，相同的查询会先查缓存
- SqlSession 关闭或执行了增删改操作后，缓存清空

```java
// 第一次查询：查数据库，结果放入一级缓存
Emp emp1 = empMapper.findById(1L);

// 第二次查询：从一级缓存获取，不查数据库
Emp emp2 = empMapper.findById(1L);
```

### 二级缓存（需要手动开启）

- 作用域：**Mapper**（全局）
- 多个 SqlSession 共享
- 需要在 Mapper 接口上加 `@CacheNamespace` 或在 XML 中加 `<cache/>`

```xml
<mapper namespace="com.itheima.mapper.EmpMapper">
    <cache/>  <!-- 开启二级缓存 -->
    <!-- ... -->
</mapper>
```

**注意：** 分布式环境下不建议使用 MyBatis 二级缓存，建议用 Redis 等分布式缓存。

---

## 常见问题与解决方案

### 问题 1：`BindingException: Invalid bound statement`

**错误信息：**

```
Invalid bound statement (not found): com.itheima.mapper.EmpMapper.findById
```

**原因：**

- XML 文件的 namespace 和 Mapper 接口不匹配
- XML 文件没有被扫描到
- 方法名和 XML 中的 id 不匹配

**解决方案：**

- 检查 namespace 是否是 Mapper 接口的全类名
- 检查 `mybatis.mapper-locations` 配置是否正确
- 检查方法名和 XML 中的 id 是否一致

### 问题 2：字段映射不上（返回 null）

**原因：**

数据库字段名（下划线）和 Java 属性名（驼峰）不一致。

**解决方案：**

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true  # 开启下划线转驼峰
```

或者在 SQL 中用别名：

```sql
SELECT emp_name AS empName, create_time AS createTime FROM emp
```

### 问题 3：`#{}` 和 `${}` 的区别

| 特性 | `#{}` | `${}` |
|------|-------|-------|
| 处理方式 | 预编译（PreparedStatement） | 字符串拼接 |
| 安全性 | 防 SQL 注入 | 有 SQL 注入风险 |
| 使用场景 | 参数值 | 动态表名、列名、ORDER BY |

### 问题 4：批量操作性能差

**原因：**

循环调用单条插入，每次都要和数据库交互。

**解决方案：**

```xml
<!-- 批量插入 -->
<insert id="insertBatch">
    INSERT INTO emp(name, age) VALUES
    <foreach collection="list" item="emp" separator=",">
        (#{emp.name}, #{emp.age})
    </foreach>
</insert>
```

---

## 小结

### 核心要点

| 要点 | 说明 |
|------|------|
| **MyBatis** | 持久层框架，Java 对象和 SQL 之间的映射 |
| **Mapper 接口** | 定义数据库操作方法 |
| **注解方式** | `@Select`、`@Insert`、`@Update`、`@Delete` |
| **XML 方式** | 复杂 SQL 写在 XML 中 |
| **`#{}` vs `${}`** | 前者预编译（安全），后者拼接（有风险） |
| **动态 SQL** | `<if>`、`<where>`、`<set>`、`<foreach>` |
| **一级缓存** | SqlSession 级别，默认开启 |
| **二级缓存** | Mapper 级别，需手动开启 |

### 口诀

**MyBatis 做映射，Java SQL 互翻译；  
Mapper 接口定方法，注解 XML 写 SQL；  
井号预编译安全，美元拼接有风险；  
where 标签去多余，foreach 循环做批量。**

### 最容易混淆的三个点

- **`#{}` vs `${}`**：前者是预编译参数（安全），后者是字符串拼接（有注入风险）
- **注解 vs XML**：简单 SQL 用注解，复杂 SQL 用 XML
- **一级缓存 vs 二级缓存**：一级是 SqlSession 级别（默认开启），二级是 Mapper 级别（需手动开启）

---

MyBatis 的本质是：**在 Java 对象和 SQL 之间建立映射关系，让程序员用 Java 方法的方式操作数据库，同时保留 SQL 的灵活性。**

掌握 MyBatis，你就能高效地操作数据库，写出简洁的持久层代码！
