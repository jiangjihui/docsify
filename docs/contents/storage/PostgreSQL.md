# PostgreSQL

## 历史

PostgreSQL（简称PG）是一个功能强大的开源对象关系数据库管理系统（ORDBMS），其历史可以追溯到1970年代后期至1980年代初期的INGRES项目，该项目由Michael Stonebraker和Eugene Wong等人发起，起源于IBM System R的一系列文档。以下是PostgreSQL的详细发展历程：

#### 一 早期发展（1980年代）

- **POSTGRES项目启动**：1986年，由加州大学伯克利分校的Michael Stonebraker教授领导，POSTGRES项目开始实施。该项目旨在克服当时关系数据库系统的限制，特别是在复杂数据结构支持和扩展性方面。POSTGRES项目得到了国防高级研究计划局（DARPA）、陆军研究办公室（ARO）、美国国家科学基金会（NSF）和ESL, Inc.的赞助。
- **早期版本发布**：POSTGRES经历了多个版本的迭代，包括1987年的第一个“演示软件”系统，以及1989年至1993年期间的多个版本，这些版本主要关注可移植性和可靠性。

#### 二 Postgres95与开源之路

- **Postgres95发布**：1994年，伯克利大学的学生Andrew Yu和Jolly Chen在POSTGRES中添加了一个SQL语言解释器，并将其重命名为Postgres95。Postgres95是原始POSTGRES Berkeley代码的开源后代，它完全是ANSI C代码，性能得到了显著提升。
- **开源版本发布**：1996年，Postgres95更名为PostgreSQL，并发布了第一个开源版本。这一名称的选择旨在反映原始POSTGRES与具有SQL功能的最新版本之间的关系。同时，版本编号从6.0开始，以延续Berkeley POSTGRES项目的序列。

#### 三 持续发展与功能增强

- **版本迭代**：自1996年以来，PostgreSQL经历了多个版本的发布，每个版本都引入了新的功能和改进。例如，2000年引入了外键约束、PL/pgSQL（过程化语言）和复杂查询等功能；2005年改进了性能和可用性，引入了模板数据库、窗口函数、共享行级锁和表空间等功能。
  - 从 PostgreSQL 7.0 版本开始，它进行了重大的架构调整，增强了对事务处理和并发控制的能力。这使得它在企业级应用场景中的可靠性得到了很大提升。例如，在高并发的电商交易系统中，能够更好地处理多个用户同时进行订单提交、支付等操作。
  - PostgreSQL 8.0 引入了窗口函数，这对于数据分析和复杂报表生成等任务提供了强大的支持。通过窗口函数，可以在不使用复杂子查询的情况下，对查询结果进行分组内的排序、聚合等操作。
  - 近年来，PostgreSQL 一直保持活跃的开发状态，不断添加新的功能，如对 JSON 和 JSONB 数据类型的支持，使其能够更好地适应现代数据处理的需求，包括在物联网（IoT）和大数据应用中处理半结构化数据。它在全球范围内的应用越来越广泛，在数据库市场中占据了重要的地位。
- **功能丰富**：随着时间的推移，PostgreSQL逐渐拥有了一系列高级功能，如高级事务性、复杂查询计划、可靠的MVCC（多版本并发控制）、GIS地理信息存储等。这些特性使得PostgreSQL在企业级应用中愈发受欢迎。

## 简介

Pg 不仅仅是 SQL 数据库它可以存储 array 和 json, 可以在 array 和 json 上建索引, 甚至还能用表达式索引. 为了实现文档数据库的功能, 设计了 jsonb 的存储结构. 有人会说为什么不用 Mongodb 的 BSON 呢? Pg 的开发团队曾经考虑过, 但是他们看到 BSON 把 ["a", "b", "c"] 存成 {0: "a", 1: "b", 2: "c"} 的时候就决定要重新做一个 jsonb 了... 现在 jsonb 的性能已经优于 BSON.

MySQL 的各种 text 字段有不同的限制, 要手动区分 small text, middle text, large text... Pg 没有这个限制, text 能支持各种大小.

它自带全文搜索功能 (不用费劲再装一个 elasticsearch 咯)。

它有地理信息处理扩展 (GIS 扩展不仅限于真实世界, 游戏里的地形什么的也可以), 可以用 Pg 搭寻路服务器和地图服务器。PG 多年来在 GIS 领域处于优势地位，因为它有丰富的几何类型，实际上不止几何类型，PG有大量字典、数组、bitmap 等数据类型，相比之下mysql就差很多，instagram就是因为PG的空间数据库扩展POSTGIS远远强于MYSQL的my spatial而采用PGSQL的。

