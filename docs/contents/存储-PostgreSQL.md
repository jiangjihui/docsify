# PostgreSQL

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

7.  **复制与高可用性**：
   
   - PostgreSQL：提供流复制和逻辑复制，支持异步和同步复制，并且一致性较好。社区还提供了成熟的分片解决方案（如Citus）。
   - MySQL：支持多种复制方式，包括主从复制和组复制等。但其在一致性方面可能稍逊于PostgreSQL，主从复制的延迟是常见问题。不过，MySQL也通过MySQL Group Replication、Galera Cluster等工具实现了高可用性和负载均衡。

8. **应用场景**：
   
   - PostgreSQL：更适合需要复杂查询、数据一致性、高度扩展性和高级SQL功能的场景，如金融系统、科学计算、大数据分析等。
   - MySQL：更适合需要快速读写操作、较简单的查询和广泛的社区支持的场景，如Web应用、内容管理系统、电子商务网站等。

9. 
   
   

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


