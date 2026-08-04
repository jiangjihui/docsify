# MyBatis

## 概述

### MyBatis 是什么

MyBatis 是一个优秀的持久层框架，原生 JDBC 操作存在大量的重复性代码（如注册驱动、创建连接、创建 Statement、结果集检测等）。框架的作用就是把这些繁琐的代码封装，让程序员专注于 SQL 语句本身。

MyBatis 通过 XML 或注解的方式将要执行的 SQL 语句配置起来，并通过 Java 对象和 SQL 语句映射成最终执行的 SQL 语句。最终由 MyBatis 框架执行 SQL，并将结果映射成 Java 对象返回。

**MyBatis 的核心优势：**
- 半自动 ORM 框架，不完全自动生成 SQL
- 手动编写 SQL，灵活可控
- 支持动态 SQL
- XML 与注解两种配置方式

### MyBatis 版本演进

| 版本 | 发布年份 | 主要特性 |
|------|----------|----------|
| MyBatis 3.0 | 2010 | 全新架构，支持注解 |
| MyBatis 3.2 | 2013 | 缓存增强、批量操作 |
| MyBatis 3.3 | 2016 | 关联查询优化 |
| MyBatis 3.4 | 2017 | 日志优化、性能提升 |
| MyBatis 3.5 | 2019 | Java 8 特性支持、流式查询 |
| MyBatis 3.6 | 2022 | 完善中文文档、性能优化 |
| MyBatis 3.7+ | 后续 | 模块化重构、持续优化 |

---

## 核心概念与术语

### 四大对象

MyBatis 通过四大对象协作完成数据库操作：

| 组件 | 职责 |
|------|------|
| **Executor** | 执行器，负责 SQL 语句的生成和查询缓存的维护 |
| **StatementHandler** | 封装 JDBC Statement 操作，负责设置参数、转换结果集 |
| **ParameterHandler** | 负责将用户传递的参数转换成 JDBC Statement 需要的参数 |
| **ResultSetHandler** | 负责将 JDBC 返回的 ResultSet 结果集转换成 List 集合 |

### 核心组件

```java
// 执行流程
SqlSessionFactoryBuilder → SqlSessionFactory → SqlSession → Executor → StatementHandler → ParameterHandler → ResultSetHandler
```

| 组件 | 说明 |
|------|------|
| **SqlSession** | MyBatis 工作的主要顶层 API，表示和数据库交互的会话 |
| **Executor** | MyBatis 执行器，是 MyBatis 调度的核心 |
| **StatementHandler** | 封装 JDBC Statement 操作 |
| **ParameterHandler** | 参数转换处理器 |
| **ResultSetHandler** | 结果集转换处理器 |
| **TypeHandler** | Java 类型和 JDBC 类型之间的映射转换 |
| **MappedStatement** | 维护 select/update/delete/insert 节点封装 |
| **SqlSource** | 动态生成 SQL，封装到 BoundSql |
| **BoundSql** | 动态生成的 SQL 语句及参数信息 |
| **Configuration** | 所有配置信息的容器 |

---

## MyBatis 配置

### 全局配置文件结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 属性配置 -->
    <properties resource="db.properties">
        <property name="username" value="root"/>
    </properties>

    <!-- 设置 -->
    <settings>
        <!-- 开启驼峰命名映射 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
        <!-- 日志实现 -->
        <setting name="logImpl" value="SLF4J"/>
    </settings>

    <!-- 类型别名 -->
    <typeAliases>
        <package name="com.example.entity"/>
    </typeAliases>

    <!-- 类型处理器 -->
    <typeHandlers>
        <package name="com.example.handler"/>
    </typeHandlers>

    <!-- 环境配置 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 映射器 -->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
        <package name="com.example.mapper"/>
    </mappers>
</configuration>
```

### Spring Boot 配置

```properties
# 数据源
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# MyBatis 配置
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.example.entity
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.cache-enabled=true