它可以把 70 种外部数据源 (包括 Mysql, Oracle, CSV, hadoop ...) 当成自己数据库中的表来查询。

## PostgreSQL 与MySQL的区别

1. **开源许可**：
   
   - PostgreSQL 是一个开源数据库系统，使用的是类似于 **BSD** 的许可协议。可以进行二开变成自己的商业版本。
   - MySQL 同样是开源的，但是使用的是 GNU General Public License (**GPL**) 许可证。虽然 MySQL 提供了商业许可证选项，但其核心版本仍然是免费且开放源代码的。

2. **数据类型支持**：
   
   - PostgreSQL 支持复杂的数据类型，如 JSON、XML 和地理空间数据，并且提供了丰富的函数来处理这些数据类型。
   - MySQL 也支持多种数据类型，但在默认情况下对 JSON 和地理空间的支持不如 PostgreSQL 强大。

3. **事务与并发控制**：
   
   - PostgreSQL：原生支持MVCC（多版本并发控制），并且对于事务隔离、并发控制有更强大的功能，包括表锁、行锁、死锁检测等功能，使其更适合高并发场景。
   - MySQL：使用MVCC机制，但仅在InnoDB引擎中支持。事务的支持相对简单，对于复杂并发操作的处理稍显薄弱。

4. **SQL 标准兼容性**：
   
   - PostgreSQL 致力于 SQL 标准的兼容性，并实现了许多 SQL:2008 标准的功能。
   - MySQL 则在一定程度上遵循 SQL 标准，但也有自己独特的语法和功能。

5. **性能和优化**：
   
   - MySQL 通常被认为在**读取密集型**负载下表现良好，并且提供了多种优化工具，如索引、缓存等。
     在读操作和简单查询场景中性能通常较高，特别是在Web应用中使用较多。在不需要复杂事务和数据完整性检查的情况下，MySQL常常是性能优选。
   
   - PostgreSQL 在**写入密集型**负载和**复杂的查询**上表现得更好，尤其是在需要执行复杂 SQL 查询的应用场景中。
     在复杂查询和数据分析场景中，PostgreSQL的性能通常表现优越。其优化器在处理复杂JOIN、子查询等场景时表现更好。

6. **安全性**：
   
   - PostgreSQL 和 MySQL 都有良好的安全记录，但 PostgreSQL 在某些方面可能提供更细粒度的安全控制。
   - MySQL 也有强大的安全功能，特别是在 MySQL 企业版中。

7. **复制与高可用性**：
   
   - PostgreSQL：提供流复制和逻辑复制，支持异步和同步复制，并且一致性较好。社区还提供了成熟的分片解决方案（如Citus）。
   - MySQL：支持多种复制方式，包括主从复制和组复制等。但其在一致性方面可能稍逊于PostgreSQL，主从复制的延迟是常见问题。不过，MySQL也通过MySQL Group Replication、Galera Cluster等工具实现了高可用性和负载均衡。

8. **应用场景**：
   
   - PostgreSQL：更适合需要复杂查询、数据一致性、高度扩展性和高级SQL功能的场景，如金融系统、科学计算、大数据分析等。
   - MySQL：更适合需要快速读写操作、较简单的查询和广泛的社区支持的场景，如Web应用、内容管理系统、电子商务网站等。

## 相关概念

### GIN和BTREE索引（JSONB数据类型）

PostgreSQL对JSONB数据类型的GIN和BTREE索引提供了高效的方式来查询和优化半结构化数据。以下是关于这两种索引的详细解释：

#### 一 GIN索引

1. **概述**：
   
   - GIN（Generalized Inverted Index，通用倒排索引）是一个存储对（key, posting list）集合的索引结构。
   - 其中，“key”是一个键值，而“posting list”是一组出现过“key”的位置。

2. **特点**：
   
   - 适用于索引JSONB字段中的任何值，支持范围查询和部分匹配查询。
   - 提供了丰富的查询操作，如路径表达式和聚合函数。
   - 可扩展性强，允许自定义数据类型和访问方法。

3. **使用场景**：
   
   - 当需要搜索JSONB字段中的多个键或值时，GIN索引可以显著提高查询性能。
   - 适用于物联网（IoT）和大数据应用中，需要对半结构化数据进行复杂查询的场景。

