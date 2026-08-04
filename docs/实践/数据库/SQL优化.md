# SQL 优化

SQL 优化的方式方法很多，此处罗列一些比较重要，容易出现问题的优化点：

- 尽量避免全表扫描，应考虑在 where 和 order by 涉及的列上建立索引
- 被驱动表的关联字段需要创建索引，否则被驱动表会走全表扫描
- 索引遵循最左前缀匹配原则，like 写法只能将 % 放在右边，如 name like ‘掌门学员%’；若 % 放在左边会导致索引失效
- 应尽量避免在 where 子句中使用 != ， <> ，not in 操作符，会导致索引失效
- 应尽量避免在 where 子句中使用 or 来连接条件，可尝试拆分为 union/union all
- 可考虑使用 join/left join 关联查询，替代子查询、in/not in 的写法，尽量不要用子查询
- 如果 in 的内容是连续的，可使用 between…and 或者 >….< 替代，改走范围扫描
- 不要在查询字段上使用函数或者表达式，会导致索引失效，可在参数字段上做函数或表达式运算
- 查询时需根据字段定义类型进行传参 ，若参数类型与字段定义类型不一致，会导致索引失效
- 不要使用 select * 写法，只查询需要的列
- 不要在数据库中使用变量 + 分组排序方式构造排序字段，MySQL 需要基于全量数据做排序分组，效率很低
- 通过 limit 方式分页会导致后续分页越来越慢，可取前一次分页的最大 ID 作为下一页参数输入，进行分页
- 不要通过 order by rand() 方式取随机数，效率极低。如果需要取随机数，可以先用随机数方法取得一个整数，然后根据 id>= 该整数即可。
- 禁止不必要的排序，排序操作极耗资源，不要轻易分组排序
- 不要在程序中使用 using index 强制索引写法



## SQL 优化示例

枯燥的知识点，哪有举个例子来的更清晰易懂，本章节我们用具体的例子讲述 SQL 优化的方法跟技巧。

### 使用 union/union all 替换 or 写法

MySQL 的 or 写法，会导致查询索引失效，而更换为 union/union all 写法，虽然代码长度会增加，但是查询效率会有很大的提升

```sql
select count(0)
from students
where students.created_at>='2021-05-01'
  or students.referrer_user_id>0;
```

> 说明：students 表的 created_at 和 referrer_user_id 列都有索引，但是由于查询使用 or 连接，导致无法走索引扫描，type 为 all 说明查询走全表扫描，该查询 10min 仍然无法查询出结果

**优化：**

使用 union/union all 写法代替 or。鉴于 or 的关联记录可能存在重复，使用 union 写法，在外层对结果进行 count

```sql
select count(0)
from 
(select id
from students
where students.created_at>='2021-05-01'
union
select id
from students
0where students.referrer_user_id>0) as t;
```

> 说明：改用 unio 后两个子查询中都能走各自对应的索引，并且因为是查询 ID 信息，查询走覆盖索引 using index，效率很高，该查询最终耗时 11.64s

如果查询条件改为=，由于**索引合并**的存在，可能不会走全表扫描；即使查询条件为范围查询，也不一定都会都走全表扫描，视数据量而定。



### 使用 join 关联替换子查询、in、exists 写法

子查询的写法虽然看起来直观清晰，但是子查询是一个独立的查询，不能参与到驱动表的数据过滤，而且子查询的结果数据会被放到临时表中存放，然后与驱动表进行关联，效率非常低。

在 MySql 的写法中，非常不建议子查询写法，而应该尽量用 join 方式替换子查询写法。

```sql
select sum(t.money)
from students
join 
(select payments.stu_id,payments.money
from payments
where payments.is_paid=1
  and payments.is_canceled=0
    and payments.money>0) as t on students.id=t.stu_id
    where students.created_at>='2021-05-01'
      and students.created_at< '2021-06-01';
```

**优化：**

不使用子查询，直接使用 join 进行多表关联，并将查询条件写入到 where 中。

```sql
select sum(payments.money)
from students
join payments on students.id=payments.stu_id
where payments.is_paid=1
  and payments.is_canceled=0
    and payments.money>0
    and students.created_at>='2021-05-01'
    and students.created_at< '2021-06-01';
```

> 说明：不使用子查询，改为 join 写法后。查询首先根据 students.created_at 走索引扫描，然后根据查询的结果与 payments 表的 stu_id 进行关联。查询耗时 1.85s



### 使用 join/left join 替代 in/exists、not in/not exists 写法