# 开启日志
logging.level.com.example.mapper=DEBUG
```

**常用配置项：**

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| mybatis.mapper-locations | Mapper XML 路径 | classpath*:mapper/**/*.xml |
| mybatis.type-aliases-package | 类型别名包 | - |
| mybatis.configuration.map-underscore-to-camel-case | 驼峰映射 | false |
| mybatis.configuration.cache-enabled | 开启二级缓存 | true |
| mybatis.configuration.default-fetch-size | 默认抓取数量 | - |
| mybatis.configuration.default-statement-timeout | 默认超时时间 | - |

---

## Mapper 接口与 XML

### XML 映射文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">

    <!-- 结果映射 -->
    <resultMap id="BaseResultMap" type="User">
        <id column="id" property="id"/>
        <result column="user_name" property="userName"/>
        <result column="create_time" property="createTime"/>
    </resultMap>

    <!-- 基础列 -->
    <sql id="Base_Column_List">
        id, user_name, age, email, create_time
    </sql>

    <!-- 查询 -->
    <select id="selectById" resultMap="BaseResultMap">
        SELECT <include refid="Base_Column_List"/>
        FROM t_user
        WHERE id = #{id}
    </select>

    <!-- 插入 -->
    <insert id="insert" parameterType="User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO t_user (user_name, age, email, create_time)
        VALUES (#{userName}, #{age}, #{email}, #{createTime})
    </insert>

    <!-- 更新 -->
    <update id="updateById" parameterType="User">
        UPDATE t_user
        <set>
            <if test="userName != null">user_name = #{userName},</if>
            <if test="age != null">age = #{age},</if>
        </set>
        WHERE id = #{id}
    </update>

    <!-- 删除 -->
    <delete id="deleteById" parameterType="long">
        DELETE FROM t_user WHERE id = #{id}
    </delete>

</mapper>
```

### 注解方式

```java
@Mapper
public interface UserMapper {

    @Select("SELECT * FROM t_user WHERE id = #{id}")
    User selectById(Long id);

    @Insert("INSERT INTO t_user(user_name, age, email) VALUES(#{userName}, #{age}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(User user);

    @Update("UPDATE t_user SET user_name = #{userName}, age = #{age} WHERE id = #{id}")
    int update(User user);

    @Delete("DELETE FROM t_user WHERE id = #{id}")
    int deleteById(Long id);

    @Select("SELECT * FROM t_user WHERE user_name LIKE CONCAT('%', #{name}, '%')")
    List<User> searchByName(String name);
}
```

### 动态 SQL

**if 条件判断：**

```xml
<select id="search" resultMap="BaseResultMap">
    SELECT * FROM t_user
    <where>
        <if test="userName != null">
            AND user_name = #{userName}
        </if>
        <if test="age != null">
            AND age = #{age}
        </if>
    </where>
</select>
```

**choose/when/otherwise：**

```xml
<select id="search" resultMap="BaseResultMap">
    SELECT * FROM t_user
    <where>
        <choose>
            <when test="userName != null">
                AND user_name = #{userName}
            </when>
            <when test="age != null">
                AND age = #{age}
            </when>
            <otherwise>
                AND status = 1
            </otherwise>
        </choose>
    </where>
</select>
```

**foreach 循环：**

```xml
<!-- IN 查询 -->
<select id="selectByIds" resultMap="BaseResultMap">
    SELECT * FROM t_user
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>

<!-- 批量插入 -->
<insert id="batchInsert">
    INSERT INTO t_user (user_name, age) VALUES
    <foreach collection="users" item="user" separator=",">
        (#{user.userName}, #{user.age})
    </foreach>
</insert>
```

**set 更新：**

```xml
<update id="updateById" parameterType="User">
    UPDATE t_user
    <set>
        <if test="userName != null">user_name = #{userName},</if>
        <if test="age != null">age = #{age},</if>
    </set>
    WHERE id = #{id}
</update>
```

---

## 执行流程与原理

### 执行流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    MyBatis 配置文件                         │
│            (mybatis-config.xml + Mapper.xml)               │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                 SqlSessionFactoryBuilder                    │
│                     (Builder 模式)                          │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   SqlSessionFactory                         │
│                   (工厂模式)                                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                      SqlSession                             │
│              (数据库会话，线程不安全)                        │
└─────────┬───────────────────────────────────┬───────────────┘
          │                                   │
          ▼                                   ▼
┌─────────────────────┐           ┌─────────────────────────┐
│   CachingExecutor   │           │      SimpleExecutor     │
│    (二级缓存装饰)    │           │     (BaseExecutor)      │
└─────────┬───────────┘           └───────────┬─────────────┘
          │                                   │
          ▼                                   ▼
┌─────────────────────────────────────────────────────────────┐
│                   StatementHandler                         │
│          (JDBC Statement 操作封装)                          │
└────────────┬──────────────────────────┬────────────────────┘
             │                          │
             ▼                          ▼