4. **创建示例**：
   
   ```sql
   CREATE INDEX idx_users_data_name ON users USING GIN (data->>'name');
   ```
   
   上述代码创建了一个GIN索引，用于索引`users`表中`data`字段的`name`键。注意，这里使用了`->>`运算符来提取JSON字段中的值。
   如果需要支持搜索任意属性，可以使用以下方式创建GIN索引：
   
   ```sql
   CREATE INDEX idx_users_data_all ON users USING GIN (data jsonb_ops);
   ```
   
   或者使用`jsonb_path_ops`操作符创建，但请注意，它只支持索引`@>`操作符：
   
   ```sql
   CREATE INDEX idx_users_data_path ON users USING GIN (data jsonb_path_ops);
   ```

#### 二 BTREE索引

1. **概述**：
   
   - BTREE（B-Tree）是一种平衡树索引结构，每个节点包含多个键值对和指向子节点的指针。
   - 在PostgreSQL中，BTREE是默认的索引类型，适用于大多数查询场景。

2. **特点**：
   
   - 适用于对JSONB字段中的特定键进行等值查询和范围查询。
   - 索引结构相对简单，查询效率较高。
   - 但对于JSONB这种半结构化数据，BTREE索引可能不如GIN索引灵活。

3. **使用场景**：
   
   - 当需要对JSONB字段中的某个特定键进行频繁查询时，BTREE索引可以提供一个有效的解决方案。
   - 适用于需要对半结构化数据进行简单查询和排序的场景。

4. **创建示例**：
   
   ```sql
   CREATE INDEX idx_users_data_value ON users USING BTREE ((data->>'value'));
   ```
   
   上述代码创建了一个BTREE索引，用于索引`users`表中`data`字段的`value`键的值。同样地，这里使用了`->>`运算符来提取JSON字段中的值。

#### 三 比较与选择

- **灵活性**：GIN索引更灵活，支持对JSONB字段中的任意键和值进行索引和查询。而BTREE索引通常只能对特定键进行索引。
- **查询性能**：对于复杂的查询和范围查询，GIN索引通常比BTREE索引性能更优。但对于简单的等值查询，BTREE索引可能具有更好的性能。
- **存储开销**：GIN索引的存储开销可能比BTREE索引大，因为它需要存储更多的元数据和指针信息。

因此，在选择索引类型时，需要根据具体的查询需求和性能要求进行权衡。对于物联网（IoT）和大数据应用中的半结构化数据查询，GIN索引通常是一个更好的选择。但对于简单的等值查询和排序操作，BTREE索引可能更为合适。

## 基本操作

**登陆PostgreSQL控制台**

```sh
# psql
或
# psql -U postgres
```

**设置用户密码**

```sh
> \password postgres
```

> 备注：设置psotgres用户密码

**创建数据库用户**

```sql
CREATE USER dbuser WITH PASSWORD 'password';
```

**创建数据库**

```sql
CREATE DATABASE exampledb OWNER dbuser;
```

**角色赋权**

```sql
GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;
```

**退出控制台**

```sh
> \q
```

> 备注：也可以直接按ctrl+D

## 命令

- \h：查看SQL命令的解释，比如\h select。
- \?：查看psql命令列表。
- \l：列出所有数据库。
- \c [database_name]：连接其他数据库。
- \d：列出当前数据库的所有表格。
- \d [table_name]：列出某一张表格的结构。
- \du：列出所有用户。
- \e：打开文本编辑器。
- \conninfo：列出当前数据库和连接的信息。

## 数据类型

PostgreSQL 支持丰富的数据类型，比大多数数据库更加灵活。

### 数值类型

| 类型名 | 说明 | 存储空间 |
|--------|------|----------|
| `smallint` | 2字节有符号整数 | -32768 到 32767 |
| `integer` | 4字节有符号整数 | -2147483648 到 2147483647 |
| `bigint` | 8字节有符号整数 | -9223372036854775808 到 9223372036854775807 |
| `real` | 4字节浮点数 | 6位十进制精度 |
| `double precision` | 8字节浮点数 | 15位十进制精度 |
| `decimal` | 用户指定精度 | 变长 |
| `numeric` | 用户指定精度 | 变长 |

```sql
-- 创建包含各种数值类型的表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price NUMERIC(10, 2),  -- 10位数字，2位小数
    quantity INTEGER,
    discount REAL
);
```

### 字符串类型

