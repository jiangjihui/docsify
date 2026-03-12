# Oracle 数据库

## 概述

### 什么是 Oracle

Oracle 数据库是 Oracle（甲骨文）公司推出的关系型数据库管理系统（RDBMS），是全球最流行的企业级数据库之一。Oracle 以其高可用性、高性能、强安全性和丰富的企业级功能著称，广泛应用于金融、电信、政府、制造等行业的大型核心系统中。

**Oracle 的核心特点：**
- 高可用性：支持 RAC（Real Application Clusters）集群
- 高性能：支持分区表、并行查询、 Hint 优化
- 强安全性：细粒度访问控制、透明数据加密
- 可扩展性：支持超大并发连接和海量数据存储
- 跨平台：支持 Windows、Linux、Unix 等主流操作系统

### Oracle 版本演进

| 版本 | 发布年份 | 主要特性 |
|------|----------|----------|
| Oracle 8 | 1997 | 对象关系特性、分区表 |
| Oracle 9i | 2001 | 真正应用集群（RAC）、Data Guard |
| Oracle 10g | 2003 | Grid 网格计算、自动管理 |
| Oracle 11g | 2007 | 主动数据库、实时应用测试 |
| Oracle 12c | 2013 | 云架构、PDB（可插拔数据库） |
| Oracle 18c | 2018 | 多模数据库、In-Memory 列式存储 |
| Oracle 19c | 2019 | 长期支持版、零停机迁移 |
| Oracle 21c | 2021 | JSON 数据类型原生支持、Blockchain 表 |
| Oracle 23c | 2023 | JSON-Relational  duality、SQL 域 |

> **说明**：Oracle 12c 引入的 PDB（Pluggable Database，可插拔数据库）架构允许在一个 CDB（Container Database）中创建多个 PDB，大幅提升了资源利用率和运维效率。

### 与 MySQL/PostgreSQL 对比

| 特性 | Oracle | MySQL | PostgreSQL |
|------|--------|-------|------------|
| 许可证 | 商业付费 | GPL 开源 | MIT 开源 |
| 存储过程 | PL/SQL | 存储过程 | PL/pgSQL |
| 并发控制 | 行级锁 + 多版本 | 行级锁 + Undo | MVCC |
| 分区表 | 支持 | 支持 | 支持（有限） |
| 集群 | RAC | MySQL Cluster | Citus |
| 最大单库容量 | 无限 | 256TB | 无限 |
| 适用场景 | 企业级核心系统 | Web 应用 | 复杂查询场景 |

---

## 核心概念

### 实例与数据库

Oracle 中有两个重要概念容易混淆：**实例（Instance）** 和 **数据库（Database）**。

- **实例**：一组内存结构（SGA）+ 后台进程，负责操作数据库
- **数据库**：物理文件（数据文件、控制文件、归档日志等）的集合

在单机环境下，实例与数据库通常是一对一关系；在 RAC 集群中，一个数据库可以对应多个实例。

```sql
-- 查看当前实例名
SELECT instance_name FROM v$instance;

-- 查看数据库名
SELECT name FROM v$database;
```

### 内存结构

Oracle 的内存结构分为两大区域：

#### SGA（System Global Area）

SGA 是 Oracle 最重要的内存区域，所有后台进程和服务器进程共享。

| 组件 | 说明 |
|------|------|
| Buffer Cache | 数据块缓存，存储从磁盘读取的数据块 |
| Shared Pool | 库缓存（SQL 语句）+ 数据字典缓存 |
| Redo Log Buffer | 重做日志缓存，记录数据库变更 |
| Java Pool | Java 代码和数据缓存 |
| Large Pool | 大型内存分配（如 RMAN、并行查询） |

```sql
-- 查看 SGA 配置
SHOW SGA;

-- 查看各组件大小
SELECT * FROM v$sgastat;
```

#### PGA（Program Global Area）

PGA 是服务器进程或后台进程的私有内存区域，每个进程独立拥有。

