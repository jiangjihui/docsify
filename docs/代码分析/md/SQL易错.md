# SQL 易错点

> left join 的 on/where 之别、NULL 的三值逻辑、隐式转换废掉索引——SQL 的坑不在语法，在语义。写不对不报错，只是悄悄返回不完整的结果集。

> ⚠️ 本文行为描述按 MySQL 8.0 语义推写（本机无 MySQL 环境未实测），标注「实测」的除外。跨库（PG/Oracle）行为以各自文档为准。

## 速查表

| 陷阱 | 一句话原理 | 避坑 |
|------|-----------|------|
| left join 后过滤条件放 where | where 在 join 之后过滤, 左表行会被删 | 右表条件放 on, 主表条件放 where |
| `not in` 碰 NULL 全空 | null 参与比较结果是 UNKNOWN | not exists 或先过滤 null |
| `in (null,...)` 悄悄丢数据 | 同上三值逻辑 | 列参与 is null or in 显式处理 |
| `not like` 查不到 NULL 行 | null 无法参与 like 比较 | 补 or col is null |
| 字符串列传数字 | 隐式转换每行做 cast, 索引失效 | 查询参数类型对齐列类型 |
| count(列) vs count(*) | count(列) 跳过 NULL | 要总行数用 count(*) |
| delete / update 忘 where | 语法合法, 全表生效 | 先 select 同条件, 开事务, 看影响行数 |

## JOIN 语义

### left join：on 与 where 的本质区别

**现象**（按 MySQL 8.0 语义推写）：

```sql
-- 表1 table1(id, No): (1,'n1'), (2,'n2'), (3,'n3')
-- 表2 table2(No, name): ('n1','aaa'), ('n2','bbb'), ('n3','ccc')

select a.id, a.No, b.name from table1 a
left join table2 b on (a.No = b.No and b.name = 'aaa');
-- 返回 3 行: id=1 带 aaa; id=2,3 的 b.name 为 NULL（左表行保住）

select a.id, a.No, b.name from table1 a
left join table2 b on (a.No = b.No) where b.name = 'aaa';
-- 返回 1 行: 只有 id=1（where 把 NULL 行过滤掉了, left join 名存实亡）
```

**原理**：SQL 逻辑执行顺序 `JOIN(on) → WHERE → GROUP BY → HAVING → SELECT`。on 条件参与**连接阶段**——不满足的右侧行不连接，但**左表行保留**（右侧补 NULL）；where 在连接结果上过滤，NULL 行不满足任何等值条件，**左表行被删除**。把右表条件放 where，left join 退化成 inner join。

**避坑**：一句话记忆——**on 决定「怎么连」，where 决定「留哪些」**。左连接想保留左表全量，右表相关条件必须放 on。

### 聚合统计：count 的三种写法

```sql
select sid, sname, avg(grade), max(grade), sum(grade),
       count(distinct course_id) as 课程数
from student
left join score on sid = student_id
group by sid;
```

| 写法 | 语义 |
|------|------|
| `count(*)` | 总行数（含 NULL） |
| `count(grade)` | grade 非 NULL 的行数 |
| `count(distinct course_id)` | 去重后的非 NULL 值个数 |

**原理**：聚合函数（sum/avg/max/count(列)）**跳过 NULL**——sum 全 NULL 得 NULL 不是 0，avg 分母只数非 NULL 行。left join 右表无匹配时右侧全 NULL，`count(右侧列)` 恰好等于「匹配行数」，这是统计匹配数的惯用技巧。

**避坑**：要「总行数」永远 `count(*)`（优化器有专门优化，不比 count(1) 慢）；`sum(可能为NULL的列)` 记得 `coalesce(sum(x), 0)`。

## NULL 三值逻辑

### 为什么 null 查不出来、也过滤不掉

SQL 的比较结果是**三值**：TRUE / FALSE / **UNKNOWN**（任一侧为 NULL 时）。where 只保留 TRUE，UNKNOWN 和 FALSE 一样被过滤。

| 表达式 | 结果 | 对查询的影响 |
|--------|------|-------------|
| `x = null` | UNKNOWN | 永远查不到 |
| `x <> null` | UNKNOWN | 永远查不到 |
| `null = null` | UNKNOWN | 两个 null 也「不相等」 |
| `x in (null, 1, 3)` | x=1 时 TRUE, 其余 UNKNOWN | 非匹配行丢失 |
| `x not in (null, 1, 3)` | 全部 UNKNOWN（not UNKNOWN 还是 UNKNOWN） | **一行都查不出来** |
| `name not like 'abc%'`（name 为 NULL） | UNKNOWN | NULL 行查不出来 |

**三个典型翻车场景**：

```sql
-- 1. not in 子查询含 NULL → 结果集永远为空
select * from course
where teacher_id not in (select teacher_id from student);
-- student 里只要有一行 teacher_id 为 NULL, 整个查询返回 0 行

-- 2. in 想同时覆盖 null → 必须显式写
select * from t where x is null or x in (1, 3);

-- 3. not like 想包含 NULL 行 → 必须补条件
select * from course
where name not like 'abc%' or name is null;
```

**原理展开**：`not in (null, 1, 3)` 展开为 `x <> null and x <> 1 and x <> 3`——第一项 UNKNOWN 与任何值 and 都不是 TRUE，整个条件永远非 TRUE，**一行不返回**。这是「上线后发现某个查询永远空结果」的头号嫌疑人。

**避坑**：not in 子查询一律改 `not exists`（其对 NULL 免疫）；或子查询里先 `where col is not null`。字段设计上能 not null 就 not null，默认值比 NULL 好处理。

## 类型与索引

### 隐式转换：索引悄悄失效

```sql
-- phone 是 varchar 且有索引
select * from user where phone = 13800001111;   -- 数字字面量
-- MySQL 把「每一行的 phone」转成数字再比较 → 索引失效, 全表扫描
```

**原理**：字符串列与数字比较时，MySQL 的规则是**把字符串转数字**（而不是把数字转字符串）——转换发生在行数据上，无法走索引。反向（数字列传字符串）反而能走索引（常量被转换一次），但两边类型一致才是正解。

**避坑**：查询参数类型与列类型严格对齐（ORM 里注意实体字段类型）；字符串列查询永远带引号 `'13800001111'`。排查「明明有索引却全表扫」时先 `explain` 看type 列，再看 where 里有没有类型不匹配。

## 高危操作清单

发布前 30 秒自检（都发生在「以为加了条件其实没加」的瞬间）：

| 操作 | 事故形态 | 保命动作 |
|------|---------|---------|
| `delete from t;` | 忘 where, 全表清空 | 先 `select count(*) where 同条件` 确认范围 |
| `update t set x=...;` | 忘 where, 全表改写 | 开事务 + 先跑同条件 select + commit 前看影响行数 |
| `truncate table t` | DDL 不可回滚 | 权限收口, 生产禁用 |
| `drop table` | 同上 | 同上 |

## 相关文档

- SQL 性能优化（索引、执行计划、分页）见 [SQL 优化](/实践/数据库/SQL优化.md)
- MySQL 系统学习见 [MySQL](/contents/storage/MySQL.md)
