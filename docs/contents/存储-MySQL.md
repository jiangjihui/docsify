# MySQL

> MySQL 是最流行的关系型数据库管理系统之一，本文档涵盖 MySQL 的安装配置、基本操作、数据类型、SQL 语法、索引、存储引擎、主从复制、备份恢复及性能优化等内容。

---

## 安装配置

### Linux 安装

#### CentOS 7 安装 MySQL 5.7

```sh
# 下载 MySQL  yum 源
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

# 安装 MySQL  yum 源
yum -y install mysql57-community-release-el7-10.noarch.rpm

# 安装 MySQL 服务器
yum -y install mysql-community-server
```

#### CentOS/RHEL 安装 MySQL 8.0

在服务器上创建文件夹：

```sh
mkdir /mysql
```

下载 `mysql-8.0.25-1.el7.x86_64.rpm-bundle.tar`，使用 `tar` 指令进行解压：

```sh
tar -xf mysql-8.0.18-1.el7.x86_64.rpm-bundle.tar
```

安装前先卸载 mariadb：

```sh
yum remove mariadb*
```

按顺序安装：

```sh
rpm -ivh mysql-community-common-8.0.18-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.18-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.18-1.el7.x86_64.rpm
```

初始化数据库：

```sh
cd /etc
mysqld --initialize --user=mysql
```

查看初始密码：

```sh
cat /var/log/mysqld.log
```

登录并修改密码：

```sql
mysql -u root -p
alter user 'root'@'localhost' identified by '123456';
```

#### Linux 安装 MySQL 5.7（通用方式）

```sh
# 添加 mysql 用户组和用户
groupadd mysql
useradd -r -g mysql mysql

# 解压安装包
tar -xzvf /data/software/mysql-5.7.13-linux-glibc2.5-x86_64.tar.gz

# 重命名
mv mysql-5.7.13-linux-glibc2.5-x86_64 mysql

# 授权
chown -R mysql:mysql ./

# 初始化（记录生成的临时密码）
bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

# 配置 SSL
bin/mysql_ssl_rsa_setup --datadir=/usr/local/mysql/data

# 复制配置文件
cp support-files/my-default.cnf /etc/my.cnf
cp support-files/mysql.server /etc/init.d/mysql

# 修改启动脚本中的路径
vim /etc/init.d/mysql
# 修改 basedir 和 datadir

# 启动 MySQL
bin/mysqld_safe --user=mysql &

# 登录并设置密码
mysql -u root -p
set password=password('A123456');

# 授权远程访问
grant all privileges on *.* to root@'%' identified by 'A123456';
flush privileges;
```

### Docker 安装

```sh
# 拉取 MySQL 镜像
docker pull mysql:8.0

# 运行 MySQL 容器
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  -p 3306:3306 \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

# 进入容器
docker exec -it mysql mysql -uroot -proot123
```

### 配置优化

#### 常用配置参数

```properties
# Buffer Pool 大小（建议设置为可用内存的 80%）
innodb_buffer_pool_size=2048M

# 重做日志文件大小
innodb_log_file_size=256M

# 最大连接数
max_connections=200

# 查询缓存（MySQL 8.0 已移除）
# query_cache_size=64M
# query_cache_type=1

# 字符集
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# 日志配置
slow_query_log=1
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2
```

#### 查看配置

```sql
-- 查看数据目录
show global variables like "%datadir%";

-- 查看存储引擎
show variables like '%storage_engine%';

-- 查看字符集
show variables like 'character%';
```

---

## 基本操作

### 启动与停止

#### Linux 系统服务方式

```sh
# 启动
sudo service mysqld start
sudo /etc/init.d/mysqld start

# 停止
sudo service mysqld stop
sudo /etc/init.d/mysqld stop

# 重启
sudo service mysqld restart
sudo /etc/init.d/mysqld restart

# 查看状态
sudo service mysqld status
```

#### Debian/Ubuntu 系统

```sh
# 启动
service mysql start

# 停止
service mysql stop

# 重启
service mysql restart

# 查看状态
service mysql status
```

#### Docker 方式

```sh
# 启动容器
docker start mysql

# 停止容器
docker stop mysql

# 重启容器
docker restart mysql
```

### 用户管理

#### 创建用户

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';