| 组件 | 说明 |
|------|------|
| Sort Area | 排序操作使用的内存 |
| Hash Area | 哈希连接使用的内存 |
| Bitmap Merge Area | 位图合并使用的内存 |

```sql
-- 查看 PGA 配置
SHOW PARAMETER pga;

-- 查看当前 PGA 使用情况
SELECT * FROM v$pgastat;
```

### 表空间

表空间（Tablespace）是 Oracle 逻辑存储结构的核心概念，可以看作是一个"房间"，数据文件则是"房间里的箱子"。

```
数据库 → 表空间 → 数据文件 → 段（表/索引） → 区（Extent） → 数据块
```

**查看表空间：**

```sql
-- 查看所有表空间
SELECT tablespace_name, status, contents FROM dba_tablespaces;

-- 查看表空间使用情况
SELECT tablespace_name,
       ROUND(bytes / 1024 / 1024, 2) AS size_mb,
       ROUND((bytes - free_bytes) / 1024 / 1024, 2) AS used_mb,
       ROUND(free_bytes / 1024 / 1024, 2) AS free_mb
FROM (SELECT tablespace_name, bytes,
             LEAD(bytes) OVER (ORDER BY tablespace_name) AS free_bytes
      FROM (SELECT tablespace_name, SUM(bytes) AS bytes
            FROM dba_data_files GROUP BY tablespace_name));
```

**创建表空间：**

```sql
-- 创建永久表空间
CREATE TABLESPACE mydata
LOGGING
DATAFILE '/u01/app/oracle/oradata/mydb/mydata01.dbf'
SIZE 100M
AUTOEXTEND ON NEXT 10M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL;

-- 创建临时表空间
CREATE TEMPORARY TABLESPACE mytemp
TEMPFILE '/u01/app/oracle/oradata/mydb/mytemp01.dbf'
SIZE 50M
AUTOEXTEND ON NEXT 10M MAXSIZE UNLIMITED;
```

### 用户与模式

Oracle 中**用户（User）**和**模式（Schema）**通常是一一对应的关系。用户创建后会自动创建一个同名的模式，模式包含该用户拥有的所有对象（表、视图、存储过程等）。

```sql
-- 创建用户
CREATE USER myuser IDENTIFIED BY password
DEFAULT TABLESPACE mydata
TEMPORARY TABLESPACE mytemp;

-- 赋予权限
GRANT CONNECT, RESOURCE TO myuser;
GRANT CREATE ANY DIRECTORY TO myuser;

-- 查看所有用户
SELECT username, created, account_status FROM all_users;

-- 查看用户对象
SELECT object_name, object_type FROM all_objects WHERE owner = 'MYUSER';
```

### 权限管理

Oracle 的权限分为**系统权限**和**对象权限**。

```sql
-- 系统权限
GRANT CREATE SESSION TO myuser;        -- 连接数据库
GRANT CREATE TABLE TO myuser;           -- 创建表
GRANT CREATE VIEW TO myuser;            -- 创建视图
GRANT CREATE PROCEDURE TO myuser;       -- 创建存储过程
GRANT dba TO myuser;                    -- 管理员权限

-- 对象权限
GRANT SELECT ON scott.emp TO myuser;    -- 查询权限
GRANT INSERT, UPDATE ON scott.emp TO myuser;
GRANT ALL ON scott.emp TO myuser;       -- 所有权限

-- 角色
GRANT CONNECT, RESOURCE TO myuser;     -- 常用角色组合
```

---

## 安装与配置

### 下载安装

**官方下载地址：**
- Oracle Database: https://www.oracle.com/database/technologies/downloads/

**Linux 安装参考：**
- Oracle 官方安装指南：https://docs.oracle.com/en/database/oracle/oracle-database/

> **注意**：Oracle 安装过程复杂，生产环境建议使用 Docker 或 Oracle Cloud 等云服务简化部署。

### 环境变量

```bash
# Linux 环境变量配置
export ORACLE_SID=orcl              # 数据库实例名
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin
export NLS_LANG=SIMPLIFIED CHINESE_CHINA.AL32UTF8
```