join 写法相当于做等值匹配，可以直接替代大部分场景的 exits 和 in 的写法

left join/right join 外关联的写法会以驱动表为母表，被驱动表只包含匹配的数据，配合在 where 条件中添加条件筛选，可用来替代 not exist 和 not in 写法

```sql
select u.id 
from users u 
where u.updated_at > '2021-05-20 12:00:00'
and u.id not in(select a.user_id from users_account_number a);
```

> 说明：查询是统计 users 表创建时间大于 '2021-05-20 12:00:00'，且不在 users_account_number 表中的用户 ID
> 请注意 id=2 的 select_typ 类型为 DEPENDENT SUBQUERY，表示外层 select 结果需要依赖于子查询的结果。效率会非常差

**优化：**

直接用 join/left join 的写法替代，效率将会有大幅提升

```sql
select u.id 
from users u 
left join users_account_number a on u.id=a.user_id
where u.updated_at > '2021-05-20 12:00:00'
and a.user_id is null;
```

> 说明：查询 select_type 都变为 simple ，并且查询先基于 updated_at 做索引过滤，然后与 users_account_number 进行匹配，效率非常高



### 根据字段类型合理传参

在 MySQL 中，如果参数类型与字段定义类型不一致，会导致查询无法走到索引，这点需要关注。

```sql
select *
from users
where mobile=13999999999;
```

> 说明：users 表的 mobile 字段是 varchar 类型，并且创建有索引，但是输入参数是整形，导致查询无法走索引扫描，type 为 ALL 全表扫描。

**优化：**

根据字段类型进行合理的传参，如果 mobile 为 varchar 类型，则在参数上添加引号''标识为字符类型

```
select *
from users
where mobile='13999999999';
```

> 说明：mobile 字段为 varchar 类型，因此在参数上使用‘’标识为字符类型，查询走到 mobile 列的唯一索引，效率非常高。



### 不要在查询字段上使用函数或表达式

如果在查询字段上使用函数或者表达式，MySQL 会首先对查询字段做函数运算，如果原本是准备基于该字段做索引匹配，函数运算会导致索引失效

```sql
select count(0)
from students
join students_seller on students.id=students_seller.student_id
where date(students.created_at)='2021-05-05'
 and students_seller.state=0;
```

> 说明：由于在 students.created_at 字段上使用 date 函数，导致无法通过 created_at 列匹配满足时间要求的数据，通过 rows 列可以看到是全表扫描 5285w 数据量。
> 或许有人会疑问既然是全表扫描，为什么 type 是 index 而不是 ALL。这是因为 students 表只用到 created_at 和 id 列，这两列是直接包含在索引中的，查询是走的覆盖索引。

**优化：**

不要在查询字段上使用函数，如果需要使用函数，那么函数可以用在参数列上

```sql
select count(0)
from students
join students_seller on students.id=students_seller.student_id
where students.created_at>='2021-05-05 00:00:00'
 and students.created_at<='2021-05-05 23:59:59'
 and students_seller.state=0;
```

> 说明：由于是查询注册时间为 2021-05-05 日的学生，那么可以替代为使用 created_at>= ... and <= 的写法，优化后查询根据 students.created_at 走索引范围查询，匹配行数 rows 为 134092 条，然后与 students_seller 表关联，效率非常高。



### **防止分页过大导致查询越来越慢，可使用id优化**

在程序开发中，经常会对大量的返回结果进行分页返回展示，但是当分的页数过大，后续的分页查询会越来越慢，例如 limit 100 和limit 1000000,100 的查询效率差别很大。

```sql
select students.id,students.user_id,students_seller.seller_id,students.stu_city
from students
join students_seller on students.id=students_seller.student_id
where students.created_at>='2021-01-01'
 and students_seller.state=0
 limit 100;
 -- limit 1000000,100;
```

> 说明：虽然 limit 100 和limit 1000000,100 的解释计划相同，但是执行时间差异非常大。
> limit m,n 中的 m 数越大，则查询越慢。limit 100 的执行时间 0.05s；limit 1000000,100 的执行时间 35s

**优化：**

MySQL 更适合于精确查询，如果满足条件的结果很多，是不合理的，应该通过范围限制使得满足条件的结果足够小，最终结果应该控制在 1000 条、100 条甚至 10 条以内。

但是如果一定要对大量结果数据分页，那么可以考虑根据 ID 进行适当的范围限制

