# JPA

## 概述

### JPA是什么

JPA（Java Persistence API）是Java官方制定的ORM（Object-Relational Mapping）**标准规范**，而非具体实现。它定义了一套用于管理Java对象与关系型数据库之间映射的API和元数据。

JPA的出现是为了统一ORM框架的实现标准。在JPA出现之前，各个ORM框架（如Hibernate、TopLink）都有自己独立的API，开发者一旦选择某个框架，就与该框架紧密耦合。JPA规范的出现使得开发者可以基于标准编程，当需要更换ORM实现时，只需修改配置文件，无需修改业务代码。

**JPA主要实现框架：**

- **Hibernate** - 最流行的JPA实现，功能最强大，性能优异
- **EclipseLink** - Eclipse社区的JPA实现，原生支持JPA所有特性
- **OpenJPA** - Apache基金会的JPA实现，已停止维护
- **DataNucleus** - 纯JPA实现，支持多种数据存储

> **注意：** Hibernate是目前功能最完善、性能最好的JPA实现，也是Spring Data JPA的默认实现。

### JPA版本演进

| 版本 | 发布年份 | 主要特性 |
|------|----------|----------|
| JPA 1.0 | 2006 | 基础实体映射、EntityManager、JPQL |
| JPA 2.0 | 2009 | 关系映射增强、Criteria API、缓存 |
| JPA 2.1 | 2013 | 转换器（Converters）、存储过程、属性 |
| JPA 2.2 | 2017 | Java 8日期时间API支持 |
| JPA 3.0 | 2019 | Java 8+特性、Schema生成增强 |
| JPA 3.1 | 2022 | 实体Graph增强、统计函数 |

### 相关故事

自从Hibernate框架被JBoss公司收购之后，作者Gavin King也加入了JBoss公司，不久就开始与JBoss管理者的理念产生了分歧。最后还是离开了JBoss公司，加入到了Sun公司。在Sun公司期间，Gavin King将ORM的理念作为标准发布，这个标准就是JPA。

当JPA出现后，基于JPA标准的ORM框架如雨后春笋般大量出现。其中比较出名的有：Apache基金会的OpenJPA，Eclipse社区的EclipseLink。标准化ORM的出现，直接威胁到了原来的ORM框架Hibernate。因此Hibernate框架不得不应战，于是也开始了对JPA标准的支持。

由于Hibernate的历史原因，Hibernate对JPA标准的支持有两套模式：

1. **兼容模式** - 操作的接口使用Hibernate原来制定的API，只有映射的注解使用JPA标准
2. **完全模式** - 操作的接口和映射的注解全部使用JPA的标准

> 建议在实际开发中优先使用JPA标准接口，以保证代码的可移植性。

### Hibernate使用JPA规范解决了什么问题

**使用注解来替代配置文件。**

注解的作用是代替配置文件，将程序的元数据写在代码上。元数据是程序必须依赖的数据，如实体类与数据库表的映射关系。

配置文件（如XML）不是编程语言的语法，无法进行断点调试。而注解是Java语法的一部分，报错时可以快速定位问题，并且IDE能提供更好的代码补全和类型检查。

---

## 核心概念与术语

### ORM

ORM（Object-Relational Mapping）是一种将Java对象与数据库表进行映射的技术。开发者可以通过操作Java对象来间接操作数据库，无需编写繁琐的SQL语句。

**ORM的核心思想：**
- 表（Table）→ 类（Class）
- 字段（Column）→ 属性（Field）
- 记录（Row）→ 对象（Object）

### EntityManager

EntityManager是JPA的核心API，负责实体的增删改查操作。它是应用程序与持久化存储之间的桥梁。

```java
EntityManager em = entityManagerFactory.createEntityManager();
```

### Persistence Context

Persistence Context（持久化上下文）是EntityManager管理实体的内存区域。当实体处于持久化上下文中时，任何对该实体的修改都会自动同步到数据库。

### EntityTransaction

EntityTransaction用于管理单个实体管理器的事务，确保数据的一致性。

```java
EntityTransaction tx = em.getTransaction();
tx.begin();
// 执行操作
tx.commit();
```

---

## JPA执行流程

### 传统JPA编程方式