| 环境变量 | 说明 |
|----------|------|
| ORACLE_SID | 数据库实例标识符 |
| ORACLE_HOME | Oracle 软件安装目录 |
| TNS_ADMIN | TNS 配置文件目录 |
| NLS_LANG | 客户端字符集 |

### 客户端工具

#### SQL*Plus

SQL*Plus 是 Oracle 自带的命令行客户端。

```bash
# 登录方式
sqlplus / as sysdba          # 本地 sys 用户
sqlplus username/password   # 用户名密码登录
sqlplus username/password@host:port/service_name  # 远程登录
```

#### SQL Developer

Oracle 官方提供的图形化数据库工具，支持 Windows、Linux、Mac。

**下载地址：** https://www.oracle.com/database/sqldeveloper/

### 字符集配置

字符集不匹配会导致中文乱码���是最常见的问题之一。

```bash
# Windows 设置中文字符集
set NLS_LANG=SIMPLIFIED CHINESE_CHINA.ZHS16GBK
set NLS_LANG=SIMPLIFIED CHINESE_CHINA.AL32UTF8

# Linux 设置中文字符集
export NLS_LANG=SIMPLIFIED CHINESE_CHINA.AL32UTF8
```

```sql
-- 查看数据库字符集
SELECT parameter, value FROM nls_database_parameters
WHERE parameter = 'NLS_CHARACTERSET';

-- 查看会话字符集
SELECT * FROM nls_session_parameters
WHERE parameter = 'NLS_LANG';
```

---

## 数据库管理

### 启动与关闭

```bash
# 1. 关闭监听器
lsnrctl stop

# 2. 以 sysdba 登录
sqlplus / as sysdba

# 3. 关闭数据库
SQL> shutdown immediate;      -- 正常关闭
SQL> shutdown abort;        -- 强制关闭（紧急情况）

# 4. 启动数据库
SQL> startup;                -- 启动到 OPEN 状态
SQL> startup mount;         -- 启动到 MOUNT 状态（用于维护）
SQL> alter database open;   -- 打开数据库

# 5. 启动监听器
lsnrctl start
```

**关闭选项说明：**

| 选项 | 说明 |
|------|------|
| shutdown normal | 等待所有连接断开后关闭（可能长时间阻塞） |
| shutdown transactional | 等待事务完成后关闭 |
| shutdown immediate | 回滚未提交事务，立即关闭（常用） |
| shutdown abort | 强制关闭，不做任何处理（紧急情况使用） |

### 常用查询

```sql
-- 查看所有用户
SELECT username, created, account_status FROM all_users;

-- 查看当前连接数
SELECT username, COUNT(*) AS connection_count
FROM v$session
GROUP BY username;

-- 查看游标数配置
SHOW PARAMETER open_cursors;

-- 查看数据文件
SELECT file_name, tablespace_name, bytes / 1024 / 1024 AS size_mb
FROM dba_data_files;

-- 查看表空间下的表
SELECT table_name, owner
FROM all_tables
WHERE tablespace_name = 'MY_DATA';
```

### 表空间管理

```sql
-- 修改表空间大小
ALTER DATABASE DATAFILE '/u01/app/oracle/oradata/mydb/data01.dbf' RESIZE 500M;

-- 增加数据文件
ALTER TABLESPACE mydata
ADD DATAFILE '/u01/app/oracle/oradata/mydb/mydata02.dbf' SIZE 100M
AUTOEXTEND ON NEXT 10M MAXSIZE 1G;

-- 删除表空间
DROP TABLESPACE mydata INCLUDING CONTENTS AND DATAFILES;

-- 设置表空间只读
ALTER TABLESPACE mydata READ ONLY;

-- 设置表空间读写
ALTER TABLESPACE mydata READ WRITE;
```

---

## 数据迁移

### 传统导入导出（exp/imp）

传统的 exp/imp 工具适用于小数据量迁移，现在已被数据泵取代。

