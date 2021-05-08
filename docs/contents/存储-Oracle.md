## **下载安装**

下载地址： http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html

Linux 安装 Oracle http://www.cnblogs.com/asimple41/p/3986018.html

 

 

## **导出/导入数据库**

**普通导入导出**：

以oracle用户登录系统：su - oracle

exp 用户名/密码 file=文件路径 log=日志路径

imp 用户名/密码@网络服务名 file=xxx.dmp full=y;

示例：exp testdb/testdb file=/tmp/expfile.dmp log=/tmp/dblog.log



**数据棒**导入导出：

expdp **testdb/testdb** dumpfile=**acct_loan**.dmpd tables=**acct_loan**

impdp **absimp/absimp** dumpfile=**acct_loan**.dmpd remap_schema=**testdb**:**absimp**

> 例子：重建用户导入数据

```shell
# 导出数据
expdp testdb/testdb dumpfile=testdb_20170811100600.dmpd

# 查看导出文件目录
select * from dba_directories;

# 删除用户及所属表数据
drop user mydb cascade;

# 查看用户会话
select sid,serial# from v$session where username='mydb';

# 删除用户会话
alter system kill session 'SID,SERIAL#';

# 创建用户
create user mydb identified by mydb;

# 赋予权限
grant dba to mydb;

# 为新用户导入数据
impdp mydb/mydb dumpfile=testdb_20170811100600.dmpd remap_schema=testdb:mydb

```





## **基础命令**

**查看所有用户**

```sql
select * from all_users;
```



**查看当前实例名**

```sql
select instance_name from v$instance;
echo $ORACLE_SID;
set ORACLE_SID=xxxx
```

 

**查询默认dump路径**

```sql
select * from dba_directories where directory_name='DATA_PUMP_DIR';
```



**查看各客户连接数目**

```
SELECT username,COUNT(*) from v$session GROUP BY USERNAME;
```



**创建索引**

```
# 单一索引
Create Index <Index-Name> On <Table_Name>(Column_Name);

# 复合索引:在emp表的deptno、job列建立索引
Create Index i_deptno_job on emp(deptno,job);
```



**重启数据库**

```
>lsnrctl stop 关闭监听
>sqlplus "/as sysdba"
>shutdown immediate 关闭数据库
>exit
>lsnrctl start 打开监听
>sqlplus "/as sysdba"
>startup 打开数据库
```



**查看游标数**

```sql
show parameter open_cursors;
```

> **注意：**需要在服务器上使用sqlplus / as sysdba登录数据库查询，不能使用第三方工具运行。





## 控制台乱码设置

```
# 常用中文字符集
set NLS_LANG=SIMPLIFIED CHINESE_CHINA.ZHS16GBK

# 常用unicode字符集
set NLS_LANG=american_america.AL32UTF8

# 查看字符集
select * from v$nls_parameters where parameter='NLS_CHARACTERSET';
```



## 表空间

把oracle数据库看作一个实在房间，表空间可以看作这个房间的空间，是可以自由分配，在这空间里面可以堆放多个箱子（箱子可以看作数据库文件），箱子里面再装物件（物件看作表）。用户指定表空间也就是你希望把属于这个用户的表放在那个房间（表空间）里面。

查看所有表空间的情况

```sql
select * from dba_tablespaces;
```

创建表空间

```sql
create tablespace HRPM0
datafile '/oradata/misdb/HRPM0.DBF' size 5m  autoextend   on next  10m maxsize unlimited;
```

删除表空间

```sql
DROP TABLESPACE data01 INCLUDING CONTENTS AND DATAFILES;
```

修改表空间大小

```sql
alter database datafile '/path/NADDate05.dbf' resize 100M
```