```java
// 1. 创建EntityManagerFactory（通常Application级别只创建一次）
EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPU");

// 2. 从工厂创建EntityManager
EntityManager em = emf.createEntityManager();

// 3. 获取事务并开启
EntityTransaction tx = em.getTransaction();
tx.begin();

// 4. 执行CRUD操作
User user = new User();
user.setName("张三");
user.setAge(25);
em.persist(user);  // 持久化

User found = em.find(User.class, 1L);  // 查询

user.setAge(26);  // 修改（自动脏检查）
em.remove(found);  // 删除

// 5. 提交事务
tx.commit();

// 6. 关闭资源
em.close();
emf.close();
```

**执行流程图：**

```
┌─────────────────────────────────────────────────────────────┐
│                     Persistence.xml                         │
│                  (或 Spring Boot 自动配置)                   │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                 EntityManagerFactory                        │
│              (线程安全，通常全局唯一)                         │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  EntityManager                              │
│              (非线程安全，每次请求创建)                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│  Persistence     │    │    Database     │
│  Context        │    │                 │
│  (一级缓存)      │    │                 │
└─────────────────┘    └─────────────────┘
```

---

## 初始化机制

### Persistence.xml配置

传统JPA需要在`META-INF/persistence.xml`中配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
             http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2">

    <persistence-unit name="myPU" transaction-type="RESOURCE_LOCAL">
        <!-- JPA实现提供者 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <!-- 实体类 -->
        <class>com.example.entity.User</class>

        <!-- 配置属性 -->
        <properties>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/test"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="123456"/>

            <!-- Hibernate特定配置 -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL8Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

### Spring Boot配置

在Spring Boot中，JPA配置更加简洁：

```properties
# 数据源配置
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate配置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.open-in-view=true
```

**常用配置项说明：**

| 配置项 | 说明 | 可选值 |
|--------|------|--------|
| spring.jpa.hibernate.ddl-auto | 自动建表策略 | none, validate, update, create, create-drop |
| spring.jpa.show-sql | 显示SQL语句 | true, false |
| spring.jpa.properties.hibernate.format_sql | 格式化SQL | true, false |
| spring.jpa.open-in-view | 开启Open Session in View | true, false |

**ddl-auto选项说明：**

- `none` - 不执行任何操作
- `validate` - 验证实体类与表结构是否匹配
- `update` - 自动更新表结构（推荐开发环境）
- `create` - 每次启动删除表并重新创建
- `create-drop` - 同create，但关闭时删除表

---

## 实体类注解详解

### 实体声明

```java
@Entity
@Table(name = "t_user")
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private String name;

    @Column
    private Integer age;

    @Enumerated(EnumType.STRING)
    private Gender gender;
}
```

**常用注解：**

| 注解 | 说明 |
|------|------|
| @Entity | 声明此类为JPA实体 |
| @Table | 指定实体对应的数据库表 |
| @MappedSuperclass | 声明为父类，不映射为独立表 |

### 主键生成策略

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;  // MySQL自增

@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
private Long id;  // 序列

@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "id_generator")
@TableGenerator(name = "id_generator", table = "id_generator", pkColumnName = "gen_name", valueColumnName = "gen_value")
private Long id;  // 表生成器

@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;  // 由JPA实现自动选择
```

**策略对比：**

| 策略 | 适用数据库 | 优点 | 缺点 |
|------|------------|------|------|
| IDENTITY | MySQL | 简单，无需额外查询 | 无法批量插入 |
| SEQUENCE | Oracle/PostgreSQL | 性能好，支持批量 | 需要创建序列 |
| TABLE | 通用 | 兼容性好 | 性能稍差 |
| AUTO | 通用 | 无需配置 | 不可控 |

### 字段映射

```java
@Column(name = "user_name", nullable = false, length = 50, unique = true)
private String name;

@Transient  // 不映射到数据库
private String temporary;

@Temporal(TemporalType.DATE)
private Date birthday;  // 只存日期

@Temporal(TemporalType.TIMESTAMP)
private Date createTime;  // 存日期和时间

@Enumerated(EnumType.STRING)
private Gender gender;  // 枚举存储为字符串

@Lob
private String content;  // 大文本

@Basic(fetch = FetchType.LAZY)
private byte[] avatar;  // 懒加载
```

### 关系映射

**一对多（OneToMany）：**

```java
@OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@OrderBy("id DESC")
private List<Order> orders;
```

**多对一（ManyToOne）：**

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;
```