| 类型名 | 说明 |
|--------|------|
| `varchar(n)` | 变长字符串，最长n个字符 |
| `char(n)` | 定长字符串，不足部分用空格填充 |
| `text` | 变长无限长度字符串 |
| `bpchar` | 定长字符类型 |

```sql
-- PostgreSQL 的 text 类型没有长度限制
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    bio TEXT,
    status CHAR(1) DEFAULT 'A'  -- A=Active, I=Inactive
);
```

### 日期时间类型

| 类型名 | 说明 | 范围 |
|--------|------|------|
| `date` | 日期 | 4713 BC 到 5874897 AD |
| `time` | 时间 | 00:00:00 到 24:00:00 |
| `timestamp` | 日期和时间 | 4713 BC 到 294276 AD |
| `timestamptz` | 带时区的日期和时间 | 4713 BC 到 294276 AD |
| `interval` | 时间间隔 | -178000000 年到 178000000 年 |

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_name VARCHAR(200),
    start_date DATE,
    start_time TIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    duration INTERVAL DEFAULT '1 hour'
);

-- 日期时间运算
SELECT created_at + INTERVAL '1 day' AS tomorrow;
SELECT age(timestamp '2024-01-01');  -- 计算年龄/时间差
SELECT EXTRACT(YEAR FROM created_at);  -- 提取年份
SELECT DATE_TRUNC('hour', created_at);  -- 按小时截断
```

### 布尔类型

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    is_published BOOLEAN DEFAULT FALSE,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- 布尔值可以用多种方式表示
INSERT INTO posts (title, is_published) VALUES
    ('Draft Post', FALSE),
    ('Published Post', TRUE),
    ('Another Draft', 'false'),
    ('Another Published', 'yes');
```

### 数组类型

PostgreSQL 原生支持数组类型，无需额外扩展。

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    skills TEXT[],  -- 字符串数组
    phone_numbers VARCHAR(20)[],  -- 固定长度数组
    scores INTEGER[] DEFAULT '{}'
);

-- 插入数组数据
INSERT INTO employees (name, skills, scores) VALUES
    ('张三', ARRAY['Java', 'Python', 'PostgreSQL'], ARRAY[90, 85, 92]),
    ('李四', '{"Go", "Rust"}', '{80, 75}');

-- 数组查询
SELECT * FROM employees WHERE skills @> ARRAY['Java'];  -- 包含Java
SELECT * FROM employees WHERE skills && ARRAY['Python'];  -- 交集
SELECT unnest(skills) FROM employees;  -- 展开数组
```

### JSON/JSONB 类型

PostgreSQL 支持两种 JSON 数据类型：`json`（存储原始JSON字符串）和 `jsonb`（二进制格式，支持索引）。

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    items JSONB,  -- 推荐使用 jsonb
    metadata JSON
);

-- 插入 JSONB 数据
INSERT INTO orders (customer_name, items) VALUES
    ('张三', '[{"product": "iPhone", "qty": 2, "price": 9999}]'),
    ('李四', '{"shipping": "express", "gift": true}');

-- JSONB 查询
SELECT items->0->>'product' FROM orders;  -- 提取第一个元素的产品名
SELECT items @> '[{"product": "iPhone"}]' FROM orders;  -- 包含查询
SELECT items->>'shipping' FROM orders WHERE items ? 'shipping';  -- 键存在查询
```

### UUID 类型

```sql
-- 需要启用 uuid-ossp 扩展
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id INTEGER,
    token VARCHAR(500),
    expires_at TIMESTAMP
);

-- UUID 生成
SELECT uuid_generate_v1();   -- 基于时间戳
SELECT uuid_generate_v4();    -- 随机 UUID（最常用）
```

### GIS 地理信息类型

需要 PostGIS 扩展支持。

```sql
-- 需要 PostGIS 扩展
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    coordinates GEOMETRY(Point, 4326),  -- WGS84 坐标系统
    area GEOMETRY(Polygon, 4326)
);

-- 插入地理数据
INSERT INTO locations (name, coordinates) VALUES
    ('天安门', ST_SetSRID(ST_MakePoint(116.397, 39.908), 4326)),
    ('故宫', ST_GeomFromText('POINT(116.397 39.908)', 4326));

-- 空间查询 - 查找附近位置
SELECT name FROM locations
WHERE ST_DWithin(
    coordinates,
    ST_SetSRID(ST_MakePoint(116.397, 39.908), 4326),
    1000  -- 1000米范围内
);
```

## 常用 SQL 操作

