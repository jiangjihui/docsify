## **什么是**[**Redis**](https://juejin.im/post/5ad6e4066fb9a028d82c4b66)

Redis 是一个使用 C 语言写成的，开源的 key-value 数据库。。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。目前，Vmware在资助着redis项目的开发和维护。

  



## **Redis常见数据结构**[**使用场景**](https://mp.weixin.qq.com/s/cGD-sKKuMH-Y6LUD5JTv1w)

1.**String**

常用命令：set,get,decr,incr,mget 等。

String数据结构是简单的key-value类型，value其实不仅可以是String，也可以是数字。

**用途：**

常规key-value缓存应用；

常规计数：微博数，粉丝数等。

2.**Hash**

常用命令： hget,hset,hgetall 等。

Hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。 比如我们可以Hash数据结构来存储用户信息，商品信息等等。

**举例：** 电商网站项目的首页使用redis的hash数据结构进行缓存，因为一个网站的首页访问量是最大的，所以通常网站的首页可以通过redis缓存来提高性能和并发量。我用**jedis客户端**来连接和操作我搭建的redis集群或者单机redis，利用jedis可以很容易的对redis进行相关操作，总的来说从搭一个简单的[集群](https://juejin.im/post/5ad54d76f265da23970759d3)到实现redis作为缓存的整个步骤不难。

3.**List**

常用命令:lpush,rpush,lpop,rpop,lrange等

list就是链表，Redis list的应用场景非常多，也是Redis最重要的数据结构之一，比如微博的关注列表，粉丝列表，最新消息排行等功能都可以用Redis的list结构来实现。

Redis list的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

4.**Set**

常用命令：sadd,spop,smembers,sunion 等

set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以自动排重的。 当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

**举例：**在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能。可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。 

5.**Sorted Set**

常用命令： zadd,zrange,zrem,zcard等

和set相比，sorted set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列。

**举例：** 在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息，适合使用Redis中的SortedSet结构进行存储。

 



## **Redis命令**

**登录**

```
redis-cli -h host -p port -a password
```

 

**查看**

```
# 查看所有keys
# 如果需要模糊查询可在*前加上需要匹配的关键字比如：keys spring*
keys *
```



**删除**

```
# 清空当前数据库中的所有 key
flushall
```



 

## **Redis安全**

默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。

通过以下命令查看是否设置了密码验证：

```
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) ""
```

通过以下命令来修改该参数：

```
127.0.0.1:6379> CONFIG set requirepass "runoob"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "runoob"
```

设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。

```
127.0.0.1:6379> AUTH "runoob"
OK
127.0.0.1:6379> SET mykey "Test value"
OK
127.0.0.1:6379> GET mykey
"Test value"
```





## Redis相比memcached有哪些区别

**数据类型丰富：**

Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。memcached支持简单的数据类型，String。

**速度快：**

redis的比memcached速度快很多

**持久化：**

Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而Memecache把数据全部存在内存之中

**数据备份：**

Redis支持数据的备份，即master-slave模式的数据备份。

**单线程：**

Memcached是多线程，非阻塞IO复用的网络模型；Redis使用单线程的IO复用模型。


 
 ## IO模型
 
 redis是基于多路复用的高性能 I/O 模型，Linux 中的 IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的 select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听套接字和已连接套接字。**内核会一直监听这些套接字上的连接请求或数据请求**。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。
 
 - Redis通信采用非阻塞IO， 内部实现采用epolll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间。
 - 单机Redis采用单进程、单线程、单实例，避免了不必要的上下文切换和竞争条件。
 
 
 
 ### 多路复用I/O
 
 - **select：**能打开的文件描述符个数有限（最多1024个），如果有1K个请求，用户进程每次都要把1K个文件描述符发送给内核，内核在内部轮询后将可读描述符返回，用户进程再依次读取。因为文件描述符（fd）相关数据需要在用户态和内核态之间拷来拷去，所以性能还是比较低。
 
 - **poll：**可打开的文件描述符数量提高，因为用链表存储，但性能仍然不够，和selector一样数据需要在用户态和内核态之间拷来拷去。
 
 - **epoll（Linux下多为该技术）：**用户态和内核态之间不用文件描述符（fd）的拷贝，而是通过mmap技术开辟共享空间，所有fd用红黑树存储，有返回结果的fd放在链表中，用户进程通过链表读取返回结果，伪异步I/O，性能较高。epoll分为水平触发和边缘出发两种模式，ET是边缘触发，LT是水平触发，一个表示只有在变化的边际触发，一个表示在某个阶段都会触发。
 
 
 
 ### 场景举例
 
 > epoll所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右。
 
 设想一下如下场景：有100万个客户端同时与一个服务器进程保持着TCP连接。而每一时刻，通常只有几百上千个TCP连接是活跃的(事实上大部分场景都是这种情况)。如何实现这样的高并发？
 
 - **select/poll**
 
   在select/poll时代，服务器进程每次都把这100万个连接告诉操作系统(从用户态复制句柄数据结构到内核态)，让操作系统内核去查询这些套接字上是否有事件发生，轮询完后，再将句柄数据复制到用户态，让服务器应用程序轮询处理已发生的网络事件，这一过程资源消耗较大，因此，select/poll一般只能处理几千的并发连接。
 
   如果没有I/O事件产生，我们的程序就会阻塞在select处。但是依然有个问题，我们从select那里仅仅知道了，有I/O事件发生了，但却并不知道是那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。
 
   但是使用select，我们有O(n)的无差别轮询复杂度，同时处理的流越多，每一次无差别轮询时间就越长。
 
   
 
 - **epoll**
 
   epoll的设计和实现与select完全不同。epoll通过在Linux内核中申请一个简易的文件系统(文件系统一般用什么数据结构实现？B+树)。把原先的select/poll调用分成了3个部分：
 
     1. 调用epoll_create()建立一个epoll对象(在epoll文件系统中为这个句柄对象分配资源)
     2. 调用epoll_ctl向epoll对象中添加这100万个连接的套接字
     3. 调用epoll_wait收集发生的事件的连接
 
     如此一来，要实现上面说是的场景，只需要在进程启动时建立一个epoll对象，然后在需要的时候向这个epoll对象中添加或者删除连接。同时，epoll_wait的效率也非常高，因为调用epoll_wait时，并没有一股脑的向操作系统复制这100万个连接的句柄数据，内核也不需要去遍历全部的连接。这是由于我们在调用epoll_create时，内核除了帮我们在epoll文件系统里建了个file结点，在内核cache里建了个红黑树用于存储以后epoll_ctl传来的socket外，还会再建立一个list链表，用于存储准备就绪的事件，当epoll_wait调用时，仅仅观察这个list链表里有没有数据即可。有数据就返回，没有数据就sleep，等到timeout时间到后即使链表没数据也返回。所以，epoll_wait非常高效。
 
 
 
 > **AIO：**异步I/O，性能最高，但是使用非常复杂，不是很常用（windows系统中多见，Java中有AIO，API在Linux上还是用epoll实现）
 >
 > 参考来源
 > [Redis02-Redis高性能与epoll](https://segmentfault.com/a/1190000021985202)
 > [redis epoll 原理梗概](https://www.cnblogs.com/peterkang202/p/10452251.html)
 

 

## **Redis实现主从同步**

Master&Slave也就是我们所说的[主从复制](https://blog.csdn.net/zhangguanghui002/article/details/78524533)，主机数据更新后根据配置和策略，自动同步到备机，Master以写为主，Slave以读为主。

**作用：**

​    1、读写分离；

​    2、容灾恢复。

**怎么玩：**

​    1、配从（库）不配主（库）；

​    2、从库配置：slaveof [主库IP] [主库端口]；

​    补充：每次slave与master断开后，都需要重新连接，除非你配置进redis.conf文件;

​    键入info replication 可以查看redis主从信息。

**薪火相传**

![image](../_images/fe8c8efb-e076-4590-b7f2-f319e7fa69f3.png)

上一个Slave可以是下一个Slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个slave的Master，如此可以有效减轻Master的写压力。如果slave中途变更转向，会清除之前的数据，重新建立最新的。

**反客为主**

当Master挂掉后，Slave可键入命令 slaveof no one使当前redis停止与其他Master redis数据同步，转成Master redis。

**复制原理**

​    1、Slave启动成功连接到master后会发送一个sync命令；

​    2、Master接到命令启动后的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，以完成一次完全同步；

​    3、全量复制：而slave服务在数据库文件数据后，将其存盘并加载到内存中；

​    4、增量复制：Master继续将新的所有收集到的修改命令依次传给slave，完成同步；

​    5、但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。

**哨兵模式（sentinel）**

反客为主的自动版，能够后台监控Master库是否故障，如果故障了根据投票数自动将slave库转换为主库。一组sentinel能同时监控多个Master。

​    1、在Master对应redis.conf同目录下新建sentinel.conf文件，名字绝对不能错；

​    2、配置哨兵，在sentinel.conf文件中填入内容：

​       sentinel monitor 被监控数据库名字（自己起名字） ip port 1

​       说明：上面最后一个数字1，表示主机挂掉后slave投票看让谁接替成为主机，得票数多少后成为主机。

​    3、启动哨兵模式：

​       命令键入：redis-sentinel /myredis/sentinel.conf

​       注：上述sentinel.conf路径按各自实际情况配置

**复制的缺点**

延时，由于所有的写操作都是在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使得这个问题更加严重。



 

## **如何保证Redis中的数据都是热点数据**

MySQL里有2000w数据，Redis中只存20w的数据，如何保证Redis中的数据都是热点数据（redis有哪些数据淘汰策略？？？）

相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略（回收策略）。

**redis 提供 6种数据淘汰策略：**

1. **volatile-lru**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. **allkeys-lru**：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
6. **no-enviction**（驱逐）：禁止驱逐数据

 

 

## **Redis的并发竞争问题如何解决**

Redis为单进程单线程模式，采用队列模式将并发访问变为串行访问。Redis本身没有锁的概念，Redis对于多个客户端连接并不存在竞争，但是在Jedis客户端对Redis进行并发访问时会发生连接超时、数据转换错误、阻塞、客户端关闭连接等问题，这些问题均是由于客户端连接混乱造成。对此有2种解决方法：

1.客户端角度，为保证每个客户端间正常有序与Redis进行通信，对连接进行池化，同时对客户端读写Redis操作采用内部锁synchronized。

2.服务器角度，利用setnx实现锁。

注：对于第一种，需要应用程序自己处理资源的同步，可以使用的方法比较通俗，可以使用synchronized也可以使用lock；第二种需要用到Redis的setnx命令，但是需要注意一些问题。

 

 

## **Redis与消息队列**

不要使用redis去做消息队列，这不是redis的设计目标。但实在太多人使用redis去做去消息队列，redis的作者看不下去，另外基于redis的核心代码，另外实现了一个消息队列**disque**： antirez/disque:https://github.com/antirez/disque部署、协议等方面都跟redis非常类似，并且支持集群，延迟消息等等。

在做网站过程接触比较多的还是使用redis做缓存，比如秒杀系统，首页缓存等等。



## **Redis作为队列使用**

```java
// 加入队列
redisTemplate.opsForList().rightPush(K key, V value)
// 从队列中拿出
redisTemplate.opsForList().leftPop(K key)
```

 



## **Redis常见性能问题和解决方案**

1. Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件
2. 如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次
3. 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内
4. 尽量避免在压力很大的主库上增加从库

 

 

## **Redis持久化**

Redis常用的[持久化方案](https://www.pdai.tech/md/db/nosql-redis/db-redis-x-rdb-aof.html)有RDB和AOF。

### RDB

RDB持久化是把当前进程数据生成快照保存到磁盘上的过程，由于是某一时刻的快照，那么快照中的值要早于或者等于内存中的值。

触发rdb持久化的方式有2种，分别是**手动触发**和**自动触发**。

**实现方式**

RDB中的核心思路是Copy-on-Write，来保证在进行快照操作的这段时间，需要压缩写入磁盘上的数据在内存中不会发生变化。在正常的快照操作中，一方面Redis主进程会**fork**一个新的快照**进程**专门来做这个事情，这样保证了Redis服务不会停止对客户端包括写请求在内的任何响应。另一方面这段时间发生的数据变化会以副本的方式存放在另一个新的内存区域，待快照操作结束后才会同步到原来的内存区域。

**优点**

- **文件体积小** RDB文件是某个时间节点的快照，默认使用LZF算法进行压缩，压缩后的文件体积远远小于内存大小，适用于备份、全量复制等场景；
- **恢复快** Redis加载RDB文件恢复数据要远远快于AOF方式；

**缺点**

- **实时性差** RDB方式实时性不够，无法做到秒级的持久化；
- **有阻塞** 每次调用bgsave都需要fork子进程，fork子进程属于重量级操作，频繁执行成本较高；
- **文件可读性差** RDB文件是二进制的，没有可读性，AOF文件在了解其结构的情况下可以手动修改或者补全；
- 版本兼容RDB文件问题；



针对RDB不适合实时持久化的问题，Redis提供了AOF持久化方式来解决



### AOF

AOF是“写后”日志，Redis先执行命令，把数据写入内存，然后才记录日志。日志里记录的是Redis收到的每一条命令，这些命令是以文本形式保存。（PS: 大多数的数据库采用的是写前日志（WAL），例如MySQL，通过写前日志和两阶段提交，实现数据和逻辑的一致性。）

**为什么采用写后日志**？

Redis要求高性能，采用写日志有两方面好处：

- 避免额外的检查开销

  Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查。所以，如果先记日志再执行命令的话，日志中就有可能记录了错误的命令，Redis 在使用日志恢复数据时，就可能会出错。

- 不会阻塞当前的写操作

- 文件可读性性高

**实现AOF**

AOF日志记录Redis的每个写命令，步骤分为：命令追加（append）、文件写入（write）和文件同步（sync）。

- **命令追加** 当AOF持久化功能打开了，服务器在执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器的 aof_buf 缓冲区。

- **文件写入和同步** 关于何时将 aof_buf 缓冲区的内容写入AOF文件中，Redis提供了三种写回策略：

  `Always`，同步写回：每个写命令执行完，立马同步地将日志写回磁盘；

  `Everysec`，每秒写回：每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；(默认策略)

  `No`，操作系统控制的写回：每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。

**AOF重写**

AOF会记录每个写命令到AOF文件，随着时间越来越长，AOF文件会变得越来越大。如果不加以控制，会对Redis服务器，甚至对操作系统造成影响，而且AOF文件越大，数据恢复也越慢。为了解决AOF文件体积膨胀的问题，Redis提供AOF文件重写机制来对AOF文件进行“瘦身”。

Redis通过创建一个新的AOF文件来替换现有的AOF，新旧两个AOF文件保存的数据相同，但新AOF文件没有了冗余命令。

- 主线程fork出子进程重写aof日志
- 子进程重写日志完成后，主线程追加aof日志缓冲
- 替换日志文件

**AOF缺点**

- 对于相同的数据集来说，AOF文件的**体积**通常要**大**于RDB文件的体积（保存的是命令列表，相较于直接保存数据更占用空间）
- 根据所使用的fsync策略，AOF的持久化**速度**可能会**慢**于RDB
- 即使你配置的 AOF 刷盘策略是 appendfsync everysec，也依旧会有阻塞主线程的风险。产生这个问题的重点在于，磁盘 IO 负载过高导致 fynsc 阻塞，进而导致主线程写 AOF page cache 也发生阻塞。所以，你一定要保证磁盘有充足的 IO 资源，[避免这个问题](https://mp.weixin.qq.com/s/UqHTJUbefC7WSm2Rb5xCvQ)。无论如何，即使 AOF 配置为每秒刷盘，在发生极端情况时，AOF 丢失的数据其实是 2 秒。



### RDB和AOF混合方式（4.0版本)

>Redis 4.0 中提出了一个**混合使用 AOF 日志和内存快照**的方法。简单来说，内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作。

这样一来，快照不用很频繁地执行，这就避免了频繁 fork 对主线程的影响。而且，AOF 日志也只用记录两次快照间的操作，也就是说，不需要记录所有操作了，因此，就不会出现文件过大的情况了，也可以避免重写开销。

这个方法既能享受到 RDB 文件快速恢复的好处，又能享受到 AOF 只记录操作命令的简单优势, 实际环境中用的很多。







## **windows下启动Radis**

**标准方式**

1.启动服务（命令行输入）：

```
D:\redis\redis-server.exe D:\redis\redis.windows.conf
```

2.打开终端（radis目录命令行输入）：

```
redis-cli -h 127.0.0.1 -p 6379
```

**简易方式**

1.启动服务：运行redis目录下的redis-server.exe即可

2.终端访问：运行redis目录下的redis-cli.exe即可

**将redis变成windows服务**

使用管理员身份运行powershell，进入redis文件夹下键入以下命令之后即可进入服务列表看到新加的redis服务：

```
./redis-server --service-install redis.windows-service.conf --loglevel verbose
```



 

## **Linux下安装启动Radis**

[**安装**](https://www.cnblogs.com/lauhp/p/8487029.html)**：**

1.获取redis资源

```
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar xzvf redis-4.0.8.tar.gz
```

2.安装

```
cd redis-4.0.8
make
cd src
make install PREFIX=/usr/local/redis
```

3.移动配置文件到安装目录下

```
cd ../
mkdir /usr/local/redis/etc
mv redis.conf /usr/local/redis/etc
```

4.配置redis为后台启动

```
# 将daemonize no 改成daemonize yes
vi /usr/local/redis/etc/redis.conf
```

5.将redis加入到开机启动

```
# 在里面添加内容：/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf (意思就是开机调用这段开启redis的命令)
vi /etc/rc.local
```

**启动：**

1.开启redis

```
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf 
# 或者
redis-server /usr/local/redis/etc/redis.conf
```

2.停止

```
pkill redis
```

**卸载：**

```
# 删除安装目录
rm -rf /usr/local/redis

# 删除所有redis相关命令脚本
rm -rf /usr/bin/redis-*

# 删除redis解压文件夹
rm -rf /root/download/redis-4.0.4
```



 

## **CacheCloud**

CacheCloud提供一个Redis[云管理平台](https://hub.docker.com/r/trumandu/cachecloud/)：实现多种类型(Redis Standalone、Redis Sentinel、Redis Cluster)自动部署、解决Redis实例碎片化现象、提供完善统计、监控、运维功能、减少运维成本和误操作，提高机器的利用率，提供灵活的伸缩性，提供方便的接入客户端。

```
#docker使用CacheCloud
docker pull trumandu/cachecloud:1.2
```



 

## **SpringBoot使用**[**Redis**](https://juejin.im/post/5bbf077d5188255c393f8c90)

Spring为我们提供了几个注解来支持Spring Cache。其核心主要是[@Cacheable](https://www.cnblogs.com/fashflying/p/6908028.html)和@CacheEvict。

使用@Cacheable标记的方法在执行后Spring Cache将缓存其返回结果

使用@CacheEvict标记的方法会在方法执行前或者执行后移除Spring Cache中的某些元素。

 

 

## Lettuce（Redis客户端）

Lettuce和Jedis的都是连接Redis Server的客户端程序。Jedis在实现上是直连redis server，多线程环境下非线程安全，除非使用连接池，为每个Jedis实例增加物理连接。[Lettuce](https://blog.csdn.net/winter_chen001/article/details/80614331)基于Netty的连接实例（StatefulRedisConnection），可以在多个线程间并发访问，且**线程安全**，满足多线程环境下的并发访问，同时它是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。

 

## **Redis**[**插入大量数据**](http://www.redis.cn/topics/mass-insert.html)

使用[Shell+Redis](https://www.cnblogs.com/ivictor/p/5446503.html)

1. 新建一个文本文件，包含redis命令

   SET Key0 Value0
   SET Key1 Value1
    …

2. 将这些命令转化成Redis Protocol**（因为Redis管道功能支持的是Redis Protocol，而不是直接的Redis命令）**

   ```
   #!/bin/bash
   while read CMD; do
   # each command begins with *{number arguments in command}\r\n
   XS=($CMD); printf "*${#XS[@]}\r\n"
   # for each argument, we append ${length}\r\n{argument}\r\n
   for X in $CMD; do printf "\$${#X}\r\n$X\r\n"; done
   done < redis.log
   ```

	执行文件
   ```
   ./convert.sh > redis_data.txt
   ```

   

3. 利用管道插入

   ```
   cat redis_data.txt | redis-cli --pipe
   ```

   



## **MongoDB与Redis比较**

| **比较指标** | **MongoDB(v2.4.9)**                                          | **Redis(v2.4.17)**                                           | **比较说明**                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 实现语言     | c++                                                          | c/c++                                                        | -                                                            |
| 协议         | BSON,自定义二进制                                            | 类telnet                                                     | -                                                            |
| 性能         | 依赖内存,TPS较高                                             | 依赖内存,TPS非常高                                           | Redis优于MongoDB                                             |
| 可操作性     | 丰富的数据表达,索引;最类似于关系型数据库,支持丰富的查询语句  | 数据丰富,较少的IO                                            | MongoDB优于Redis                                             |
| 内存及存储   | 适合大数据量存储,依赖系统虚拟内存,采用镜像文件存储;内存占用率比较高,官方建议独立部署在64位系统 | Redis2.0后支持虚拟内存特性(VM)  突破物理内存限制;数据可以设置时效性,类似于memcache | 不同的应用场景,各有千秋                                      |
| 可用性       | 支持master-slave,replicatset(内部采用paxos选举算法,自动故障恢复),auto  sharding机制,对客户端屏蔽了故障转移和切片机制 | 依赖客户端来实现分布式读写;主从复制时,每次从节点重新连接主节点都要依赖整个快照,无增量复制;不支持auto  sharding,需要依赖程序设定一致性hash机制 | MongoDB优于Redis；单点问题上,MongoDB应用简单,相对用户透明,Redis比较复杂,需要客户端主动解决.(MongoDB一般使用replicasets和sharding相结合,replicasets侧重高可用性以及高可靠,sharding侧重性能,水平扩展) |
| 可靠性       | 从1.8版本后,采用binlog方式(类似Mysql)  支持持久化            | 依赖快照进行持久化;AOF增强可靠性;增强性的同时,影响访问性能   |                                                              |
| 一致性       | 不支持事务,靠客户端保证                                      | 支持事务,比较脆,仅能保证事务中的操作按顺序执行               | Redis优于MongoDB                                             |
| 数据分析     | 内置数据分析功能(mapreduce)                                  | 不支持                                                       | MongoDB优于Redis                                             |
| 应用场景     | 海量数据的访问效率提升                                       | 较小数据量的性能和运算                                       | MongoDB优于Redis                                             |

> **注：**MongoDB建议集群部署，更多的考虑到集群方案，Redis更偏重于进程顺序写入，虽然支持集群，也仅限于主-从模式。