-- 示例
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'admin'@'%' IDENTIFIED BY 'admin123';
```

参数说明：
- `username`：用户名
- `host`：指定该用户在哪个主机上可以登陆
  - `localhost`：本地用户
  - `%`：任意远程主机
  - `192.168.1.%`：指定网段
- `password`：密码（可为空，为空则无需密码登录）

#### 设置与修改密码

```sql
-- 修改指定用户密码
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');

-- 修改当前用户密码
SET PASSWORD = PASSWORD('newpassword');

-- 示例
SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
```

#### 授权

```sql
-- 授予权限
GRANT privileges ON databasename.tablename TO 'username'@'host';

-- 示例
GRANT SELECT, INSERT ON testdb.* TO 'user1'@'localhost';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;
```

常用权限：`SELECT`、`INSERT`、`UPDATE`、`DELETE`、`CREATE`、`DROP`、`ALTER`、`ALL PRIVILEGES`

#### 撤销权限

```sql
REVOKE privilege ON databasename.tablename FROM 'username'@'host';

-- 示例
REVOKE INSERT ON testdb.* FROM 'user1'@'localhost';
```

#### 删除用户

```sql
DROP USER 'username'@'host';

-- 示例
DROP USER 'dog'@'localhost';
```

#### 查看用户

```sql
-- 查看所有用户
SELECT user, host FROM mysql.user;

-- 查看用户权限
SHOW GRANTS FOR 'username'@'host';
```

### 数据库管理

#### 创建数据库

```sql
-- 创建数据库（指定字符集）
CREATE DATABASE database_name CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 示例
CREATE DATABASE mydb CHARACTER SET utf8mb4;
```

#### 查看数据库

```sql
-- 查看所有数据库
SHOW DATABASES;

-- 查看数据库创建信息
SHOW CREATE DATABASE database_name;
```

#### 删除数据库

```sql
DROP DATABASE database_name;
```

#### 选择数据库

```sql
USE database_name;
```

#### 表管理

```sql
-- 查看所有表
SHOW TABLES;

-- 查看表结构
DESC table_name;
SHOW CREATE TABLE table_name;
```

---

## 数据类型

### 数值类型

#### 整数类型

| 类型 | 存储空间 | 有符号范围 | 无符号范围 |
|------|----------|------------|------------|
| TINYINT | 1字节 | -128 ~ 127 | 0 ~ 255 |
| SMALLINT | 2字节 | -32768 ~ 32767 | 0 ~ 65535 |
| MEDIUMINT | 3字节 | -8388608 ~ 8388607 | 0 ~ 16777215 |
| INT | 4字节 | -2147483648 ~ 2147483647 | 0 ~ 4294967295 |
| BIGINT | 8字节 | -9223372036854775808 ~ 9223372036854775807 | 0 ~ 18446744073709551615 |

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    age TINYINT UNSIGNED,
    score INT DEFAULT 0
);
```

#### 浮点数类型

| 类型 | 存储空间 | 精度 |
|------|----------|------|
| FLOAT | 4字节 | 单精度浮点数，约7位小数 |
| DOUBLE | 8字节 | 双精度浮点数，约15位小数 |
| DECIMAL | 变长 | 精确数值，用于财务计算 |

```sql
-- DECIMAL 用于精确计算
CREATE TABLE account (
    id INT PRIMARY KEY AUTO_INCREMENT,
    balance DECIMAL(10, 2)  -- 总共10位，其中2位小数
);
```

### 字符串类型

| 类型 | 最大长度 | 特点 |
|------|----------|------|
| CHAR | 255字符 | 固定长度，不足部分用空格填充 |
| VARCHAR | 65535字节 | 变长存储，需要1-2字节存储长度 |
| TEXT | 64KB | 长文本 |
| MEDIUMTEXT | 16MB | 中等长度文本 |
| LONGTEXT | 4GB | 长文本 |

```sql
-- CHAR vs VARCHAR 对比
CREATE TABLE char_test (
    fixed_char CHAR(10),      -- 固定分配10个字符
    variable_char VARCHAR(10) -- 根据实际内容分配
);

-- TEXT 类型
CREATE TABLE article (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200),
    content TEXT,
    summary MEDIUMTEXT
);
```

> **注意**：MySQL 5.0+ 后，VARCHAR 按字符计算长度，而非字节。

### 日期时间类型

