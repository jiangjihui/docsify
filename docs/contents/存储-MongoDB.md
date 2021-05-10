## **MongoDB简介**

MongoDB是一个跨平台，面向文档的数据库，提供高性能，高可用性和易于扩展。MongoDB是工作在集合和文档上一种概念。

MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

**数据数**

数据库是一个集合的物理容器。每个数据库获取其自己设定在文件系统上的文件。一个单一的MongoDB服务器通常有多个数据库。

**集合**

集合是一组MongoDB的文件。它与一个RDBMS表是等效的。一个集合存在于数据库中。集合不强制执行模式。集合中的文档可以有不同的字段。通常情况下，在一个集合中的所有文件都是类似或相关目的。

**文档**

文档是一组键值对。文档具有动态模式。动态模式是指，在同一个集合的文件不必具有相同一组集合的文档字段或结构，并且相同的字段可以保持不同类型的数据。



## **Windows下运行**

要在Windows上安装MongoDB，首先从 http://www.mongodb.org/downloads 下载 MongoDB 的最新版本

先执行：

```
C:\Program Files\MongoDB\Server\3.4\bin>mongod.exe --dbpath "D:\Temp\MongoDB"
```

再打开一个窗口执行：

```
mongo.exe
```



## 基础命令

**连接数据库**

**mongo** -umongouser -plxh2081* 172.16.0.56:27017/admin



**创建用户和赋权**

```
use database;

db.createUser({
   user:"cloudSense",
   pwd:"chada123",
   roles: ["readWrite"]
}) 

db.grantRolesToUser( "jjh" , [ { role: "dbOwner", db: "admin" },{ "role": "clusterAdmin", "db": "admin" },
{ "role": "userAdminAnyDatabase", "db": "admin" },
{ "role": "dbAdminAnyDatabase", "db": "admin" },
{ role: "root", db: "admin" } ])
```



**数据库操作**

```
# 创建数据库 （该命令如果数据库不存在，将创建一个新的数据库， 否则将返回现有的数据库）
# 注意：所创建的数据库不存在于列表中。要显示数据库的话，需要至少插入一个文档进去。
# MongoDB的默认数据库是test。 如果没有创建任何数据库，那么集合将被保存在测试数据库。
> use DATABASE_NAME
# 查看当前数据库
> db
# 查询数据库列表
> show dbs
# 删除数据库
> db.dropDatabase()
```



**集合操作**

```
# 创建集合
> db.createCollection("mycollection")
# 查看集合（在MongoDB中并不需要创建集合。 当插入一些文档 MongoDB 会自动创建集合）
> show collections
# 删除集合
> db.COLLECTION_NAME.drop()
```



**文档操作**

```
# 插入文档
> db.COLLECTION_NAME.insert(document)
```

> 示例：这里 mycol 是我们的集合名称，它是在之前的教程中创建。如果集合不存在于数据库中，那么MongoDB创建此集合，然后插入文档进去。
> 在如果我们不指定_id参数插入的文档，那么 MongoDB 将为文档分配一个唯一的ObjectId。
> db.mycol.insert({
>    _id: ObjectId(7df78ad8902c),
>    title: 'MongoDB Overview', 
> })

```
# 查询文档（非结构化的方式显示）
> db.COLLECTION_NAME.find()
# 查询文档（结构化的方式显示）
> db.COLLECTION_NAME.pretty()
```

> | **操作**             | **语法**               | **示例**                                           | **RDBMS等效语句**              |
> | -------------------- | ---------------------- | -------------------------------------------------- | ------------------------------ |
> | Equality             | {<key>:<value>}        | db.mycol.find({"by":"yiibai  tutorials"}).pretty() | where  by = 'yiibai tutorials' |
> | Less  Than           | {<key>:{$lt:<value>}}  | db.mycol.find({"likes":{$lt:50}}).pretty()         | where  likes < 50              |
> | Less  Than Equals    | {<key>:{$lte:<value>}} | db.mycol.find({"likes":{$lte:50}}).pretty()        | where  likes <= 50             |
> | Greater  Than        | {<key>:{$gt:<value>}}  | db.mycol.find({"likes":{$gt:50}}).pretty()         | where  likes > 50              |
> | Greater  Than Equals | {<key>:{$gte:<value>}} | db.mycol.find({"likes":{$gte:50}}).pretty()        | where  likes >= 50             |
> | Not  Equals          | {<key>:{$ne:<value>}}  | db.mycol.find({"likes":{$ne:50}}).pretty()         | where  likes != 50             |