[查看](http://blog.itpub.net/29485627/viewspace-1280367/)该表空间下所有的表

```sql
select table_name from dba_tables where tablespace_name='MY_01';
```



 

## [**表分区**](http://blog.itpub.net/14190034/viewspace-606278/)

当表中的数据量不断增大，查询数据的速度就会变慢，应用程序的性能就会下降，这时就应该考虑对表进行分区。表进行分区后，逻辑上表仍然是一张完整的表，只是将表中的数据在物理上存放到多个表空间(物理文件上)，这样查询数据时，不至于每次都扫描整张表。 

> **注意：**在创建表进行分区时，表空间必须先存在，而且建议将不同的分区放入不同的表空间中。 

**一、范围分区**

这种类型的分区是使用列的一组值，通常将该列成为分区键。

> **示例1：**假设有一个CUSTOMER表，表中有数据200000行，我们将此表通过CUSTOMER_ID进行分区，每个分区存储100000行，我们将每个分区保存到单独的表空间中，这样数据文件就可以跨越多个物理磁盘。下面是创建表和分区的代码，如下： 

```sql
CREATE TABLE CUSTOMER 
( 
  CUSTOMER_ID NUMBER NOT NULL PRIMARY KEY, 
  FIRST_NAME VARCHAR2(30) NOT NULL, 
  STATUS CHAR(1) 
) 
PARTITION BY RANGE (CUSTOMER_ID) 
( 
  PARTITION CUS_PART1 VALUES LESS THAN (100000) TABLESPACE CUS_TS01, 
  PARTITION CUS_PART2 VALUES LESS THAN (200000) TABLESPACE CUS_TS02 
) 
```

> **示例2：**假设有ORDER_ACTIVITIES表，每6个月对订单进行清理，我们可以按月份对表进行分区，分区代码如下：

```sql
CREATE TABLE ORDER_ACTIVITIES 
( 
  ORDER_ID NUMBER(7) NOT NULL, 
  ORDER_DATE DATE, 
  PAID CHAR(1) 
) 
PARTITION BY RANGE (ORDER_DATE) 
( 
  PARTITION ORD_ACT_PART01 VALUES LESS THAN (TO_DATE('01-MAY-2003','DD-MON-YYYY')) TABLESPACE ORD_TS01, 
  PARTITION ORD_ACT_PART02 VALUES LESS THAN (TO_DATE('01-JUN-2003','DD-MON-YYYY')) TABLESPACE ORD_TS02, 
  PARTITION ORD_ACT_PART02 VALUES LESS THAN (TO_DATE('01-JUL-2003','DD-MON-YYYY')) TABLESPACE ORD_TS03 
)
```

**二、列表分区：**

该分区的特点是某列的值只有几个，基于这样的特点我们可以采用列表分区。 

```sql
CREATE TABLE PROBLEM_TICKETS 
( 
  PROBLEM_ID NUMBER(7) NOT NULL PRIMARY KEY, 
  DESCRIPTION VARCHAR2(2000), 
  CUSTOMER_ID NUMBER(7) NOT NULL, 
  DATE_ENTERED DATE NOT NULL, 
  STATUS VARCHAR2(20) 
) 
PARTITION BY LIST (STATUS) 
( 
  PARTITION PROB_ACTIVE      VALUES ('ACTIVE')       TABLESPACE PROB_TS01, 
  PARTITION PROB_INACTIVE    VALUES ('INACTIVE')     TABLESPACE PROB_TS02 
)
```

**三、散列分区：**

这类分区是在列值上使用散列算法，以确定将行放入哪个分区中。当列的值没有合适的条件时，建议使用散列分区。请看下列示例： 

```sql
CREATE TABLE HASH_TABLE 
( 
  COL NUMBER(8), 
  INF VARCHAR2(100) 
) 
PARTITION BY HASH (COL) 
( 
  PARTITION PART01 TABLESPACE HASH_TS01, 
  PARTITION PART02 TABLESPACE HASH_TS02, 
   PARTITION PART03 TABLESPACE HASH_TS03 
)
```

**四、复合范围列表分区：**

这种分区是基于范围分区和列表分区，表首先按某列进行范围分区，然后再按某列进行列表分区，分区之中的分区被称为子分区。 

**五、复合范围散列分区：**

这种分区是基于范围分区和散列分区，表首先按某列进行范围分区，然后再按某列进行散列分区。与上面的定义方式非常的类似，在此不单独举例。

表分区对于**用户**来说是**透明**的，我们在插入数据时Oracle会自动判断插入的数据，然后放入相应的表分区中。但有时我们想单独查询某个分区中的数据时，就必须手工指定分区的名称。



**查看表分区数据**

指定P1表分区查询SALES表信息： 

```
SELECT * FROM SALES PARTITION(P1);
```

指定P1SUB1子分区查询SALES表信息: 

```
SELECT * FROM SALES SUBPARTITION(P1SUB1);
```



## impdp导入dmp[文件](http://blog.csdn.net/zengmingen/article/details/60957942)

impdp命令在cmd下直接用，不必登录oracle。只能导入expdp导出的dmp文件。

expdp导出的时候，需要创建 DIRECTORY

导出什么表空间，导入也要什么表空间。

导出什么用户，导入也要什么用户。

如果没有要新建。

从杭州服务器expdp导出了TOOLBOX用户的数据库dmp文件，要导入宁波本地开发环境中。宁波本地oracle环境是全新的（windows环境）。

**创建表空间**

```
create tablespace TOOLBOX
logging
datafile 'C:\oraclexe\app\oracle\oradata\XE\TOOLBOX.dbf'
size 50m
autoextend on
next 32m maxsize unlimited
extent management local;
```

 

**创建用户，赋予权限**

```
create user TOOLBOX identified by 123456;
alter user TOOLBOX default tablespace TOOLBOX;
grant CREATE ANY DIRECTORY,create session,create table,create view,unlimited tablespace to TOOLBOX;
```

登录ToolBox用户

 

**创建DIRECTORY**

```
CREATE OR REPLACE DIRECTORY DMPDIR AS 'D:\Temp\orcl\product\11.1.0\db_1\admin\sample\bdump'; 
```

目录创建以后，就可以把读写权限授予特定用户，具体语法如下:

```
GRANT READ[,WRITE] ON DIRECTORY directory TO username;
```

 

**编写导入impdp语句**

```
impdp toolbox/123456 DIRECTORY=DMPDIR DUMPFILE=hz_toolbox_20160613.dmp full=y
```

> **注意：**DUMPFILE参数不可以包含路径，要么是默认路径，要么指定directory参数来指定自己的dump文件路径。



**导入错误处理**

> 连接到: Personal Oracle Database 11g Release 11.2.0.1.0 - 64bit Production
> With the Partitioning, OLAP, Data Mining and Real Application Testing options
> ORA-39001: 参数值无效
> ORA-39000: 转储文件说明错误
> ORA-39088: 文件名不能包含路径说明

解：出现上述错误是因为导入的文件的需要放在oracle的默认的导入导出目录下面。如果不想放在oracle默认的导入导出目录下面的话，可以创建一个directory进行导入导出操作，在导入的时候只需指定directory参数（directory=directoryname）即可。

```
CREATE [OR REPLACE] DIRECTORY directoryname AS 'pathname';
create or replace directory dump_dir as 'D:\dump\dir'
```

这样把目录d:\dump\dir设置成dump_dir代表的directory

 

> ORA-14460只能指定一个COMPRESS或NOCOMPRESS子句

解：需要在这个导入语句中加入transform=segment_attributes:n参数。该参数可与忽略expdp导出时附带的相关表空间和存储子句约束。

**示例：**

```
impdp ilanni/numen@192.168.24.249:/orcl **transform=segment_attributes:n** directory=wpdp_dir remap_schema=numen: ilanni dumpfile=140109.dmp logfile=1401092.log
```

 

> ORA-02374: conversion error loading table "SXCMS"."T_FACT_LOAN_M"
> ORA-12899: value too large for column REGISTERADD (actual: 82, maximum: 80)
> ORA-02372: data for row: REGISTERADD : 0X'D3C0B4A8C7F8D0C7B9E2B4F3B5C0393939BAC531B4B1A3A8D6'

解：字符集差异导致.中文在UTF-8里占3个字节，ZHS16GBK里占2个字节。

查看两个库的字符集：select * from nls_database_parameters where parameter ='NLS_CHARACTERSET';

第一种方法可以通过：修改字符集方法在这个网址：http://blog.csdn.net/lyb3290/article/details/53758884，

还有一种方法是创建新的不同字符集的实例再导入即可。再导入的时候在cmd里面使用set ORACLE_SID=PROJECT 指定新创建的实例即可。