| 类型 | 格式 | 范围 |
|------|------|------|
| DATE | 'YYYY-MM-DD' | '1000-01-01' ~ '9999-12-31' |
| TIME | 'HH:MM:SS' | '-838:59:59' ~ '838:59:59' |
| DATETIME | 'YYYY-MM-DD HH:MM:SS' | '1000-01-01 00:00:00' ~ '9999-12-31 23:59:59' |
| TIMESTAMP | 'YYYY-MM-DD HH:MM:SS' | '1970-01-01 00:00:01' ~ '2038-01-19 03:14:07' |
| YEAR | 'YYYY' | 1901 ~ 2155 |

```sql
CREATE TABLE event (
    id INT PRIMARY KEY AUTO_INCREMENT,
    event_date DATE,
    event_time TIME,
    event_datetime DATETIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### JSON 类型

MySQL 5.7+ 支持原生的 JSON 数据类型，可以高效地存储和查询 JSON 文档。

```sql
-- JSON 类型示例
CREATE TABLE config (
    id INT PRIMARY KEY AUTO_INCREMENT,
    settings JSON,
    metadata JSON
);

-- 插入 JSON 数据
INSERT INTO config (settings) VALUES
('{"theme": "dark", "language": "zh-CN", "notifications": true}');

-- 查询 JSON 字段
SELECT settings->>'$.theme' AS theme FROM config;

-- 使用 JSON_EXTRACT 函数
SELECT JSON_EXTRACT(settings, '$.language') FROM config;
```

#### JSON 类型存储结构

MySQL 将 JSON 转为二进制格式（doc 对象）存储，包含：
- **type**（1字节）：表示 JSON 值的类型
- **value**：实际数据

类型包括：object、array、literal、number、string 等。

> **注意**：
> - JSON 对象键按长度和 code point 排序
> - 单个 JSON 文档大小不能超过 4G
> - 单个 key 不能超过 64KB

---

## SQL 操作

### SELECT 查询

```sql
-- 基本查询
SELECT * FROM users;

-- 条件查询
SELECT * FROM users WHERE age > 18 AND status = 'active';

-- 排序
SELECT * FROM users ORDER BY created_at DESC;

-- 分页
SELECT * FROM users LIMIT 10 OFFSET 20;

-- 聚合函数
SELECT COUNT(*) FROM users;
SELECT AVG(age) FROM users;
SELECT SUM(score) FROM users;
SELECT MIN(age), MAX(age) FROM users;

-- 分组
SELECT status, COUNT(*) FROM users GROUP BY status;

-- 分组过滤
SELECT status, COUNT(*) FROM users GROUP BY status HAVING COUNT(*) > 10;

-- 多表连接
SELECT u.name, o.order_no, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- 子查询
SELECT * FROM users WHERE age > (SELECT AVG(age) FROM users);

-- DISTINCT 去重
SELECT DISTINCT status FROM users;
```

### INSERT 插入

```sql
-- 单条插入
INSERT INTO users (name, age, email) VALUES ('张三', 25, 'zhangsan@example.com');

-- 批量插入
INSERT INTO users (name, age, email) VALUES
('李四', 30, 'lisi@example.com'),
('王五', 28, 'wangwu@example.com');

-- 插入查询结果
INSERT INTO backup_users (name, age)
SELECT name, age FROM users WHERE status = 'inactive';
```

### UPDATE 更新

```sql
-- 基本更新
UPDATE users SET age = 26 WHERE id = 1;

-- 多个字段更新
UPDATE users SET age = 27, status = 'active' WHERE id = 2;

-- 条件更新
UPDATE users SET status = 'inactive' WHERE created_at < '2023-01-01';
```

### DELETE 删除

```sql
-- 删除指定记录
DELETE FROM users WHERE id = 1;

-- 条件删除
DELETE FROM users WHERE status = 'inactive' AND created_at < '2023-01-01';