**多对多（ManyToMany）：**

```java
@ManyToMany
@JoinTable(
    name = "user_role",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "role_id")
)
private Set<Role> roles;
```

**一对一（OneToOne）：**

```java
@OneToOne
@JoinColumn(name = "profile_id")
private UserProfile profile;
```

### 级联操作

| 级联类型 | 说明 |
|----------|------|
| CascadeType.PERSIST | 级联持久化（保存） |
| CascadeType.MERGE | 级联合并（更新） |
| CascadeType.REMOVE | 级联删除 |
| CascadeType.REFRESH | 级联刷新 |
| CascadeType.DETACH | 级联脱管 |
| CascadeType.ALL | 包含以上所有 |

### 审计字段

```java
@Version  // 乐观锁
private Long version;

@CreationTimestamp  // 创建时间
private LocalDateTime createTime;

@UpdateTimestamp  // 更新时间
private LocalDateTime updateTime;

@CreatedBy  // 创建人
private String createBy;

@LastModifiedBy  // 最后修改人
private String lastModifiedBy;
```

### 继承映射策略

```java
// 单表策略（默认）
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "type")
public abstract class Animal { }

// joined策略 - 每个子类一张表
@Inheritance(strategy = InheritanceType.JOINED)

// table-per-class策略 - 每个实体一张表
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
```

---

## 实体生命周期与状态

### 四种状态

**1. 瞬时状态（Transient）**

数据库中没有数据与之对应。瞬时状态的实体就是一个普通的Java对象，和持久化上下文无关联。

```java
User user = new User();  // 瞬时状态
user.setName("张三");
```

**2. 托管状态（Persistent）**

数据库中有数据与之对应，并且处于持久化上下文中。对实体的任何修改都会自动同步到数据库。

```java
User user = em.find(User.class, 1L);  // 托管状态
user.setName("李四");  // 自动脏检查，提交时同步到数据库
```

**3. 游离状态（Detached）**

数据库中有数据与之对应，但当前没有EntityManager与之关联。事务提交后，托管对象就转变为游离状态。

```java
em.close();  // 关闭后，user变为游离状态
user.setName("王五");  // 修改不会同步到数据库
```

**4. 删除状态（Removed）**

当调用EntityManager的remove()方法后，实体对象处于删除状态。事务提交时从数据库删除。

```java
User user = em.find(User.class, 1L);
em.remove(user);  // 删除状态，提交后从数据库删除
```

### 状态转换图

```
                    ┌─────────────────┐
                    │                 │
                    │   new Object()  │
                    │                 │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Transient    │
                    │   (瞬时状态)     │
                    └────────┬────────┘
                             │ persist()
                             ▼
                    ┌─────────────────┐
                    │    Persistent   │
                    │   (托管状态)     │
                    │  ┌───────────┐  │
                    │  │ 一级缓存  │  │
                    │  └───────────┘  │
                    └────────┬────────┘
                             │ close() / clear() / detach()
                             ▼
                    ┌─────────────────┐
                    │    Detached     │
                    │   (游离状态)     │
                    └────────┬────────┘
                             │ merge()
                             ▼
                    ┌─────────────────┐
                    │    Persistent   │
                    │   (托管状态)     │
                    └────────┬────────┘
                             │ remove()
                             ▼
                    ┌─────────────────┐
                    │    Removed      │
                    │   (删除状态)     │
                    └─────────────────┘
```

### flush机制

JPA会自动将内存中的更改同步到数据库：

1. **事务提交时** - 自动执行flush
2. **查询执行前** - 自动flush（如查询前有修改）
3. **手动调用** - `em.flush()`

---

## EntityManager API详解

### 查找操作

```java
// 立即加载
User user1 = em.find(User.class, 1L);

// 延迟加载代理（不触发SQL）
User user2 = em.getReference(User.class, 1L);

// 查询单个结果
User user3 = em.createQuery("SELECT u FROM User u WHERE u.name = :name", User.class)
    .setParameter("name", "张三")
    .getSingleResult();
```

### 持久化操作

```java
// persist: 将瞬时对象转为托管状态
User user = new User();
user.setName("张三");
em.persist(user);  // 此时user变为托管状态，flush时插入数据库

// merge: 将游离对象合并到新的托管对象
User detachedUser = em.find(User.class, 1L);
em.close();  // detachedUser变为游离状态

User mergedUser = em.merge(detachedUser);  // 返回新的托管对象
```

