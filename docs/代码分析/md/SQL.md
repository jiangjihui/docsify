# 易错代码分析



## 简单统计

### join使用

student

| sid  | sname | sex  |
| ---- | ----- | ---- |
| 1    | aa    | m    |
| 2    | bb    | f    |
| 3    | cc    | m    |

course

| cid  | cname  |
| ---- | ------ |
| 1    | java   |
| 2    | python |
| 3    | php    |

score

| id   | student_id | course_id | grade |
| ---- | ---------- | --------- | ----- |
| 1    | 1          | 1         | 80    |
| 2    | 1          | 2         | 88    |
| 3    | 1          | 3         | 90    |
| 4    | 2          | 1         | 80    |
| 1    | 2          | 2         | 88    |
| 6    | 2          | 3         | 70    |
| 7    | 3          | 1         | 75    |

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