-- 清空表（谨慎使用）
DELETE FROM users;  -- 可以回滚
TRUNCATE TABLE users;  -- 不可回滚，自增计数器重置
```

### 常用函数

#### 字符串函数

```sql
SELECT CONCAT('Hello', ' ', 'World');  -- 连接字符串
SELECT UPPER('hello');  -- 转大写
SELECT LOWER('HELLO');  -- 转小写
SELECT SUBSTRING('Hello World', 1, 5);  -- 截取字符串
SELECT TRIM('  hello  ');  -- 去除首尾空格
SELECT LENGTH('你好');  -- 字节长度
SELECT CHAR_LENGTH('你好');  -- 字符长度
```

#### 数值函数

```sql
SELECT ABS(-10);  -- 绝对值
SELECT CEIL(3.14);  -- 向上取整
SELECT FLOOR(3.14);  -- 向下取整
SELECT ROUND(3.14159, 2);  -- 四舍五入
SELECT MOD(10, 3);  -- 取模
```

#### 日期函数

```sql
SELECT NOW();  -- 当前日期时间
SELECT CURDATE();  -- 当前日期
SELECT CURTIME();  -- 当前时间
SELECT DATE('2024-01-15 10:30:00');  -- 提取日期
SELECT YEAR('2024-01-15');  -- 提取年份
SELECT MONTH('2024-01-15');  -- 提取月份
SELECT DAY('2024-01-15');  -- 提取日期
SELECT DATE_ADD('2024-01-15', INTERVAL 1 DAY);  -- 日期加减
SELECT DATEDIFF('2024-01-15', '2024-01-01');  -- 日期差
```

#### 条件函数

```sql
SELECT IF(age > 18, '成年', '未成年') FROM users;
SELECT IFNULL(NULL, '默认值');  -- NULL值处理

-- CASE 表达式
SELECT CASE status
    WHEN 'active' THEN '激活'
    WHEN 'inactive' THEN '未激活'
    ELSE '未知'
END FROM users;
```

---

## 索引

### 索引类型

#### 主键索引 (Primary Key)

- 唯一且非空
- 每表只能有一个主键索引
- 聚集索引，数据物理存储顺序与索引顺序一致

```sql
-- 创建主键索引
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- 或在表创建后添加
ALTER TABLE user ADD PRIMARY KEY (id);
```

#### 唯一索引 (Unique Index)

- 唯一但允许为空
- 可有多个唯一索引

```sql
-- 创建唯一索引
CREATE UNIQUE INDEX idx_email ON user(email);