```bash
# 导出
exp username/password file=/tmp/expfile.dmp log=/tmp/exp.log

# 导入
imp username/password@service_name file=/tmp/expfile.dmp full=y
```

> **注意**：exp/imp 效率较低，建议使用 expdp/impdp（数据泵）。

### 数据泵（expdp/impdp）

数据泵是 Oracle 10g 引入的高效导入导出工具。

**基本语法：**

```bash
# 导出
expdp username/password directory=DATA_PUMP_DIR dumpfile=exp.dmp schemas=scott

# 导入
impdp username/password directory=DATA_PUMP_DIR dumpfile=exp.dmp full=y
```

**常用参数：**

| 参数 | 说明 |
|------|------|
| directory | 导出目录对象（必须先创建） |
| dumpfile | 导出文件名称 |
| schemas | 导出的用户 |
| tables | 导出的表 |
| full | 全量导出 |
| include | 包含的对象类型 |
| exclude | 排除的对象类型 |
| remap_schema | 映射用户 |
| remap_tablespace | 映射表空间 |

**完整示例：**

```sql
-- 1. 创建目录
CREATE DIRECTORY dp_dir AS '/u01/backup';
GRANT READ, WRITE ON DIRECTORY dp_dir TO scott;

-- 2. 导出数据
expdp scott/tiger directory=dp_dir dumpfile=scott_20240301.dmp schemas=scott

-- 3. 导入到新用户
-- 先创建新用户
CREATE USER mydb IDENTIFIED BY mydb;
GRANT CONNECT, RESOURCE TO mydb;

-- 导入（用户映射）
impdp mydb/mydb directory=dp_dir dumpfile=scott_20240301.dmp remap_schema=scott:mydb
```

### 常见错误处理

#### ORA-39001: 参数值无效

```
ORA-39001: 参数值无效
ORA-39000: 转储文件说明错误
ORA-39088: 文件名不能包含路径说明
```

**解决**：确保 dumpfile 不包含路径，使用 directory 参数指定目录。

```bash
# 错误写法
expdp scott/tiger dumpfile=/tmp/exp.dmp

# 正确写法
expdp scott/tiger directory=DUMP_DIR dumpfile=exp.dmp
```

#### ORA-14460: 只指定一个 COMPRESS 或 NOCOMPRESS

**解决**：添加 `transform=segment_attributes:n` 参数忽略表空间约束。

```bash
impdp scott/tiger directory=DUMP_DIR dumpfile=exp.dmp transform=segment_attributes:n
```

#### ORA-12899: 列值太大

```
ORA-12899: value too large for column "NAME" (actual: 82, maximum: 80)
```

**原因**：字符集差异，UTF-8 中文字符占 3 字节，GBK 占 2 字节。

**解决**：
1. 修改目标库字符集
2. 扩大目标字段长度
3. 使用 `convert()` 函数转换

---

## SQL 基础

### 常用函数

#### 字符串函数

| 函数 | 说明 | 示例 |
|------|------|------|
| CONCAT(s1, s2) | 拼接字符串 | CONCAT('Hello', 'World') |
| SUBSTR(s, start, len) | 截取字符串 | SUBSTR('Hello', 2, 3) → ell |
| LENGTH(s) | 返回长度 | LENGTH('你好') → 2 |
| INSTR(s, substr) | 查找位置 | INSTR('Hello', 'll') → 3 |
| REPLACE(s, old, new) | 替换字符串 | REPLACE('Hello', 'l', 'L') |
| TRIM(s) | 去除空格 | TRIM(' Hello ') |
| LPAD(s, len, pad) | 左填充 | LPAD('5', 5, '0') → 00005 |
| RPAD(s, len, pad) | 右填充 | RPAD('5', 5, '0') → 50000 |

#### 数值函数

| 函数 | 说明 | 示例 |
|------|------|------|
| ROUND(n, d) | 四舍五入 | ROUND(3.14159, 2) → 3.14 |
| TRUNC(n, d) | 截断 | TRUNC(3.14159, 2) → 3.14 |
| MOD(m, n) | 取余 | MOD(10, 3) → 1 |
| CEIL(n) | 向上取整 | CEIL(3.1) → 4 |
| FLOOR(n) | 向下取整 | FLOOR(3.9) → 3 |