> **persist vs merge：**
> - `persist` 只能用于新增，实体变为托管状态
> - `merge` 可用于新增或更新，总是返回托管对象

### 删除操作

```java
User user = em.find(User.class, 1L);
em.remove(user);  // 标记为删除，提交时从数据库删除
```

### 查询操作

```java
// JPQL查询
Query query = em.createQuery("SELECT u FROM User u WHERE u.age > :age");
query.setParameter("age", 18);
List<User> users = query.getResultList();

// 命名查询
@NamedQuery(name = "User.findByName", query = "SELECT u FROM User u WHERE u.name = :name")
List<User> users = em.createNamedQuery("User.findByName", User.class)
    .setParameter("name", "张三")
    .getResultList();

// 原生SQL查询
Query sqlQuery = em.createNativeQuery("SELECT * FROM t_user WHERE name = ?", User.class);
sqlQuery.setParameter(1, "张三");
List<User> users = sqlQuery.getResultList();
```

### 刷新与清除

```java
em.refresh(user);  // 从数据库刷新实体
em.clear();  // 清除所有托管实体
em.contains(user);  // 检查是否在持久化上下文中
```

---

## JPQL查询语言

### 基本查询

```java
// 查询所有
List<User> users = em.createQuery("SELECT u FROM User u", User.class).getResultList();

// 条件查询
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE u.age > :age AND u.name LIKE :name", User.class)
    .setParameter("age", 18)
    .setParameter("name", "%张%")
    .getResultList();

// 排序
List<User> users = em.createQuery(
    "SELECT u FROM User u ORDER BY u.age DESC, u.name ASC", User.class)
    .getResultList();

// 分页
List<User> users = em.createQuery("SELECT u FROM User u", User.class)
    .setFirstResult(0)   // 起始偏移
    .setMaxResults(10)  // 每页数量
    .getResultList();
```

### 关联查询

```java
// JOIN查询
List<Order> orders = em.createQuery(
    "SELECT o FROM Order o JOIN o.user u WHERE u.name = :name", Order.class)
    .setParameter("name", "张三")
    .getResultList();

// LEFT JOIN（包含没有关联的记录）
List<User> users = em.createQuery(
    "SELECT u FROM User u LEFT JOIN u.orders o", User.class)
    .getResultList();

// 多个JOIN
List<Order> orders = em.createQuery(
    "SELECT o FROM Order o JOIN o.user u JOIN o.items i WHERE u.name = :name AND i.quantity > 1", Order.class)
    .getResultList();
```

### 聚合函数

```java
// COUNT
long count = em.createQuery("SELECT COUNT(u) FROM User u", Long.class).getSingleResult();

// SUM
double total = em.createQuery("SELECT SUM(u.balance) FROM User u", Double.class).getSingleResult();

// AVG
double avgAge = em.createQuery("SELECT AVG(u.age) FROM User u", Double.class).getSingleResult();

// 分组统计
List<Object[]> results = em.createQuery(
    "SELECT u.department, COUNT(u) FROM User u GROUP BY u.department", Object[].class)
    .getResultList();

// 分组过滤
List<Object[]> results = em.createQuery(
    "SELECT u.department, COUNT(u) FROM User u GROUP BY u.department HAVING COUNT(u) > 5", Object[].class)
    .getResultList();
```

### 命名查询

```java
@Entity
@NamedQueries({
    @NamedQuery(name = "User.findAll", query = "SELECT u FROM User u"),
    @NamedQuery(name = "User.findByName", query = "SELECT u FROM User u WHERE u.name = :name"),
    @NamedQuery(name = "User.findByAgeBetween", query = "SELECT u FROM User u WHERE u.age BETWEEN :min AND :max")
})
public class User { }
```

```java
// 使用命名查询
List<User> users = em.createNamedQuery("User.findByName", User.class)
    .setParameter("name", "张三")
    .getResultList();
```

### 常用JPQL语法

```java
// IN查询
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE u.id IN :ids", User.class)
    .setParameter("ids", Arrays.asList(1L, 2L, 3L))
    .getResultList();

// EXISTS
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE EXISTS (SELECT o FROM Order o WHERE o.user = u)", User.class)
    .getResultList();

// CASE表达式
List<Object[]> results = em.createQuery(
    "SELECT u.name, CASE WHEN u.age < 18 THEN '未成年' WHEN u.age < 60 THEN '成年' ELSE '老年' END FROM User u", Object[].class)
    .getResultList();

// 函数
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE UPPER(u.name) LIKE :name", User.class)
    .setParameter("name", "%ZHANG%")
    .getResultList();
```

