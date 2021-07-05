# RabbitMQ

许多新手在刚接触RabbitMQ的时候，会被各种名词弄晕，包括ConnectionFactory 、Connection 、Channel、Exchange、Queue、RoutingKey、BindingKey等等，现在我言简意赅的描绘一下这些名词到底是啥概念

> 参考：
>
> [RabbitMQ Exchange Queue RoutingKey BindingKey解析](https://blog.csdn.net/ad132126/article/details/83539213)
>
> [springboot集成RabbitMQ](https://blog.csdn.net/qq_38455201/article/details/80308771)



## 构成

### ConnectionFactory

首先，想要让队列不在本地运行，而在网络中运行，肯定会有连接这个概念，所以就会有Connection，我们发一条消息连接一次，这样很显然是浪费资源的，建立连接的过程也很耗时，所以我们就会做一个东西让他来管理连接，当我用的时候，直接从里边拿出来已经建立好的连接发信息，那么ConnectionFactory应运而生。



### Channel

接下来，当程序开发时，可能不止用到一个队列，可能有订单的队列、消息的队列、任务的队列等等，那么就需要给不同的queue发信息，那么和每一个队列连接的这个概念，就叫Channel



### Exchange

再往下来，当我们开发的时候还有时候会用到这样一种功能，就是当我发送一条消息，需要让几个queue都收到，那么怎么解决这个问题呢，难道我要给每一个queue发送一次消息？那岂不是浪费带宽又浪费资源，我们能想到什么办法呢，当然是我们发送给RabbitMQ服务器一次，然后让RabbitMQ服务器自己解析需要给哪个Queue发，那么Exchange就是干这件事的



### BindingKey

BindingKey是Exchange和Queue绑定的规则描述，这个描述用来解析当Exchange接收到消息时，Exchange接收到的消息会带有RoutingKey这个字段，Exchange就是根据这个RoutingKey和当前Exchange所有绑定的BindingKey做匹配，如果满足要求，就往BindingKey所绑定的Queue发送消息，这样我们就解决了我们向RabbitMQ发送一次消息，可以分发到不同的Queue的过程



### 名词解释

- ConnectionFactory：与RabbitMQ服务器连接的管理器
- Connection：与RabbitMQ服务器的连接
- Channel：与Exchange的连接
- Exchange：接受消息提供者（生产者）的消息，并根据消息的RoutingKey和Exchange绑定的BindingKey分配消息
- Queue：存储消息接收者（消费者）的消息
- RoutingKey：指定当前消息被谁接受
- BindingKey：指定当前Exchange下，什么样的RoutingKey会被下派到当前绑定的Queue中

![img](../_images/95517-20170109162632728-1069401237.png)





**生产者 **关心exchange、queue、routingKey

> 在direct类型的exchange中，只有这两个routingkey完全相同，exchange才会选择对应的binging进行消息路由。

**消费者** 关心queue



