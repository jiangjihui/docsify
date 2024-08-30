## [**概述**](https://segmentfault.com/a/1190000014567118?utm_source=feed-content)

Spring Cloud，它是在 Spring Boot 的基础上，增加了一堆微服务相关的规范，并对应用上下文（Application Context）进行了功能增强。

Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

微服务是可以独立部署、水平扩展、独立访问（或者有独立的数据库）的服务单元，springcloud就是这些微服务的大管家，采用了微服务这种架构之后，项目的数量会非常多，springcloud做为大管家需要管理好这些微服务，自然需要很多小弟来帮忙。

 

 

## Spring Cloud的核心功能

- 分布式/版本化配置
- 服务注册和发现
- 路由
- 服务和服务之间的调用
- 负载均衡
- 断路器
- 分布式消息传递

 

 

## 运行流程

1、请求统一通过API网关（Zuul）来访问内部服务.

2、网关接收到请求后，从注册中心（Eureka）获取可用服务

3、由Ribbon进行均衡负载后，分发到后端具体实例

4、微服务之间通过Feign进行通信处理业务

5、Hystrix负责处理服务超时熔断

6、Turbine监控服务间的调用和熔断相关指标

 

 

## Spring Cloud [体系介绍](https://segmentfault.com/a/1190000014567118?utm_source=feed-content)

上图只是Spring Cloud体系的一部分，Spring Cloud共集成了19个子项目，里面都包含一个或者多个第三方的组件或者框架！

 1、Spring Cloud **Config** 配置中心，利用git集中管理程序的配置。

 2、Spring Cloud **Netflix** 集成众多Netflix的开源软件

 3、Spring Cloud **Bus** 消息总线，利用分布式消息将服务和服务实例连接在一起，用于在一个集群中传播状态的变化

 4、Spring Cloud **for Cloud Foundry** 利用Pivotal Cloudfoundry集成你的应用程序

 5、Spring Cloud **Cloud Foundry Service Broker** 为建立管理云托管服务的服务代理提供了一个起点。

 6、Spring Cloud **Cluster** 基于Zookeeper, Redis, Hazelcast, Consul实现的领导选举和平民状态模式的抽象和实现。

 7、Spring Cloud **Consul** 基于Hashicorp Consul实现的服务发现和配置管理。

 8、Spring Cloud **Security** 在Zuul代理中为OAuth2 rest客户端和认证头转发提供负载均衡

 9、Spring Cloud **Sleuth SpringCloud**应用的分布式追踪系统，和Zipkin，HTrace，ELK兼容。

10、Spring Cloud **Data Flow** 一个云本地程序和操作模型，组成数据微服务在一个结构化的平台上。

11、Spring Cloud **Stream** 基于Redis,Rabbit,Kafka实现的消息微服务，简单声明模型用以在Spring Cloud应用中收发消息。

12、Spring Cloud **Stream App Starters** 基于Spring Boot为外部系统提供spring的集成

13、Spring Cloud **Task** 短生命周期的微服务，为SpringBooot应用简单声明添加功能和非功能特性。

14、Spring Cloud **Task App Starters**

15、Spring Cloud **Zookeeper** 服务发现和配置管理基于Apache Zookeeper。

16、Spring Cloud for **Amazon Web Services** 快速和亚马逊网络服务集成。

17、Spring Cloud **Connectors** 便于PaaS应用在各种平台上连接到后端像数据库和消息经纪服务。

18、Spring Cloud **Starters** （项目已经终止并且在Angel.SR2后的版本和其他项目合并）

19、Spring Cloud **CLI** 插件用Groovy快速的创建Spring Cloud组件应用。

当然这个数量还在一直增加…



## **子项目**

- 分布式/版本化配置：Spring Cloud Config
- 服务注册和发现：Netflix Eureka 或者 Spring     Cloud Eureka（对前者的二次封装）
- 路由：Spring Cloud Zuul，基于 Netflix Zuul
- service-to-service调用：Spring Cloud Feign
- 负载均衡：Spring Cloud Ribbon 基于Netflix Ribbon 实现
- 断路器：Spring Cloud Hystrix
- 分布式消息传递：Spring Cloud Bus