#### 日期函数

| 函数 | 说明 | 示例 |
|------|------|------|
| SYSDATE | 当前日期时间 | SYSDATE |
| SYSTIMESTAMP | 当前时间戳 | SYSTIMESTAMP |
| ADD_MONTHS(d, n) | 加月份 | ADD_MONTHS(SYSDATE, 3) |
| MONTHS_BETWEEN(d1, d2) | 月份差 | MONTHS_BETWEEN(d1, d2) |
| TRUNC(d, format) | 日期截断 | TRUNC(SYSDATE, 'YYYY') |
| TO_CHAR(d, format) | 日期转字符串 | TO_CHAR(SYSDATE, 'YYYY-MM-DD') |
| TO_DATE(s, format) | 字符串转日期 | TO_DATE('2024-01-01', 'YYYY-MM-DD') |

#### 类型转换函数

| 函数 | 说明 | 示例 |
|------|------|------|
| TO_CHAR | 转换为字符 | TO_CHAR(123) → '123' |
| TO_NUMBER | 转换为数字 | TO_NUMBER('123') → 123 |
| TO_DATE | 转换为日期 | TO_DATE('2024', 'YYYY') |
| CAST | 类型转换 | CAST('123' AS NUMBER) |
| DECODE | 条件转换 | DECODE(status, 1, '启用', '禁用') |

### 常用查询

```sql
-- 分页查询（12c 之前）
SELECT * FROM (
    SELECT ROWNUM AS rn, t.* FROM (
        SELECT * FROM emp ORDER BY empno
    ) t WHERE ROWNUM <= 20
) WHERE rn > 10;

-- 分页查询（12c+）
SELECT * FROM emp ORDER BY empno OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;

-- 递归查询（树形结构）
SELECT employee_id, last_name, manager_id, LEVEL
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;

-- 高级分组
SELECT department_id, job_id, SUM(salary)
FROM employees
GROUP BY ROLLUP(department_id, job_id);

-- 分析函数
SELECT employee_id, department_id, salary,
       RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank,
       DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank,
       ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS row_num
FROM employees;
```

---

## 表分区

### 什么是表分区

当表数据量不断增大时，查询性能会下降。表分区将数据在物理上分散存储到多个表空间，但逻辑上仍是一张完整的表。

**分区优点：**
- 提高查询性能（只需扫描相关分区）
- 便于数据维护（可独立管理分区）
- 提高可用性（单个分区损坏不影响其他分区）
- 支持数据生命周期管理（历史数据归档）

### 范围分区

基于列值范围进行分区，适合日期、数值类型。

```sql
CREATE TABLE order (
    order_id NUMBER(10) NOT NULL,
    order_date DATE NOT NULL,
    status VARCHAR2(20)
)
PARTITION BY RANGE (order_date) (
    PARTITION p2023_q1 VALUES LESS THAN (TO_DATE('2023-04-01', 'YYYY-MM-DD')),
    PARTITION p2023_q2 VALUES LESS THAN (TO_DATE('2023-07-01', 'YYYY-MM-DD')),
    PARTITION p2023_q3 VALUES LESS THAN (TO_DATE('2023-10-01', 'YYYY-MM-DD')),
    PARTITION p2023_q4 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
    PARTITION p_max VALUES LESS THAN (MAXVALUE)
);
```

### 列表分区

基于离散值列表进行分区。

```sql
CREATE TABLE sales (
    sale_id NUMBER(10),
    region VARCHAR2(20),
    amount NUMBER(12, 2)
)
PARTITION BY LIST (region) (
    PARTITION p_east VALUES ('北京', '上海', '广东'),
    PARTITION p_west VALUES ('四川', '重庆', '陕西'),
    PARTITION p_other VALUES (DEFAULT)
);
```

### 散列分区