```
# 更新文档
>db.COLLECTION_NAME.update(SELECTIOIN_CRITERIA, UPDATED_DATA)
示例：db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}})
注意：默认情况下，MongoDB将只更新单一文件，更新多，需要一个参数 'multi' 设置为 true。
>db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}},{multi:true})
# 保存文档
> db.COLLECTION_NAME.save({_id:ObjectId(),NEW_DATA})
示例：db.mycol.save({"_id" : ObjectId(5983548781331adf45ec7), "title":"Yiibai Yiibai New Topic", "by":"Yiibai Yiibai"})
# 删除文档
>db.COLLECTION_NAME.remove(DELLETION_CRITTERIA)
示例：下面的例子将删除所有的文件，其标题为 'MongoDB Overview'
>db.mycol.remove({'title':'MongoDB Overview'})
# 删除所有文件（如果没有指定删除条件，则MongoDB将从集合中删除整个文件。这相当于SQL的 truncate 命令。）
>db.mycol.remove()
```





## **查看集合（表）的**[**信息**](https://blog.csdn.net/weixin_41287692/article/details/88418788)

db.集合（表名）.starts()

```
db.vibrationCalcEntity.stats();
```

> 注：默认是bytes单位
>
> | 名词           | 解释                                                         |
> | -------------- | ------------------------------------------------------------ |
> | ns             | 域名空间，由数据库名和集合名构成                             |
> | count          | 集合中包含的文档数量                                         |
> | size           | 集合中所有记录的总大小，这里的值不包括record  head,每个record head是16个字节，但是不包括record's  padding.也不包括所有索引的大小，索引的大小由totalIndexSize的值决定，scale也会影响这个值 |
> | avgObjSize     | 集合中一个对象的平均大小。scale的值会影响这个值              |
> | storageSize    | 分配给集合存储所有文档的存储空间,scale的值会影响这个值，移除或缩小文档，存储空间不会减小 |
> | numExtents     | The total number of  contiguously allocated data file regions. |
> | nindexes       | 这个集合的索引数量，所有集合至少在_id字段有个索引            |
> | lastExtentSize | 最后一个extent分配的大小，scale会影响这个值                  |
> | paddingFactor  | 在插入时间时，在每个文档末尾增加的总空间大小，通过为每个文档在磁盘上分配额外的空间，可以使用每个文档小幅增长时不需要移动文档。 |
> | systemFlags    | 通常值为1                                                    |
> | totalIndexSize | 索引的总大小，scale会影响这个值                              |
> | indexSizes     | 分开显示每个索引的大小，scale会影响这个值                    |





## **MongoDB**[**慢查询**](http://hancang2000.blog.sohu.com/272666566.html)

```
# 查看当前是否开启profile功能（0-不开启/1-记录慢命令 (默认为>100ms) /2-记录所有命令）
> db.getProfilingLevel()

# mongoDB开启慢查询记录
# 第一个参数表示记录级别：0-不开启/1-记录慢命令 (默认为>100ms) /2-记录所有命令
# 第二个参数表示会记录的慢查询时间上限
> db.setProfilingLevel( 1 , 10000); 

# 列出最近5 条执行时间超过1ms的 Profile  记录。
> show profile
```



 

## **查看正在执行的操作**

```
# 用户可以通过 Mongo Shell 连接，并执行 db.currentOp() 命令，能看到数据库当前正在执行的操作
> db.currentOp()
```

> client：请求是由哪个客户端发起的；
> opid：操作的opid，有需要的话，可以通过 db.killOp(opid) 直接干掉的操作；
> secs_running/microsecs_running： 这个值重点关注，代表请求运行的时间，如果这个值特别大，就得注意了，看看请求是否合理；
> query/ns: 这个能看出是对哪个集合正在执行什么操作；
> lock*：还有一些跟锁相关的参数，需要了解可以看官网文档，本文不做详细介绍；





## **解析计划**

mongoDB 执行语句解析：

”queryPlanner”, “executionStats”, 和”allPlansExecution” 分别表示：概要模式，执行状态模式，所有信息模式，默认的是概要模式。

示例：

```
db.featureValueEntity.find().limit(30).explain('executionStats')
```

> COLLSCAN，表示进行了一次全集合扫描
>
> IXSCAN，表示使用了索引扫描
>
> nReturned 显示为 3，表示匹配查询并返回的文档数目为 3