---

## Criteria API

Criteria API提供了一种类型安全的查询方式，适合构建动态查询。

### 基本查询

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);

Root<User> user = cq.from(User.class);
cq.select(user).where(cb.equal(user.get("name"), "张三"));

List<User> results = em.createQuery(cq).getResultList();
```

### 动态查询

```java
public List<User> searchUsers(String name, Integer minAge, Integer maxAge) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<User> cq = cb.createQuery(User.class);
    Root<User> user = cq.from(User.class);

    List<Predicate> predicates = new ArrayList<>();

    if (name != null) {
        predicates.add(cb.like(user.get("name"), "%" + name + "%"));
    }
    if (minAge != null) {
        predicates.add(cb.ge(user.get("age"), minAge));
    }
    if (maxAge != null) {
        predicates.add(cb.le(user.get("age"), maxAge));
    }

    cq.where(predicates.toArray(new Predicate[0]));
    cq.orderBy(cb.desc(user.get("createTime")));

    return em.createQuery(cq).getResultList();
}
```

### 聚合查询

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Long> cq = cb.createQuery(Long.class);
Root<User> user = cq.from(User.class);
cq.select(cb.count(user));
Long count = em.createQuery(cq).getSingleResult();
```

### 多表查询

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Order> cq = cb.createQuery(Order.class);
Root<Order> order = cq.from(Order.class);
Join<Order, User> user = order.join("user");

cq.select(order).where(cb.equal(user.get("name"), "张三"));
List<Order> orders = em.createQuery(cq).getResultList();
```

---

## Spring Data JPA

### Repository接口层次

```
Repository (接口)
    │
    ├─ CrudRepository (增删改查)
    │       │
    │       └─ PagingAndSortingRepository (分页排序)
    │               │
    │               └─ JpaRepository (JPA特定)
```

### 快速入门

```java
// 实体类
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Integer age;
    // getters/setters
}

// Repository接口
public interface UserRepository extends JpaRepository<User, Long> {
}
```

### 方法命名规则

Spring Data JPA会根据方法名自动生成SQL：

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // 根据name查询
    Optional<User> findByName(String name);

    // AND条件
    List<User> findByNameAndAge(String name, Integer age);

    // OR条件
    List<User> findByNameOrAge(String name, Integer age);

    // LIKE查询
    List<User> findByNameContaining(String name);

    // 大于/小于
    List<User> findByAgeGreaterThan(Integer age);
    List<User> findByAgeLessThan(Integer age);

    // BETWEEN
    List<User> findByAgeBetween(Integer minAge, Integer maxAge);

    // IN
    List<User> findByIdIn(Collection<Long> ids);

    // 排序
    List<User> findByAgeGreaterThan(Integer age, Sort sort);

    // 分页
    Page<User> findByAge(Integer age, Pageable pageable);
}
```

### @Query注解

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // JPQL查询
    @Query("SELECT u FROM User u WHERE u.name = :name")
    User findByName(@Param("name") String name);

    // 占位符
    @Query("SELECT u FROM User u WHERE u.age > ?1")
    List<User> findByAgeGreaterThan(Integer age);

    // 原生SQL
    @Query(value = "SELECT * FROM t_user WHERE name = :name", nativeQuery = true)
    User findByNameNative(@Param("name") String name);

    // 更新操作
    @Modifying
    @Query("UPDATE User u SET u.age = :age WHERE u.id = :id")
    int updateAgeById(@Param("id") Long id, @Param("age") Integer age);

    // 删除操作
    @Modifying
    @Query("DELETE FROM User u WHERE u.id = :id")
    void deleteByIdCustom(@Param("id") Long id);
}
```

### 分页与排序

```java
// 分页查询
Page<User> page = userRepository.findAll(PageRequest.of(0, 10, Sort.by("age").descending()));

// 带条件分页
Page<User> page = userRepository.findByAgeGreaterThan(18, PageRequest.of(0, 10));