-- 或在表创建时指定
CREATE TABLE user (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
```

#### 普通索引 (Normal Index)

- 最基本的索引类型
- 允许重复值

```sql
-- 创建普通索引
CREATE INDEX idx_name ON user(name);

-- 复合索引
CREATE INDEX idx_name_age ON user(name, age);
```

#### 全文索引 (Fulltext Index)

- 用于全文搜索
- 仅支持 InnoDB 和 MyISAM 引擎
- 针对文本内容进行分词搜索

```sql
-- 创建全文索引
CREATE FULLTEXT INDEX idx_content ON article(content);

-- 使用 MATCH AGAINST 查询
SELECT * FROM article WHERE MATCH(content) AGAINST('数据库' IN NATURAL LANGUAGE MODE);
```

### 索引优化

#### 复合索引最左前缀原则

复合索引遵循最左前缀原则，即索引左侧的列可以使用索引，但跳过左边的列则无法使用。

```sql
-- 创建复合索引 (name, age, status)
CREATE INDEX idx_user ON user(name, age, status);

-- 有效的查询
SELECT * FROM user WHERE name = '张三';           -- 使用索引
SELECT * FROM user WHERE name = '张三' AND age = 25;  -- 使用索引
SELECT * FROM user WHERE name = '张三' AND age = 25 AND status = 'active';  -- 使用索引

-- 无效的查询（不使用索引）
SELECT * FROM user WHERE age = 25;                -- 不使用索引
SELECT * FROM user WHERE status = 'active';       -- 不使用索引
SELECT * FROM user WHERE name = '张三' AND status = 'active';  -- 只使用 name 部分
```

#### 索引下推 (Index Condition Pushdown)

索引下推是 MySQL 5.6+ 的优化特性，将部分 WHERE 条件下推到索引层面进行过滤，减少回表次数。

```sql
-- 假设有索引 (name, age)
SELECT * FROM user WHERE name = '张三' AND age > 18;

-- 优化前：先根据 name 找到所有记录，再在服务器层过滤 age
-- 优化后：在索引层面直接过滤 age，只回表符合条件的数据
```

#### 索引失效的场景

- 使用函数或运算：`WHERE YEAR(created_at) = 2024`
- 类型转换：`WHERE phone = 13800138000`（phone 是 VARCHAR 类型）
- 模糊查询以通配符开头：`WHERE name LIKE '%张'`
- 使用 OR 连接非索引列：`WHERE name = '张三' OR age = 25`
- 字符串不加引号：`WHERE name = 123`

#### 索引长度计算

```
索引长度 = 字段长度 + 是否为空(+1) + 是否变长(+2)
```

---

## 存储引擎

### InnoDB

InnoDB 是 MySQL 默认的事务型存储引擎。

#### 特点

- 支持事务、外键、行锁
- 聚集索引：数据文件与主键索引绑在一起
- 支持 MVCC（多版本并发控制）

#### 锁算法

- **Record Lock**：单行记录锁
- **Gap Lock**：间隙锁，锁定一个范围，但不包括记录本身
- **Next-Key Lock**：Record Lock + Gap Lock 的组合，防止幻读

> **注意**：InnoDB 行锁是通过索引实现的，而非记录本身。只有通过索引条件检索数据时才使用行锁，否则使用表锁。

### MyISAM

#### 特点

- 不支持事务、不支持外键
- 非聚集索引：索引与数据分离
- 表级锁：读读并发，读写串行
- 存储为三个文件：.frm、.MYD、.MYI

#### 适用场景

- 只读数据或小规模数据
- 可以容忍修复操作
- 需要全文搜索

### 存储引擎对比

| 特性 | MyISAM | InnoDB |
|------|--------|--------|
| 事务支持 | 不支持 | 支持 |
| 外键支持 | 不支持 | 支持 |
| 锁粒度 | 表级 | 行级 |
| 索引类型 | 非聚集 | 聚集 |
| 空间碎片 | 产生 | 不产生 |
| 文件格式 | 分离 | 统一 |

---

## MySQL 锁

MySQL 锁机制是保证数据并发访问一致性的重要手段。

### 锁类型

#### 共享锁与排他锁

- **共享锁（S Lock）**：允许事务读取一行数据，多个事务可以同时持有共享锁
- **排他锁（X Lock）**：允许事务删除或更新一行数据，排他锁与其他锁互斥

```sql
-- 共享锁
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE;

-- 排他锁
SELECT * FROM users WHERE id = 1 FOR UPDATE;
```

#### 意向锁

InnoDB 支持多粒度锁，意向锁是表级锁，表示事务将要对表中的行加锁。

- **意向共享锁（IS）**：事务将要获取行的共享锁
- **意向排他锁（IX）**：事务将要获取行的排他锁

```sql
-- 查看意向锁
SHOW ENGINE INNODB STATUS;
```

#### 记录锁（Record Lock）

锁定索引记录一行，而非整个记录。

```sql
-- 当使用唯一索引（等值查询）时，使用记录锁
SELECT * FROM users WHERE id = 1 FOR UPDATE;
```

#### 间隙锁（Gap Lock）

锁定索引记录之间的间隙，防止其他事务插入新记录。

```sql
-- 范围查询时使用间隙锁
SELECT * FROM users WHERE id > 1 AND id < 10 FOR UPDATE;
```

#### 临键锁（Next-Key Lock）

记录锁 + 间隙锁的组合，锁定一个范围且包括记录本身。防止幻读。

```sql
-- 临键锁示例
SELECT * FROM users WHERE id >= 1 AND id <= 10 FOR UPDATE;
```

### 锁等待与死锁

```sql
-- 查看当前锁等待
SELECT * FROM information_schema.INNODB_LOCKS;

-- 查看事务等待状态
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- 查看当前事务
SELECT * FROM information_schema.INNODB_TRX;

-- 死锁超时时间
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';
```

### 解决死锁

1. **设置超时时间**：超过指定时间自动回滚
2. **检测死锁**：InnoDB 自动检测死锁并回滚较小事务
3. **优化业务逻辑**：合理设计索引，减少锁范围

```sql
-- 调整死锁检测
SET GLOBAL innodb_deadlock_detect = 'ON';

-- 调整锁等待超时
SET GLOBAL innodb_lock_wait_timeout = 50;
```

---

## MVCC

MVCC（Multi-Version Concurrency Control，多版本并发控制）是 InnoDB 实现的并发控制机制，通过保存数据的多个版本来实现高并发访问。

### 工作原理

InnoDB 为每行数据添加三个隐藏字段：
- **DB_TRX_ID**：最近修改的事务 ID
- **DB_ROLL_PTR**：回滚指针，指向 undo log
- **DB_ROW_ID**：行 ID（仅当表没有主键时由系统生成）

### 事务隔离级别与 MVCC

| 隔离级别 | 读取方式 | 脏读 | 不可重复读 | 幻读 |
|----------|----------|------|------------|------|
| READ UNCOMMITTED | 读取最新版本 | 是 | 是 | 是 |
| READ COMMITTED | 读取最新提交版本 | 否 | 是 | 是 |
| REPEATABLE READ | 读取事务开始时版本 | 否 | 否 | 是 |
| SERIALIZABLE | 加锁读取 | 否 | 否 | 否 |

```sql
-- 查看当前隔离级别
SELECT @@transaction_isolation;

-- 设置隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### Read View

Read View 记录活跃事务列表，用于判断行的可见性：

- **m_ids**：活跃事务 ID 列表
- **min_trx_id**：最小活跃事务 ID
- **max_trx_id**：创建 Read View 时最大事务 ID + 1
- **creator_trx_id**：创建该 Read View 的事务 ID

**可见性判断规则**：
1. 行的 trx_id < min_trx_id：说明事务已提交，可见
2. 行的 trx_id >= max_trx_id：说明事务在 Read View 创建后开始，不可见
3. 行的 trx_id 在 m_ids 中：说明事务未提交，不可见
4. 行的 trx_id 不在 m_ids 中：说明事务已提交，可见

### 快照读与当前读

- **快照读**：SELECT 语句不加锁，通过 MVCC 读取历史版本
- **当前读**：SELECT ... FOR UPDATE、INSERT、UPDATE、DELETE 语句，加锁读取最新数据

```sql
-- 快照读（不加锁）
SELECT * FROM users WHERE id = 1;

-- 当前读（加锁）
SELECT * FROM users WHERE id = 1 FOR UPDATE;
UPDATE users SET name = '张三' WHERE id = 1;
```

### 总结

MVCC 优势：
- 读写不冲突，提高并发性能
- 减少锁的使用，降低死锁概率
- 保证事务的隔离性

MVCC 局限：
- 需要额外的存储空间存储 undo log
- 清理旧版本需要后台进程处理

---

## 主从复制

MySQL 主从复制是一种数据同步机制，将主服务器的数据复制到从服务器。

### 应用场景

- **读写分离**：将读操作分配到从数据库，减轻主数据库负载
- **数据备份**：从数据库作为主数据库的备份
- **数据分析**：在从数据库上进行数据分析，避免影响主库

### 工作原理

1. **主服务器记录 Binlog**：执行事务或 SQL 时记录二进制日志
2. **从服务器拉取并应用**：从服务器 IO 线程拉取 Binlog，SQL 线程应用日志

### 配置主从复制

#### 主服务器配置

```sh
# my.cnf 配置
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin
binlog_format = ROW
sync_binlog = 1
```

```sql
-- 创建复制用户
CREATE USER 'repl'@'%' IDENTIFIED BY 'repl_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- 查看主服务器状态
SHOW MASTER STATUS;
```

#### 从服务器配置

```sh
# my.cnf 配置
[mysqld]
server-id = 2
relay_log = /var/log/mysql/mysql-relay-bin
read_only = 1
```

```sql
-- 配置复制
CHANGE MASTER TO
    MASTER_HOST = 'master_host',
    MASTER_USER = 'repl',
    MASTER_PASSWORD = 'repl_password',
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 123;

-- 启动复制
START SLAVE;

-- 查看复制状态
SHOW SLAVE STATUS\G;
```

### 半同步复制

确保至少一个从服务器收到并持久化了 Binlog 后，主服务器才返回成功。

```sql
-- 主服务器安装半同步插件
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';

-- 从服务器安装半同步插件
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- 启用半同步
SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_slave_enabled = 1;
```

### 组复制 (MGR)

MySQL 5.7.17+ 引入的高可用方案，基于 Paxos 协议实现多主复制。

```sh
# my.cnf 配置
[mysqld]
server-id = 1
gtid_mode = ON
enforce_gtid_consistency = ON
group_replication_group_name = "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
group_replication_start_on_boot = OFF
group_replication_local_address = "127.0.0.1:33061"
group_replication_group_seeds = "127.0.0.1:33061,127.0.0.1:33062,127.0.0.1:33063"
group_replication_bootstrap_group = ON
```

```sql
-- 创建复制用户
CREATE USER 'rpl_user'@'%' IDENTIFIED BY 'rpl_password';
GRANT BACKUP_ADMIN, GROUP_REPLICATION_STREAM, REPLICATION SLAVE ON *.* TO 'rpl_user'@'%';

-- 启动组复制
START GROUP_REPLICATION;
```

---

## 备份与恢复

### mysqldump 备份

```sh
# 备份单个数据库
mysqldump -u root -p mydb > mydb.sql

# 备份所有数据库
mysqldump -u root -p --all-databases > all_databases.sql

# 备份指定表
mysqldump -u root -p mydb users orders > tables.sql

# 备份结构（不含数据）
mysqldump -u root -p -d mydb > mydb_structure.sql

# 备份数据（不含结构）
mysqldump -u root -p -t mydb > mydb_data.sql

# 备份远程数据库
mysqldump -h remote_host -u root -p mydb > mydb.sql
```

#### 高级选项

```sh
# 导出事务保持一致性（适合备份）
mysqldump -u root -p --single-transaction --master-data=2 mydb > mydb.sql

# 压缩备份
mysqldump -u root -p mydb | gzip > mydb.sql.gz

# 分库备份
mysqldump -u root -p --databases db1 db2 db3 > databases.sql
```

### mysqlbinlog 恢复

```sh
# 基于时间点恢复
mysqlbinlog --stop-datetime="2024-01-15 10:00:00" mysql-bin.000001 | mysql -u root -p

# 基于位置恢复
mysqlbinlog --stop-position=1234 mysql-bin.000001 | mysql -u root -p

# 导出指定时间段的 Binlog
mysqlbinlog --start-datetime="2024-01-15 09:00:00" --stop-datetime="2024-01-15 10:00:00" mysql-bin.000001 > binlog.sql
```

### xtrabackup 备份

Percona Xtrabackup 是高性能的 InnoDB 备份工具，支持热备份。

```sh
# 完全备份
xtrabackup --backup --target-dir=/backup/full --user=root --password=root

# 增量备份
xtrabackup --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/full --user=root --password=root

# 准备备份（恢复前准备）
xtrabackup --prepare --target-dir=/backup/full

# 恢复
xtrabackup --copy-back --target-dir=/backup/full --user=root --password=root
```

### 数据恢复

```sql
-- 恢复整个数据库
mysql -u root -p mydb < mydb.sql

-- 恢复压缩的备份
gunzip < mydb.sql.gz | mysql -u root -p mydb

-- 恢复指定表
mysql -u root -p mydb -e "SOURCE table_backup.sql"
```

---

## 性能优化

### EXPLAIN 分析

使用 EXPLAIN 分析查询执行计划，找出性能瓶颈。

```sql
-- 基本分析
EXPLAIN SELECT * FROM users WHERE id = 1;

-- 详细分析
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE age > 18;

-- 分析连接
EXPLAIN SELECT u.name, o.order_no
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

EXPLAIN 关键字段说明：

| 字段 | 说明 |
|------|------|
| type | 连接类型，从最优到最差：system > const > eq_ref > ref > range > index > ALL |
| key | 实际使用的索引 |
| rows | 预计扫描的行数 |
| Extra | 额外信息，如 Using filesort, Using temporary |

### 慢查询日志

启用慢查询日志找出执行慢的 SQL。

```sql
-- 查看慢查询配置
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW VARIABLES LIKE 'long_query_time';

-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 设置阈值为1秒

-- 查看慢查询数量
SHOW GLOBAL STATUS LIKE 'Slow_queries';
```

### SQL 优化技巧

1. **避免 SELECT ***：只查询需要的字段

```sql
-- 不推荐
SELECT * FROM users;

-- 推荐
SELECT id, name, email FROM users;
```

2. **使用 LIMIT 分页**：大量数据分页查询

```sql
-- 传统 OFFSET 方式（页数大时性能差）
SELECT * FROM users LIMIT 1000000, 10;

-- 性能更好的方式（基于主键）
SELECT * FROM users WHERE id > 1000000 LIMIT 10;
```

3. **批量操作**：减少数据库交互次数

```sql
-- 不推荐（循环插入）
FOR item IN items:
    INSERT INTO users (name) VALUES (item);

-- 推荐（批量插入）
INSERT INTO users (name) VALUES ('a'), ('b'), ('c');
```

4. **使用连接代替子查询**

```sql
-- 子查询（性能较差）
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);