### SELECT 查询

```sql
-- 基本查询
SELECT * FROM users;

-- 条件查询
SELECT * FROM users WHERE age >= 18 AND status = 'active';

-- 排序
SELECT * FROM products ORDER BY price DESC, name ASC;

-- 分页（PostgreSQL 特有语法）
SELECT * FROM users LIMIT 10 OFFSET 20;

-- 去重
SELECT DISTINCT category FROM products;

-- 别名
SELECT
    u.id AS user_id,
    u.name AS user_name,
    p.title AS post_title
FROM users u
JOIN posts p ON u.id = p.user_id;

-- 聚合函数
SELECT
    category,
    COUNT(*) AS total,
    AVG(price) AS avg_price,
    MAX(price) AS max_price,
    MIN(price) AS min_price
FROM products
GROUP BY category
HAVING COUNT(*) > 5
ORDER BY total DESC;

-- 子查询
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- 窗口函数
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

### INSERT 插入

```sql
-- 单行插入
INSERT INTO users (name, email, age) VALUES ('张三', 'zhangsan@example.com', 25);

-- 多行插入
INSERT INTO users (name, email, age) VALUES
    ('李四', 'lisi@example.com', 30),
    ('王五', 'wangwu@example.com', 28),
    ('赵六', 'zhaoliu@example.com', 35);

-- 从查询结果插入
INSERT INTO users (name, email, age)
SELECT name, email, age FROM temp_users WHERE status = 'active';

-- 插入或更新（Upsert）
INSERT INTO users (id, name, email)
VALUES (1, '张三', 'zhangsan_new@example.com')
ON CONFLICT (id) DO UPDATE
    SET name = EXCLUDED.name,
        email = EXCLUDED.email;

-- 使用 RETURNING
INSERT INTO products (name, price) VALUES ('新产品', 99.99)
    RETURNING id, name, created_at;
```

### UPDATE 更新

```sql
-- 简单更新
UPDATE users SET age = 30 WHERE id = 1;

-- 多个字段更新
UPDATE users SET
    name = '张三',
    email = 'zhangsan@example.com',
    updated_at = CURRENT_TIMESTAMP
WHERE id = 1;

-- 使用子查询
UPDATE products SET price = price * 0.9
WHERE category_id IN (
    SELECT id FROM categories WHERE name = '电子产品'
);

-- 使用 RETURNING
UPDATE users SET status = 'inactive' WHERE last_login < '2024-01-01'
    RETURNING id, name, email;
```

### DELETE 删除

```sql
-- 删除满足条件的行
DELETE FROM users WHERE id = 1;

-- 删除重复数据（保留ID最小的）
DELETE FROM users u1
WHERE EXISTS (
    SELECT 1 FROM users u2
    WHERE u1.email = u2.email AND u1.id > u2.id
);

-- 使用子查询删除
DELETE FROM products
WHERE category_id IN (
    SELECT id FROM categories WHERE is_deleted = TRUE
);

-- 使用 RETURNING
DELETE FROM orders WHERE status = 'cancelled'
    RETURNING id, user_id, total;
```

### 常用函数

```sql
-- 字符串函数
SELECT LENGTH('Hello PostgreSQL');  -- 16
SELECT UPPER('hello');  -- 'HELLO'
SELECT LOWER('HELLO');  -- 'hello'
SELECT TRIM('  hello  ');  -- 'hello'
SELECT SUBSTRING('Hello', 1, 4);  -- 'Hell'
SELECT CONCAT('Hello', ' ', 'World');  -- 'Hello World'
SELECT REPLACE('Hello World', 'World', 'PostgreSQL');  -- 'Hello PostgreSQL'

-- 数值函数
SELECT ROUND(3.14159, 2);  -- 3.14
SELECT CEIL(3.14);  -- 4
SELECT FLOOR(3.14);  -- 3
SELECT ABS(-100);  -- 100
SELECT MOD(10, 3);  -- 1

-- 日期函数
SELECT CURRENT_DATE;  -- 当前日期
SELECT CURRENT_TIMESTAMP;  -- 当前时间戳
SELECT NOW();  -- 当前时间戳
SELECT EXTRACT(YEAR FROM CURRENT_DATE);  -- 当前年份
SELECT DATE_TRUNC('month', CURRENT_TIMESTAMP);  -- 月初
SELECT TO_CHAR(CURRENT_DATE, 'YYYY-MM-DD');  -- 格式化日期
SELECT TO_DATE('20240101', 'YYYYMMDD');  -- 字符串转日期