// 获取分页信息
page.getTotalElements();  // 总记录数
page.getTotalPages();    // 总页数
page.getNumber();        // 当前页码
page.getContent();       // 数据列表
```

### Specification复杂查询

```java
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
}
```

```java
// Service层
public List<User> searchUsers(String name, Integer minAge) {
    Specification<User> spec = Specification.where(null);

    if (name != null) {
        spec = spec.and((root, query, cb) ->
            cb.like(root.get("name"), "%" + name + "%"));
    }

    if (minAge != null) {
        spec = spec.and((root, query, cb) ->
            cb.ge(root.get("age"), minAge));
    }

    return userRepository.findAll(spec);
}
```

### 事务配置

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    // 默认事务：读写事务
    @Transactional
    public User saveUser(User user) {
        return userRepository.save(user);
    }

    // 只读事务（性能优化）
    @Transactional(readOnly = true)
    public List<User> findAll() {
        return userRepository.findAll();
    }

    // 指定回滚异常
    @Transactional(rollbackFor = Exception.class)
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

## JPA缓存机制

### 一级缓存（Persistence Context）

一级缓存是EntityManager级别的缓存，默认开启，无需配置。

```java
User user1 = em.find(User.class, 1L);  // 查询数据库
User user2 = em.find(User.class, 1L);  // 从一级缓存获取

em.clear();  // 清除一级缓存
User user3 = em.find(User.class, 1L);  // 再次查询数据库
```

**特点：**
- 生命周期与EntityManager一致
- 同一EntityManager内，相同实体只查询一次
- 事务范围内共享

### 二级缓存（Shared Cache）

二级缓存是EntityManagerFactory级别的缓存，需要手动开启。

```java
// 实体类启用二级缓存
@Entity
@Cacheable(true)
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // ...
}
```

```properties
# 开启二级缓存
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory

# 或使用EhCache
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
```

**缓存策略：**

| 策略 | 说明 |
|------|------|
| NONE | 不使用缓存 |
| READ_ONLY | 只读，修改会抛异常 |
| NONSTRICT_READ_WRITE | 不严格读写 |
| READ_WRITE | 读写，保证一致性 |
| TRANSACTIONAL | 事务级别 |

### 查询缓存

查询缓存缓存的是查询结果而非实体。

```java
@QueryHints({@QueryHint(name = "org.hibernate.cacheable", value = "true")})
@Query("SELECT u FROM User u WHERE u.age > :age")
List<User> findByAgeGreaterThan(@Param("age") Integer age);
```

```properties
# 开启查询缓存
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

### N+1问题

**问题：** 当查询一个实体及其关联对象时，如果关联对象是懒加载，会产生额外的SQL查询。

```java
// 只会触发1次查询User
List<User> users = em.createQuery("SELECT u FROM User u").getResultList();

// 遍历时每个User触发1次查询Orders（N+1问题）
for (User user : users) {
    System.out.println(user.getOrders().size());
}
```

**解决方案：**

```java
// 1. 使用JOIN FETCH
List<User> users = em.createQuery(
    "SELECT u FROM User u LEFT JOIN FETCH u.orders", User.class)
    .getResultList();

// 2. 使用@BatchSize
@OneToMany(mappedBy = "user")
@BatchSize(size = 10)
private List<Order> orders;

// 3. 使用@Fetch(FetchMode.SUBSELECT)
@OneToMany(mappedBy = "user")
@Fetch(FetchMode.SUBSELECT)
private List<Order> orders;
```

---

## Spring Boot集成

### 自动配置原理

Spring Boot通过`HibernateJpaAutoConfiguration`自动配置JPA：

1. 自动配置DataSource
2. 自动创建EntityManagerFactory
3. 自动配置JpaTransactionManager
4. 自动扫描@Entity注解的类

### 常用配置

```properties
# 基础配置
spring.jpa.open-in-view=true  # OSIV模式，懒加载需要
spring.jpa.properties.jakarta.persistence.lock.timeout=3000  # 锁超时

# SQL输出
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# 自动建表
spring.jpa.hibernate.ddl-auto=update

# 数据库方言
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# 批量操作
spring.jpa.properties.hibernate.jdbc.batch_size=20
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
spring.jpa.properties.hibernate.batch_versioned_data=true
```

### 审计功能