┌─────────────────────┐           ┌─────────────────────────┐
│  ParameterHandler   │           │   ResultSetHandler      │
│    (参数处理)       │           │     (结果处理)          │
└─────────────────────┘           └─────────────────────────┘
```

### Executor 执行器

MyBatis 有三种执行器：

| 执行器 | 说明 | 特点 |
|--------|------|------|
| **SimpleExecutor** | 普通执行器 | 每次执行创建新的 PreparedStatement |
| **ReuseExecutor** | 重用执行器 | 重复使用已预编译的 Statement |
| **BatchExecutor** | 批量执行器 | 批量执行所有更新语句 |

**配置方式：**

```xml
<!-- mybatis-config.xml -->
<settings>
    <setting name="defaultExecutorType" value="SIMPLE"/>
</settings>
```

```java
// Spring Boot 配置
mybatis.configuration.default-executor-type=SIMPLE
```

```java
// Spring Boot 指定特定 Mapper 使用批处理
@Configuration
public class MyBatisConfig {
    @Bean
    public SqlSessionTemplate sqlSessionTemplateBatch(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory, ExecutorType.BATCH);
    }
}
```

> **注意：** BatchExecutor 在事务未提交前无法获取自增 ID。

### 预编译语句重用

```java
// ReuseExecutor 效果示例
// 第一次执行
PreparedStatement ps = connection.prepareStatement("SELECT * FROM user WHERE id = ?");
ps.setInt(1, 1);
ps.executeQuery();

// 第二次执行相同 SQL（复用）
// 不再创建新的 PreparedStatement，直接重用
ps = connection.prepareStatement("SELECT * FROM user WHERE id = ?");
ps.setInt(1, 2);
ps.executeQuery();
```

---

## MyBatis 缓存

### 一级缓存

一级缓存是 SqlSession 级别的缓存，默认开启。

```java
// 同一个 SqlSession 中，两次查询相同 ID 的用户
User user1 = userMapper.selectById(1L);  // 查询数据库
User user2 = userMapper.selectById(1L);  // 从一级缓存获取

// 不同的 SqlSession
SqlSession session1 = sqlSessionFactory.openSession();
SqlSession session2 = sqlSessionFactory.openSession();

UserMapper mapper1 = session1.getMapper(UserMapper.class);
UserMapper mapper2 = session2.getMapper(UserMapper.class);

User user1 = mapper1.selectById(1L);  // 查询数据库
User user2 = mapper2.selectById(1L);  // 再次查询数据库
```

**特点：**
- 生命周期与 SqlSession 一致
- 范围是 SqlSession 内部
- 多 SqlSession 并发操作可能产生脏数据

**失效场景：**
- SqlSession 关闭或 clear
- 执行了 update/delete/insert 操作
- 手动调用 `sqlSession.clearCache()`

### 二级缓存

二级缓存是 namespace 级别的缓存，需要手动开启。

```xml
<!-- 方式1：XML 配置 -->
<mapper namespace="com.example.mapper.UserMapper">
    <cache/>

    <!-- 或者配置缓存参数 -->
    <cache
        eviction="FIFO"
        flushInterval="60000"
        size="512"
        readOnly="true"/>
</mapper>
```

```java
// 方式2：注解方式
@CacheNamespace(eviction = FIFO.class, flushInterval = 60000, size = 512)
@Mapper
public interface UserMapper { }
```

**查询流程：**

```
二级缓存 → 一级缓存 → 数据库
```

**问题：**
- 多表查询时可能产生脏数据
- 分布式环境下需要使用集中式缓存（Redis）

### Redis 二级缓存

```xml
<!-- pom.xml 添加依赖 -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-redis</artifactId>
    <version>1.0.0-beta2</version>
</dependency>
```

```xml
<!-- Redis 配置 -->
<cache type="org.mybatis.caches.redis.RedisCache">
    <property name="host" value="localhost"/>
    <property name="port" value="6379"/>
    <property name="password" value=""/>
</cache>
```

---

## Spring Boot 集成

### 快速入门

```java
// 1. 添加依赖
// pom.xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

```java
// 2. Mapper 接口
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM t_user WHERE id = #{id}")
    User findById(Long id);

    @Insert("INSERT INTO t_user(name, age) VALUES(#{name}, #{age})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(User user);
}
```

```java
// 3. 启动类配置
@SpringBootApplication
@MapperScan("com.example.mapper")  // 扫描 Mapper 接口
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### SqlSessionTemplate

在 Spring Boot 中，SqlSession 的实现是 SqlSessionTemplate，它是线程安全的。

```java
// 源码解析
public class SqlSessionTemplate implements SqlSession {
    private final SqlSessionProxy proxy;  // 动态代理