-- 条件表达式
SELECT
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age < 35 THEN '青年'
        WHEN age < 60 THEN '中年'
        ELSE '老年'
    END AS age_group,
    COALESCE(nickname, username, '匿名用户') AS display_name,
    NULLIF(price, 0) AS price  -- 如果price为0则返回NULL
FROM users;
```

## 索引详解

PostgreSQL 支持多种索引类型，每种类型适用于不同的场景。

### BTREE 索引

最常用的索引类型，默认索引。

```sql
-- 创建 BTREE 索引（默认类型）
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_products_price ON products(price DESC);

-- 多列索引
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- 唯一索引
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- 表达式索引
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
CREATE INDEX idx_products_discounted ON products(price * (1 - discount));

-- 部分索引
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

**适用场景**：
- 等值查询（=、<>）
- 范围查询（<、>、<=、>=、BETWEEN）
- 排序操作（ORDER BY）
- 唯一性约束

### GIN 索引

倒排索引，适用于复杂数据类型。

```sql
-- 数组索引
CREATE INDEX idx_products_tags ON products USING GIN(tags);

-- JSONB 索引
CREATE INDEX idx_orders_items ON orders USING GIN(items jsonb_ops);
CREATE INDEX idx_orders_items_path ON orders USING GIN(items jsonb_path_ops);

-- 全文搜索索引
CREATE INDEX idx_articles_content ON articles USING GIN(to_tsvector('english', content));
```

**适用场景**：
- 数组类型查询（&&、@>、<@）
- JSONB 数据查询
- 全文搜索

### GiST 索引

通用搜索树，适用于几何、地理数据类型。

```sql
-- 地理空间索引
CREATE INDEX idx_locations_geom ON locations USING GiST(coordinates);

-- 范围类型索引
CREATE INDEX idx_reservations_time ON reservations USING GIST(ts_range);

-- 近似搜索
CREATE INDEX idx_documents_content ON documents USING GiST(content);
```

**适用场景**：
- 地理空间数据（PostGIS）
- 范围类型
- 近似搜索
- 模糊匹配

### BRIN 索引

块范围索引，适用于大规模顺序数据。

```sql
-- 时间序列数据索引
CREATE INDEX idx_logs_created_at ON logs USING BRIN(created_at);

-- 按块范围分区
CREATE INDEX idx_sales_date ON sales USING BRIN(sale_date)
    WITH (pages_per_range = 128);
```

**适用场景**：
- 时间序列数据
- 按插入顺序存储的数据
- 大表且查询通常基于范围的数据

### Hash 索引

哈希索引，用于等值查询。

```sql
CREATE INDEX idx_users_phone ON users USING HASH(phone_number);
```

**适用场景**：
- 等值查询
- 不支持范围查询和排序

**注意**：PostgreSQL 16 之前的 Hash 索引不在 WAL 中记录，建议谨慎使用。

### 索引选择建议

| 索引类型 | 适用场景 |
|----------|----------|
| BTREE | 默认选择，适用于大多数场景 |
| GIN | 数组、JSON、全文搜索 |
| GiST | 地理空间、范围、复杂几何 |
| BRIN | 大规模顺序数据、时间序列 |
| Hash | 大数据等值查询（需 WAL） |

## 性能优化

### EXPLAIN 分析

```sql
-- 分析查询计划
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 分析并显示实际执行时间
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE user_id = 100;

-- JSON 格式输出
EXPLAIN (FORMAT JSON) SELECT * FROM products WHERE price > 100;
```

关键指标：
- **Seq Scan**：全表扫描，应尽量避免
- **Index Scan / Index Only Scan**：使用索引
- **Bitmap Scan**：位图扫描
- **Nested Loop**：嵌套循环join
- **Hash Join / Merge Join**：哈希/归并join
- **cost**：估算成本
- **rows**：预计返回行数
- **width**：平均行宽度

### 查询优化技巧

```sql
-- 1. 选择需要的列而非 SELECT *
SELECT id, name, email FROM users;

-- 2. 使用 EXPLAIN 分析慢查询
-- 3. 创建合适的索引
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- 4. 避免在索引列上使用函数
-- 不好
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';
-- 好
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = LOWER('test@example.com');

-- 5. 使用 LIMIT 限制结果集
SELECT * FROM logs ORDER BY created_at DESC LIMIT 100;

-- 6. 批量插入使用 COPY 或 INSERT 多行
INSERT INTO users (name, email) VALUES
    ('用户1', 'user1@example.com'),
    ('用户2', 'user2@example.com'),
    ('用户3', 'user3@example.com');

-- 7. 合理使用 JOIN
-- 优先使用 JOIN 而非子查询
SELECT u.name, o.total
FROM users u
INNER JOIN (SELECT user_id, SUM(total) AS total FROM orders GROUP BY user_id) o
ON u.id = o.user_id;
```