```java
// 启用审计
@EntityScan
@EnableJpaAuditing
public class Application {
}

// 实体类使用审计
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    @CreatedDate
    private LocalDateTime createTime;

    @LastModifiedDate
    private LocalDateTime updateTime;

    @CreatedBy
    private String createBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

### 多数据源配置

```java
// 配置多个DataSource
@Configuration
public class DataSourceConfig {

    @Primary
    @Bean
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}

// 为每个数据源创建EntityManager
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.primary.repo",
    entityManagerFactoryRef = "primaryEntityManagerFactory",
    transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryJpaConfig {
    // ...
}
```

---

## JPA与Hibernate

### Hibernate特有注解

```java
// 动态SQL生成
@Entity
@DynamicInsert(true)   // 只插入非空字段
@DynamicUpdate(true)   // 只更新修改的字段
public class User { }

// 派生属性
@Formula("age + 10")
private Integer agePlusTen;

// 复合主键
@EmbeddedId
private UserId id;

// 索引
@Table(indexes = @Index(name = "idx_name", columnList = "name"))
public class User { }
```

### Hibernate Validator

```java
@Entity
public class User {

    @NotBlank(message = "姓名不能为空")
    @Size(min = 2, max = 20, message = "姓名长度必须在2-20之间")
    private String name;

    @Email(message = "邮箱格式不正确")
    private String email;

    @Min(value = 0, message = "年龄必须大于0")
    @Max(value = 150, message = "年龄必须小于150")
    private Integer age;

    @Past(message = "生日必须是过去的时间")
    private Date birthday;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
}
```

---

## JPA与MyBatis比较

### 核心区别

| 特性 | JPA/Hibernate | MyBatis |
|------|---------------|---------|
| ORM程度 | 全自动化 | 半自动 |
| SQL控制 | 自动生成 | 手动编写 |
| 学习曲线 | 较陡 | 较平缓 |
| 性能调优 | 困难 | 灵活 |
| 移植性 | 好 | 一般 |

### 适用场景

**JPA/Hibernate适用于：**
- 需求变化不多的中小型项目
- 快速开发，原型迭代
- 团队熟悉ORM概念
- 业务逻辑为主，数据模型稳定

**MyBatis适用于：**
- 需求变化较多的项目
- 复杂查询场景
- 需要精细SQL优化
- 已有数据库 schema

### 代码对比

**JPA方式：**

```java
// 实体
@Entity
public class User {
    @Id
    private Long id;
    private String name;
}

// 查询
userRepository.findByName("张三");
```

**MyBatis方式：**

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

> **总结：** JPA对POJO侵入性强，需要添加注解；MyBatis需要手动写SQL但更灵活。选择时应根据项目需求和团队技术栈决定。

---

## Tips与FAQ

### Tips

**1. 打印SQL日志**

```properties
# 方式1：显示SQL
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# 方式2：使用SQL日志
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

**2. 乐观锁与悲观锁**

```java
// 乐观锁 - 使用@Version
@Version
private Long version;

// 悲观锁 - 使用LockMode
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT u FROM User u WHERE u.id = :id")
User findByIdWithLock(@Param("id") Long id);
```

**3. 实体类设计最佳实践**

- 使用包装类型而非基本类型（Integer vs int）
- 合理设置fetch类型，避免N+1问题
- 使用逻辑删除替代物理删除
- 避免循环引用

### FAQ

**Q: JPA无法自动建表？**

A: 检查配置：
```properties
spring.jpa.hibernate.ddl-auto=update
```
确认实体类在包扫描范围内。

**Q: EntityManager线程安全问题？**

A: EntityManager是非线程安全的，在Spring中应使用`@PersistenceContext`注入，每次请求自动获得新实例。

**Q: 循环引用导致StackOverflow？**

A: 在toString()、equals()中避免调用关联对象，或使用`@JsonIgnore`/`@JsonManagedReference`。

**Q: 实体类必须有主键？**

A: JPA要求每个实体必须有@Id标注的主键字段。

**Q: 如何处理复合主键？**

A: 使用@EmbeddedId或@IdClass。

**Q: 事务不生效的原因？**

A: 检查是否在Service层添加@Transactional，Spring Boot默认不开启事务代理。

**Q: 懒加载失败（LazyInitializationException）？**

A: 原因是在Session关闭后访问懒加载属性。解决方案：
- 开启Open Session in View
- 使用JOIN FETCH
- 在事务内完成数据访问