    // 内部类 SqlSessionInterceptor 实现自动管理 SqlSession
    private class SqlSessionInterceptor implements InvocationHandler {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            SqlSession sqlSession = getSqlSession(
                SqlSessionTemplate.this.sqlSessionFactory,
                SqlSessionTemplate.this.executorType,
                SqlSessionTemplate.this.exceptionTranslator);

            try {
                Object result = method.invoke(sqlSession, args);
                if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
                    sqlSession.commit(true);  // 提交
                }
                return result;
            } finally {
                if (sqlSession != null) {
                    closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
                }
            }
        }
    }
}
```

### 事务管理

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    @Transactional(rollbackFor = Exception.class)
    public void saveUser(User user) {
        userMapper.insert(user);
        // 其他操作
    }

    // 只读事务（性能优化）
    @Transactional(readOnly = true)
    public List<User> findAll() {
        return userMapper.selectAll();
    }
}
```

---

## MyBatis-Plus

MyBatis-Plus（简称 MP）是 MyBatis 的增强工具，简化开发。

### 快速入门

```java
// 1. 添加依赖
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.5</version>
</dependency>
```

```java
// 2. 实体类
@Data
@TableName("t_user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;

    private String name;
    private Integer age;
    private String email;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

### 快速入门

MyBatis-Plus 提供两套方案：

**方案一：只使用 BaseMapper（推荐简单场景）**

```java
// Mapper 层：继承 BaseMapper 获得基础 CRUD
public interface UserMapper extends BaseMapper<User> {
    // BaseMapper 已提供：insert, deleteById, updateById, selectById, selectList 等方法
}
```

```java
// Service 层直接调用
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public User getUserById(Long id) {
        return userMapper.selectById(id);  // 基础 CRUD
    }

    public List<User> searchUsers(String name) {
        return userMapper.selectList(new QueryWrapper<User>().like("name", name));
    }
}
```

**方案二：IService + ServiceImpl（推荐复杂业务场景）**

```java
// Mapper 层：同样需要继承 BaseMapper
public interface UserMapper extends BaseMapper<User> {
}
```

```java
// Service 层：继承 ServiceImpl，获得更丰富的业务方法
@Service
public class UserService extends ServiceImpl<UserMapper, User> {

    // IService 提供的增强方法
    public User getUserById(Long id) {
        return getById(id);  // 继承自 IService
    }

    public boolean saveUser(User user) {
        return save(user);  // 继承自 IService，包含自动填充等逻辑
    }

    // 条件构造器查询
    public List<User> searchUsers(String name, Integer age) {
        return list(new QueryWrapper<User>()
            .like("name", name)
            .eq("age", age)
            .orderByDesc("create_time"));
    }

    // 链式调用（更推荐）
    public List<User> searchUsersChain(String name) {
        return list(new QueryWrapper<User>().like("name", name));
    }
}
```

> **说明：**
> - BaseMapper 是 MyBatis-Plus 的核心，提供基础的增删改查方法
> - IService 是对 BaseMapper 的进一步封装，在 Service 层使用更方便
> - 两者都需要 Mapper 继承 BaseMapper，但 Service 层可以根据需要选择是否使用 IService
> - IService 的 save/update 方法会自动填充 `@TableField(fill = FieldFill.INSERT)` 等注解的字段

### 常用注解

| 注解 | 说明 |
|------|------|
| @TableName | 表名 |
| @TableId | 主键 |
| @TableField | 字段映射 |
| @TableLogic | 逻辑删除 |

### 分页查询

```java
// 1. 配置分页插件
@Configuration
public class MyBatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

```java
// 2. 分页查询
IPage<User> page = new Page<>(1, 10);
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.gt("age", 18);
IPage<User> result = userMapper.selectPage(page, wrapper);

System.out.println("总记录数: " + result.getTotal());
System.out.println("数据: " + result.getRecords());
```

### 条件构造器

```java
// 等于 =
new QueryWrapper<User>().eq("age", 20);

// 不等于 <>
new QueryWrapper<User>().ne("age", 20);

// 大于 >
new QueryWrapper<User>().gt("age", 18);

// 小于 <
new QueryWrapper<User>().lt("age", 60);

// LIKE
new QueryWrapper<User>().like("name", "张");

// IN
new QueryWrapper<User>().in("id", Arrays.asList(1, 2, 3));

// 排序
new QueryWrapper<User>().orderByDesc("create_time").orderByAsc("age");

// 链式调用
new QueryWrapper<User>()
    .like("name", "张")
    .gt("age", 18)
    .orderByDesc("create_time")
    .last("LIMIT 10");
```