### 连接池配置

PostgreSQL 默认不包含连接池，通常使用外部工具。

**PgBouncer**：
```sh
# 安装
apt install pgbouncer

# 配置 /etc/pgbouncer/pgbouncer.ini
[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb

[pgbouncer]
listen_addr = 127.0.0.1
listen_port = 6432
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

**PgBouncer 连接字符串**：
```sh
# 客户端连接
psql -h 127.0.0.1 -p 6432 -U myuser mydb
```

### 常用配置参数

```sql
-- 查看当前配置
SHOW work_mem;
SHOW shared_buffers;
SHOW effective_cache_size;

-- 临时修改（会话级别）
SET work_mem = '256MB';

-- 建议配置（根据服务器内存调整）
-- postgresql.conf
shared_buffers = 256MB          # 建议为系统内存的25%
effective_cache_size = 768MB     # 建议为系统内存的75%
work_mem = 64MB                  # 每个排序操作使用的内存
maintenance_work_mem = 128MB     # 维护操作使用的内存
```

## 备份与恢复

### pg_dump 备份

```sh
# 备份单个数据库
pg_dump -U postgres -Fc mydb > mydb.dump

# 备份为纯文本格式
pg_dump -U postgres -Fp mydb > mydb.sql

# 备份所有数据库
pg_dumpall -U postgres > all_databases.sql

# 只备份表结构
pg_dump -U postgres -s mydb > mydb_schema.sql

# 只备份数据
pg_dump -U postgres -a mydb > mydb_data.sql

# 备份指定表
pg_dump -U postgres -t users -t orders mydb > tables.dump

# 压缩备份
pg_dump -U postgres -Fc mydb | gzip > mydb.dump.gz
```

### pg_restore 恢复

```sh
# 从压缩备份恢复
pg_restore -U postgres -d mydb mydb.dump

# 恢复并创建新数据库
pg_restore -U postgres -C -d postgres mydb.dump

# 只恢复指定表
pg_restore -U postgres -d mydb -t users mydb.dump

# 恢复时删除已存在的对象
pg_restore -U postgres -d mydb --clean mydb.dump

# 恢复并行度
pg_restore -U postgres -d mydb -j 4 mydb.dump
```

### pg_basebackup 物理备份

```sh
# 基础备份（需要配置 replication）
pg_basebackup -h localhost -U replication -D /backup/base -Ft -z -P

# 使用复制槽
pg_basebackup -h localhost -U replication -D /backup/base -Ft -z -P --slot=backup_slot

# 流复制基础备份
pg_basebackup -h localhost -U replication -D /backup -Xs -Pw
```

### 时间点恢复 (PITR)

```sh
# 1. 配置 WAL 归档
# postgresql.conf
archive_mode = on
archive_command = 'cp %p /archive/%f'
restore_command = 'cp /archive/%f %p'

# 2. 恢复到指定时间点
pg_restore -d mydb --target-time="2024-01-01 12:00:00" mydb.dump
```

## 高可用方案

### 流复制

```sql
-- 主库：创建复制用户
CREATE USER replicator WITH REPLICATION PASSWORD 'repl_password';

-- 主库：配置 pg_hba.conf
host    replication     replicator     10.0.0.0/24         md5

-- 主库：配置 postgresql.conf
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1GB
```

```sh
# 从库：使用 pg_basebackup 创建基础备份
pg_basebackup -h master_host -U replicator -D /var/lib/postgresql/16/main -Pw -Xs -Rs

# 从库：配置 postgresql.conf
hot_standby = on

# 从库：创建 recovery.conf（PostgreSQL 12+ 为 standby.signal）
primary_conninfo = 'host=master_host port=5432 user=replicator password=repl_password'
```

### 逻辑复制

```sql
-- 主库：发布表
CREATE PUBLICATION my_publication FOR TABLE users, orders;

-- 主库：创建订阅用户
CREATE USER subscriber WITH REPLICATION PASSWORD 'sub_password';