```sql
select students.id,students.user_id,students_seller.seller_id,students.stu_city
from students
join students_seller on students.id=students_seller.student_id
where students.created_at>='2021-01-01'
 and students_seller.state=0
 and students.id>54978976 -- 取上一次分页的最大ID
limit 100;
```

> 说明：例如假设第 10000 页取到的 100 行数据中最大的 students.ID 为 54978976，现在需要取得第 10001 页的 100 行数据
> 那么可以添加 students.id>54978976,然后取 100 行数据，此方法取得的即为第 10001 页的数据。
> 通过 id 字段的范围限制，比简单的 limit m,n 更加高效，即使是大的数据分页也不会导致效率变低。该方法执行时间稳定为 0.05s



### **where + order by + limit 查询陷阱**

在程序设计中，经常需要用到 where+order by+limit 写法获取满足条件的、排序后的 N 条数据结果。这种写法本身并没有问题，但是在实际使用中，这条语句却常常引起严重的性能问题，需要我们重点关注。

```sql
select students.id,students.user_id,students_seller.seller_id,students.stu_city
from students
join students_seller on students.id=students_seller.student_id
where students.created_at>='2021-04-05'
 and students.created_at<='2021-04-05 23:59:59'
 and students_seller.state=0
order by students.updated_at desc 
limit 100;
```

> 说明：查询是想取得学生注册时间在 2021-04-05 的，并且 state=0 的，按照更新时间取得最近 100 条学生记录。
> 请注意解释计划中得 key 为 updated_at 字段，我们明明是对 created_at 字段做范围限制，为啥 MySQL 选择走 updated_at 列索引？

当查询语句中包含 where+order by+limit 时

由于 MySQL 优化器会认为排序 order by 是非常耗时的操作，如果能在一开始就将结果做排序返回，那是最好的。而且 limit 100 会让优化器认为这 100 条记录是很容易满足的，此时优化器会走如下执行计划：

1. 由于 students.updated_at 字段本身就有索引（已排序），按照 updated_at 字段每次取 100 条结果，然后走 where 匹配，将满足的结果返回
2. 重复以上操作，直到有 100 条结果满足要求，循环结束
3. 如果查询按照 students.updated_at 排序的数据经过 where 条件过滤后能快速满足 100 条结果输出，那么查询或许会非常快
4. 但如果一致没有取到满足 where 条件的 100 条结果，会一直循环操作直至遍历 students 整个表！那这个代价是非常恐怖的

**优化：**

可以有很多办法避免以上执行计划与预想不一致的情况，本文列举两种办法：

方法1. 

order by 的字段调整为与查询条件中的字段一致，但是可能造成结果并非原始期望结果，需要沟通业务部门是否能接受该 SQL 改造。（最优）

```sql
select students.id,students.user_id,students_seller.seller_id,students.stu_city
from students
join students_seller on students.id=students_seller.student_id
where students.created_at>='2021-04-05'
 and students.created_at<='2021-04-05 23:59:59'
 and students_seller.state=0
order by students.created_at desc 
limit 100;
```

> 说明：排序字段修改为与查询字段一致即 students.created_at，由于索引查询本身就是排序的，不必再额外排序，效率非常高。

方法2.

将内部查询使用括号包裹，强制其作为一个先导执行的子查询，然后对子查询的最终结果集进行排序返回。由于子查询中会存放所有满足条件的结果，并且进行文件排序，如果满足条件的结果非常大，该方法会消耗较多资源效率较低。（亦可）

```sql
select t.*
from
(select students.id,students.user_id,students_seller.seller_id,students.stu_city,students.updated_at
from students
join students_seller on students.id=students_seller.student_id
where students.created_at>='2021-04-05'
 and students.created_at<='2021-04-05 23:59:59'
 and students_seller.state=0) as t
0order by t.updated_at desc 
1limit 100;
```

> 说明：查询根据 created_at 字段检索满足条件的记录构建为子查询。外层查询结果对子查询进行排序然后取得 100 条记录。
> 此方法不会改变原始业务逻辑，如果满足条件结果集较小，效率很高；但是如果满足条件的中间结果集非常大，则查询效率也会较差



> 文章来源：[掌门 MySQL 数据库规约落地及优化实战](https://mp.weixin.qq.com/s/GpRX1uEqUz4eRAQpEU3-eg)





### null 查询

```
select * from staff where name like null;
```

即使union_id字段有索引，进行like null查询的话，也会进行全表扫描。而且和 `is null` 不一样，`like null`查不出任何结果。所以在平时使用的过程中，尽量避免出现like null查询，使用`is null` 或者`like ''`。