---

## MyBatis 与 JPA/Hibernate 对比

### 核心区别

| 特性 | JPA/Hibernate | MyBatis |
|------|---------------|----------|
| ORM 程度 | 全自动化 | 半自动 |
| SQL 控制 | 自动生成 | 手动编写 |
| 学习曲线 | 较陡 | 较平缓 |
| 性能调优 | 困难 | 灵活 |
| 侵入性 | 高（需加注解） | 低（无需改 POJO） |

### 适用场景

**JPA/Hibernate 适用于：**
- 需求变化不多的中小型项目
- 快速开发，原型迭代
- 业务逻辑为主，数据模型稳定
- 团队熟悉 ORM 概念

**MyBatis 适用于：**
- 需求变化较多的项目
- 复杂查询场景
- 需要精细 SQL 优化
- 已有数据库 Schema
- 需要高度控制 SQL

### 代码对比

**MyBatis 方式：**

```java
// Mapper
@Mapper
public interface UserMapper {
    User selectByName(String name);
}

// XML
<select id="selectByName" resultType="User">
    SELECT * FROM user WHERE name = #{name}
</select>
```

**JPA 方式：**

```java
// 实体
@Entity
public class User {
    @Id
    private Long id;
    private String name;
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByName(String name);
}
```

> **总结：** JPA 对 POJO 侵入性强，需要添加注解；MyBatis 需要手动写 SQL 但更灵活。

---

## SQL 注入防护

### #{} vs ${}

- **#{}** - 预编译参数，防止 SQL 注入
- **${}** - 字符串拼接，存在 SQL 注入风险

```xml
<!-- 安全：使用 #{} -->
<select id="selectByName" resultType="User">
    SELECT * FROM user WHERE name = #{name}
</select>

<!-- 危险：使用 ${} -->
<select id="selectByName" resultType="User">
    SELECT * FROM user WHERE name = '${name}'
</select>
```

### Like 查询

```xml
<!-- 方式1：程序拼接 -->
<select id="search" resultType="User">
    SELECT * FROM user WHERE name LIKE #{name}
</select>
// Java: userMapper.search("%张%");

<!-- 方式2：函数拼接（推荐） -->
<select id="search" resultType="User">
    SELECT * FROM user WHERE name LIKE CONCAT('%', #{name}, '%')
</select>
// Java: userMapper.search("张");
```

### In 查询

```xml
<select id="selectByIds" resultType="User">
    SELECT * FROM user
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

---

## 常见问题与 Tips

### Tips

**1. 设置单次查询超时时间**

```xml
<select id="selectUser" timeout="10">
    SELECT * FROM user WHERE id = #{id}
</select>
```

**2. Mapper 方法重载问题**

MyBatis 使用 `package+Mapper+method` 全限名作为 key 查找 SQL，**不支持方法重载**。

```java
// 错误：无法区分
User selectById(Long id);
User selectById(Integer id);  // 编译通过但运行异常
```

**3. 返回主键**

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user (name, age) VALUES (#{name}, #{age})
</insert>
```

### FAQ

**Q: MyBatis 无法注入 Mapper？**

A: 检查是否配置了 `@MapperScan`：
```java
@SpringBootApplication
@MapperScan("com.example.mapper")  // 扫描路径
public class Application { }
```

**Q: BindingException: Invalid bound statement？**

A: 可能原因：
1. XML 文件位置不对
2. 未正确配置 `mapper-locations`
3. 未在 pom.xml 中配置 xml 打包

```xml
<!-- pom.xml -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

**Q: 一级缓存没有生效？**

A: 检查是否每次都创建新的 SqlSession，确保在同一个事务内。

**Q: 如何打印 SQL 日志？**

```properties
logging.level.com.example.mapper=DEBUG
logging.level.org.mybatis=DEBUG
```

**Q: 懒加载失效？**

A: MyBatis 本身不提供懒加载（与 JPA/Hibernate 不同），如需懒加载可使用 `mybatis-association` 等插件。

**Q: 多数据源配置？**

```java
@Bean
@Primary
@ConfigurationProperties("spring.datasource.primary")
public DataSource primaryDataSource() {
    return DataSourceBuilder.create().build();
}

@Bean
@ConfigurationProperties("spring.datasource.secondary")
public DataSource secondaryDataSource() {
    return DataSourceBuilder.create().build();
}
```

然后为不同数据源创建不同的 SqlSessionFactory 和 MapperScan。