使用散列算法均匀分布数据，适合没有明显范围特征的列。

```sql
CREATE TABLE sales_hash (
    sale_id NUMBER(10),
    amount NUMBER(12, 2)
)
PARTITION BY HASH (sale_id) (
    PARTITION p1 TABLESPACE ts1,
    PARTITION p2 TABLESPACE ts2,
    PARTITION p3 TABLESPACE ts3,
    PARTITION p4 TABLESPACE ts4
);
```

### 复合分区

#### 范围-列表复合分区

```sql
CREATE TABLE sales_range_list (
    sale_id NUMBER(10),
    sale_date DATE,
    region VARCHAR2(20),
    amount NUMBER(12, 2)
)
PARTITION BY RANGE (sale_date)
SUBPARTITION BY LIST (region) (
    PARTITION p2023 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD'))
    SUBPARTITION p2023_east VALUES ('北京', '上海'),
    SUBPARTITION p2023_west VALUES ('四川', '重庆'),
    SUBPARTITION p2023_other VALUES (DEFAULT)
);
```

### 分区维护

```sql
-- 查看分区
SELECT partition_name, tablespace_name
FROM user_tab_partitions
WHERE table_name = 'ORDER';

-- 查询特定分区数据
SELECT * FROM sales PARTITION (p2023_q1);

-- 添加分区
ALTER TABLE order ADD PARTITION p2024_q1
VALUES LESS THAN (TO_DATE('2024-04-01', 'YYYY-MM-DD'));

-- 删除分区
ALTER TABLE order DROP PARTITION p2023_q1;

-- 合并分区
ALTER TABLE order MERGE PARTITIONS p2023_q3, p2023_q4 INTO PARTITION p2023_h2;

-- 拆分分区
ALTER TABLE order SPLIT PARTITION p2023_h2
AT (TO_DATE('2023-10-01', 'YYYY-MM-DD'))
INTO (PARTITION p2023_q3, PARTITION p2023_q4);
```

---

## 性能优化

### 执行计划分析

执行计划展示了 SQL 的执行方式，是优化的基础。

```sql
-- 查看执行计划
EXPLAIN PLAN FOR
SELECT e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 5000;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- 实时查看执行计划（SQL 正在执行）
SELECT * FROM v$sql_plan WHERE sql_id = 'your_sql_id';

-- 查看历史执行计划
SELECT * FROM v$sql_plan_history WHERE sql_id = 'your_sql_id';
```

### 查看执行计划的常用视图

```sql
-- 查看正在执行的 SQL
SELECT sql_text, sql_id, status
FROM v$session s
JOIN v$sqlarea a ON s.sql_id = a.sql_id
WHERE s.username = 'SCOTT';

-- 查看等待事件
SELECT event, wait_time, state
FROM v$session_wait
WHERE sid = (SELECT sid FROM v$mystat WHERE ROWNUM = 1);

-- 查看系统统计信息
SELECT * FROM v$sysstat WHERE statistic# = 12;  -- 物理读取
```

### 索引优化

```sql
-- 创建索引
CREATE INDEX idx_emp_dept ON employees(department_id);
CREATE INDEX idx_emp_name ON employees(last_name, first_name);

-- 创建唯一索引
CREATE UNIQUE INDEX uk_emp_email ON employees(email);

-- 创建函数索引
CREATE INDEX idx_emp_upper ON employees(UPPER(last_name));

-- 重建索引
ALTER INDEX idx_emp_dept REBUILD;

-- 合并索引
ALTER INDEX idx_emp_dept COALESCE;

-- 删除索引
DROP INDEX idx_emp_dept;
```

### Hint 优化

Hint 可以影响优化器的执行计划选择。

```sql
-- 强制使用索引
SELECT /*+ INDEX(e idx_emp_dept) */ *
FROM employees e
WHERE department_id = 50;

-- 强制全表扫描
SELECT /*+ FULL(e) */ *
FROM employees e
WHERE department_id = 50;

-- 强制排序合并连接
SELECT /*+ USE_MERGE(e d) */ *
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- 强制嵌套循环连接
SELECT /*+ USE_NL(e d) */ *
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- 并行执行
SELECT /*+ PARALLEL(e, 4) */ *
FROM employees e;
```