-- 连接查询（性能更好）
SELECT u.* FROM users u INNER JOIN orders o ON u.id = o.user_id;
```

### 压测工具 mysqlslap

MySQL 自带的压力测试工具。

```sh
# 自动生成 SQL 测试
mysqlslap --auto-generate-sql -uroot -proot

# 并发测试
mysqlslap --auto-generate-sql --concurrency=100 -uroot -proot

# 多轮测试
mysqlslap --auto-generate-sql --concurrency=150 --iterations=10 -uroot -proot

# 存储引擎测试
mysqlslap --auto-generate-sql --concurrency=150 --iterations=3 --engine=innodb -uroot -proot
```

---

## MySQL 8.0 新特性

### 窗口函数

窗口函数允许在保留原表结构的同时，对数据进行分组排序和聚合计算。

```sql
-- 排名函数
SELECT name, department, salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as row_number
FROM employees;

-- 累计分布
SELECT name, salary,
    SUM(salary) OVER (ORDER BY salary) as cumulative_sum,
    AVG(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_avg
FROM employees;
```

### CTE (Common Table Expression)

公用表表达式类似于临时视图，可以简化复杂查询。

```sql
-- 基本 CTE
WITH temp AS (
    SELECT * FROM users WHERE age > 18
)
SELECT * FROM temp WHERE status = 'active';

-- 递归 CTE（生成序列）
WITH RECURSIVE cte AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM cte WHERE n < 10
)
SELECT * FROM cte;
```

### 角色管理

MySQL 8.0 引入了基于角色的访问控制（RBAC）。

```sql
-- 创建角色
CREATE ROLE 'app_read', 'app_write', 'app_admin';

