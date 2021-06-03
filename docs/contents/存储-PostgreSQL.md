# PostgreSQL

## 简介

Pg 不仅仅是 SQL 数据库它可以存储 array 和 json, 可以在 array 和 json 上建索引, 甚至还能用表达式索引. 为了实现文档数据库的功能, 设计了 jsonb 的存储结构. 有人会说为什么不用 Mongodb 的 BSON 呢? Pg 的开发团队曾经考虑过, 但是他们看到 BSON 把 ["a", "b", "c"] 存成 {0: "a", 1: "b", 2: "c"} 的时候就决定要重新做一个 jsonb 了... 现在 jsonb 的性能已经优于 BSON.

MySQL 的各种 text 字段有不同的限制, 要手动区分 small text, middle text, large text... Pg 没有这个限制, text 能支持各种大小.

它自带全文搜索功能 (不用费劲再装一个 elasticsearch 咯)。

它有地理信息处理扩展 (GIS 扩展不仅限于真实世界, 游戏里的地形什么的也可以), 可以用 Pg 搭寻路服务器和地图服务器。PG 多年来在 GIS 领域处于优势地位，因为它有丰富的几何类型，实际上不止几何类型，PG有大量字典、数组、bitmap 等数据类型，相比之下mysql就差很多，instagram就是因为PG的空间数据库扩展POSTGIS远远强于MYSQL的my spatial而采用PGSQL的。

它可以把 70 种外部数据源 (包括 Mysql, Oracle, CSV, hadoop ...) 当成自己数据库中的表来查询。



## 基本操作

**登陆PostgreSQL控制台**

```
# psql
或
# psql -U postgres
```

 

**设置用户密码**

```
> \password postgres
```

> 备注：设置psotgres用户密码

 

**创建数据库用户**

```java
> CREATE USER dbuser WITH PASSWORD 'password';
```

 

**创建数据库**

```
> CREATE DATABASE exampledb OWNER dbuser;
```

 

**角色赋权**

```
> GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;
```

 

**退出控制台**

```
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