### 常用性能视图

```sql
-- 查看 Top SQL（按 CPU 时间）
SELECT sql_id, sql_text, cpu_time / 1000000 AS cpu_sec, executions
FROM v$sqlarea
ORDER BY cpu_time DESC
FETCH FIRST 10 ROWS ONLY;

-- 查看 Top SQL（按物理读取）
SELECT sql_id, sql_text, buffer_gets, disk_reads
FROM v$sqlarea
ORDER BY disk_reads DESC
FETCH FIRST 10 ROWS ONLY;

-- 查看表统计信息
SELECT table_name, num_rows, blocks, avg_row_len
FROM user_tables
WHERE table_name = 'EMPLOYEES';

-- 查看索引统计信息
SELECT index_name, blevel, leaf_blocks, distinct_keys
FROM user_indexes
WHERE table_name = 'EMPLOYEES';
```

---

## 常见问题

### 连接问题

**Q: ORA-12541: TNS: 无监听程序**

```bash
# 检查监听器状态
lsnrctl status

# 启动监听器
lsnrctl start
```

**Q: ORA-01017: 用户名/口令无效**

```bash
# 检查用户名密码是否正确
# 注意 Oracle 密码区分大小写（11g+ 默认区分）
```

**Q: ORA-12514: TNS: 监听程序当前无法识别连接描述符中请求的服务**

检查 tnsnames.ora 中的 SERVICE_NAME 是否正确，或服务是否启动。

### 性能问题

**Q: SQL 执行很慢，怎么排查？**

```sql
-- 1. 查看执行计划
EXPLAIN PLAN FOR your_sql;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- 2. 检查统计信息是否最新
SELECT table_name, num_rows, last_analyzed
FROM user_tables;

-- 3. 重新收集统计信息
EXEC DBMS_STATS.GATHER_TABLE_STATS(USER, 'YOUR_TABLE');
```

**Q: 如何查看锁表？**

```sql
-- 查看被锁的表
SELECT l.session_id AS sid,
       l.locked_mode,
       l.oracle_username,
       l.os_user_name,
       o.object_name,
       o.object_type
FROM v$locked_object l
JOIN dba_objects o ON l.object_id = o.object_id;

-- 杀掉会话
ALTER SYSTEM KILL SESSION 'sid, serial#';
```

### 表空间问题

**Q: ORA-01653: 表空间不足**

```sql
-- 1. 查看表空间使用情况
SELECT tablespace_name, (bytes - free_bytes) / bytes * 100 AS usage_pct
FROM (SELECT tablespace_name, bytes, LEAD(bytes) OVER (ORDER BY tablespace_name) AS free_bytes
      FROM (SELECT tablespace_name, SUM(bytes) AS bytes
            FROM dba_data_files GROUP BY tablespace_name));

-- 2. 扩展表空间
ALTER TABLESPACE tablespace_name ADD DATAFILE '/path/to/file.dbf' SIZE 100M;
```

### 导入导出问题

**Q: impdp 导入报错 ORA-01917？**

用户不存在，需要先创建用户：
```sql
CREATE USER newuser IDENTIFIED BY password;
GRANT CONNECT, RESOURCE TO newuser;
```

**Q: 如何查看导入导出进度？**

```sql
-- 查看 Job 状态
SELECT * FROM dba_datapump_jobs;
```

---

## 参考资料

- [Oracle 官方文档](https://docs.oracle.com/en/database/oracle/oracle-database/)
- [Oracle Live SQL](https://livesql.oracle.com/)
- [Oracle Database 19c 安装指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/xe/install-and-configure-oracle-database.html)
- [SQL 语言参考](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/)
- [DBMS_STATS 包文档](https://docs.oracle.com/en/database/oracle/oracle-database/21/arpls/DBMS_STATS.html)