-- 授予权限
GRANT SELECT ON mydb.* TO 'app_read';
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_write';
GRANT ALL ON mydb.* TO 'app_admin';

-- 创建用户并授予角色
CREATE USER 'user1'@'localhost' IDENTIFIED BY 'password';
GRANT 'app_read' TO 'user1'@'localhost';

-- 用户激活角色
SET DEFAULT ROLE 'app_read' TO 'user1'@'localhost';
```

### 不可见索引

不可见索引允许在不删除索引的情况下测试索引对性能的影响。

```sql
-- 创建不可见索引
CREATE INDEX idx_name ON users(name) INVISIBLE;

-- 修改索引可见性
ALTER TABLE users ALTER INDEX idx_name VISIBLE;
ALTER TABLE users ALTER INDEX idx_name INVISIBLE;
```

### 降序索引

MySQL 8.0 支持创建降序索引，优化 ORDER BY DESC 查询。

```sql
-- 创建降序索引
CREATE INDEX idx_created_at ON posts(created_at DESC);
```

### 查询缓存移除

> **注意**：MySQL 8.0 已移除查询缓存功能。如果需要缓存功能，建议使用应用层缓存（Redis、Memcached）或 ProxySQL 等中间件。

---

## MySQL 常见问题

### ID 自增问题

问：一表有 ID 自增主键，insert 17 条后删除 15-17 条，重启 MySQL，再 insert 一条，ID 是 18 还是 15？

答：
- **MyISAM 表**：ID 是 18（自增主键最大 ID 记录在数据文件，重启不丢失）
- **InnoDB 表**：ID 是 15（自增主键最大 ID 只记录在内存，重启或 OPTIMIZE 后丢失）

### Buffer Pool

Buffer Pool 是 InnoDB 的内存缓存机制，缓存数据页和索引页。

**作用**：
- 读操作：数据首次读取时放入 Buffer Pool，后续直接从内存读取
- 写操作：修改先保存在 Buffer Pool，定期刷新到磁盘

### 双写缓冲区 (Doublewrite Buffer)

解决部分写入问题，确保数据页的完整性。

**工作原理**：
1. 刷新数据页时，先复制到双写缓冲区
2. 双写缓冲区刷盘完成后，再写入数据文件
3. 如果刷盘过程中断，可从双写缓冲区恢复

---

> 更多内容请参考 [MySQL 官方文档](https://dev.mysql.com/doc/)