-- 从库：创建订阅
CREATE SUBSCRIPTION my_subscription
    CONNECTION 'host=master_host port=5432 dbname=mydb user=subscriber password=sub_password'
    PUBLICATION my_publication;
```

### Patroni 高可用

Patroni 是一个成熟的 PostgreSQL 高可用解决方案。

```yaml
# patroni.yml 配置示例
scope: postgres-cluster
namespace: /service/
name: postgresql0

restapi:
  listen: 0.0.0.0:8008
  connect_address: 10.0.0.1:8008

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 10.0.0.1:5432
  data_dir: /data/postgresql0
  parameters:
    wal_level: replica
    max_wal_senders: 10
    max_replication_slots: 10
    hot_standby: on

consul:
  host: 10.0.0.5:8500
  register_service: true
```

### Citus 分片

Citus 是 PostgreSQL 的分布式扩展，支持水平分片。

```sql
-- 安装 Citus 扩展
CREATE EXTENSION citus;

-- 协调节点配置
SELECT citus_set_coordinator_host(' coordinator_host', 5432);

-- 创建分片表
CREATE TABLE orders (
    order_id BIGSERIAL,
    user_id INT,
    total DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

-- 分布表（按 user_id 分片）
SELECT create_distributed_table('orders', 'user_id');

-- 查询分片信息
SELECT * FROM citus_shards;
SELECT * FROM citus_tables;
```

## 常用扩展

PostgreSQL 的扩展系统是其强大功能的重要组成部分。

### PostGIS

地理空间数据库扩展。

```sql
-- 安装扩展
CREATE EXTENSION postgis;

-- 创建空间表
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    geom GEOMETRY(Point, 4326)
);

-- 空间查询
SELECT name FROM locations
WHERE ST_DWithin(
    geom,
    ST_MakePoint(116.4, 39.9)::geography,
    5000  -- 5公里范围内
);

-- 距离计算
SELECT ST_Distance(
    ST_MakePoint(116.4, 39.9)::geography,
    ST_MakePoint(116.5, 40.0)::geography
);
```

### pgvector

向量数据库扩展，用于 AI 应用。

```sql
-- 安装扩展
CREATE EXTENSION vector;

-- 创建向量表
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536)  -- OpenAI embeddings 维度
);

-- 创建向量索引
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops);

-- 向量相似度搜索
SELECT id, content, 1 - (embedding <=> $1) AS similarity
FROM documents
ORDER BY embedding <=> $1
LIMIT 5;
```

### pg_trgm

模糊匹配和相似度搜索。

```sql
-- 安装扩展
CREATE EXTENSION pg_trgm;

-- 创建索引
CREATE INDEX idx_users_name ON users USING GIN (name gin_trgm_ops);

-- 模糊查询
SELECT * FROM users WHERE name LIKE '%张%';
SELECT * FROM users WHERE name ~ '张.*';

-- 相似度搜索
SELECT name, similarity(name, '张三') AS sim
FROM users
WHERE similarity(name, '张三') > 0.3
ORDER BY sim DESC;
```

### uuid-ossp

UUID 生成扩展。

```sql
-- 安装扩展
CREATE EXTENSION uuid-ossp;

-- 生成 UUID
SELECT uuid_generate_v1();   -- 基于时间戳
SELECT uuid_generate_v3(uuid_ns_url(), 'http://example.com');  -- MD5
SELECT uuid_generate_v4();    -- 随机 UUID（最常用）
SELECT uuid_generate_v5(uuid_ns_url(), 'http://example.com');  -- SHA-1

-- 在表中使用
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id INT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### hstore

键值存储扩展。

```sql
-- 安装扩展
CREATE EXTENSION hstore;

-- 创建表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes HSTORE
);

-- 插入数据
INSERT INTO products (name, attributes) VALUES
    ('iPhone', 'color=>"black",storage=>"256GB",price=>"9999"');

-- 查询
SELECT * FROM products WHERE attributes -> 'color' = 'black';
SELECT * FROM products WHERE attributes ? 'storage';
SELECT * FROM products WHERE attributes @> 'color=>"black"';

-- 更新
UPDATE products SET attributes = attributes || 'color=>"white"'::hstore;
UPDATE products WHERE attributes - 'color' IS NOT NULL;
```

### pg_stat_statements

查询性能统计。

```sql
-- 安装扩展
CREATE EXTENSION pg_stat_statements;

-- 查看查询统计
SELECT
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    rows,
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS hit_ratio
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```
