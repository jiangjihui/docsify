# 易错代码分析

## 简单统计

### join使用

student

| sid | sname | sex |
| --- | ----- | --- |
| 1   | aa    | m   |
| 2   | bb    | f   |
| 3   | cc    | m   |

course

| cid | cname  |
| --- | ------ |
| 1   | java   |
| 2   | python |
| 3   | php    |

score

| id  | student_id | course_id | grade |
| --- | ---------- | --------- | ----- |
| 1   | 1          | 1         | 80    |
| 2   | 1          | 2         | 88    |
| 3   | 1          | 3         | 90    |
| 4   | 2          | 1         | 80    |
| 1   | 2          | 2         | 88    |
| 6   | 2          | 3         | 70    |
| 7   | 3          | 1         | 75    |

student为主表查询相关信息：最大值，总和

```
select sid,sname,
avg(grade),max(grade),sum(grade) ,count(distinct course_id) 
from student
left join score on sid = student_id 
group by sid;
```

course为主表查询相关信息：平均值，大于某个条件的值

```
select cid,cname,avg(grade),sum(if(grade > 80 , 1, 0))
from course
left join score on score.course_id = cid
group by cid;
```

## SQL语法

### left join on 后and 和 where 的使用

来源：[left join on 后and 和 where 的区别](https://blog.csdn.net/ahwsk/article/details/82886732)

表1：table2

| id  | No  |
| --- | --- |
| 1   | n1  |
| 2   | n2  |
| 3   | n3  |

表2：table2

| No  | name |
|:--- |:---- |
| n1  | aaa  |
| n2  | bbb  |
| n3  | ccc  |

```sql
select a.id,a.No,b.name from table1 a left join table2 b on (a.No = b.No and b.name='aaa');
select a.id,a.No,b.name from table1 a left join table2 b on (a.No = b.No) where b.name='aaa';
```

第一个结果集：

```
|id |No |name|
|---|---|---|
|1  |n1 |aaa|
|2  |n2 |(Null)|
|3  |n3 |(Null)|    
```

第二个结果集：

```
|id |No |name|
|---|---|---|
|1  |n1 |aaa|
```

第一个sql的执行流程：首先找到b表的name为aaa的记录行`（on (a.No = b.No and b.name=’aaa’) ）`。然后找到a的数据（即使不符合b表的规则），生成临时表返回用户。

第二个sql的执行流程：首先生成临时表，然后执行where过滤`b.name=’aaa’`不为真的结果集，最后返回给用户。

**因为on会首先过滤掉不符合条件的行，然后才会进行其它运算，所以按理说on是最快的。**

在多表查询时，on比where更早起作用。系统首先根据各个表之间的联接条件，把多个表合成一个临时表后，再由where进行过滤，然后再计算，计算完后再由having进行过滤。由此可见，要想过滤条件起到正确的作用，首先要明白这个条件应该在什么时候起作用，然后再决定放在那里。

对于JOIN参与的表的关联操作，**如果需要不满足连接条件的行也在我们的查询范围内的话，我们就必需把连接条件放在ON后面，而不能放在WHERE后面**（比如左表的数据范围比连接表的范围大，连接条件放在on后面，不满足连接表的会以null形式显示，如果放在where后面，就会把这些数据过滤掉）**。**

如果我们把连接条件放在了WHERE后面，那么所有的LEFT,RIGHT,等这些操作将不起任何作用，对于这种情况，它的效果就完全等同于INNER连接。对于那些不影响选择行的条件，放在ON或者WHERE后面就可以。

**注意：所有的连接条件都必需要放在ON后面，不然前面的所有LEFT,和RIGHT关联将作为摆设，而不起任何作用。**

## NULL字段的处理

null值需要用is null或者 is not，在有子查询的情况下，如果子查询可能会返回null类型的数据，会导致整个子查询的结果集无效，比如：

```sql
select * from course where teacher_id not in (select teacher_id from student)
```

如果 `student` 表中存在一条`teacher_id`为空的数据，将导致整个结果集都查询不到数据。

所以下面情况使用时要注意下：

**1. x in**

(null,1,3)的情况，相当于(x = null or x = 1 or x or 3),必须要查出x为null的情况要写成 => x is null or x in (1,3)

> 即：单次查询结果中存在null和非null的数据，能正常查询到非null条件的结果集，但是null结果集的数据会被忽略掉，会缺失null结果集的数据。导致最终的查询结果不完整。

**2. x not in**

(null,1,3)的情况，相当于(x <> null and x <> 1 and x <>3)，会导致一条数据都查不出来

原因是：null不能直接跟任何值(包括null)直接比较，x= null 都是false，x <> null 都是true

**3. not like 'abc%'**

```sql
select * from course where name not like 'abc%';
```

假设想查出 course 表中一条 name 为 null 的数据，上述sql也无法查出到该条数据。因为null无法参与比较。


