> 参考：https://www.bilibili.com/video/av89898642

## 介绍

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

## 主要功能

**服务限流降级** ：默认支持 WebServlet、WebFlux， OpenFeign、RestTemplate、Spring Cloud Gateway， Zuul， Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。

**服务注册与发现** ：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。

**分布式配置管理** ：支持分布式系统中的外部化配置，配置更改时自动刷新。

**消息驱动能力** ：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。

**分布式事务** ：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。

**阿里云对象存储** ：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。

**分布式任务调度** ：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有Worker（schedulerx-client）上执行。

**阿里云短信服务** ：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## 组件

**Sentinel** ：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

**Nacos** ：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

**RocketMQ** ：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

**Dubbo** ：Apache Dubbo? 是一款高性能 Java RPC 框架。

**Seata** ：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

**Alibaba Cloud ACM** ：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。

**Alibaba Cloud OSS** : 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

**Alibaba Cloud SchedulerX** : 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

**Alibaba Cloud SMS** : 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## Nacos Discovery--服务治理

### 简介

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

从上面的介绍就可以看出， nacos的作用就是一个注册中心 ，用来管理注册上来的各个微服务。

> nacos解决微服务之间互相调用的问题。（IP地址+端口）

### 使用

**1** **在pom.xml中添加nacos的依赖**

```xml
<!--nacos客户端-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**2** **在主类上添加@EnableDiscoveryElient注解**

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ProductApplication(){}
```

**3** **在application.yml中添加nacos服务的地址**

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr:127.0.0.1:8848
```

4启动服务,观察nacos的控制面板中是否有注册.上来的商品微服务

### 结合Ribbon进行负载均衡

1 在RestTemplate 的生成方法上添加@loadBalanced注解

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

2 修改服务调用方法

使用服务名替换url作为restTemplate的请求地址。

3 (可选)我们可以通过修改配置来调整Ribbon的负载均衡策略，具体代码如下
#调用的提供者的名称

```yaml
service-product:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 结合Fegin实现服务调用

作用：替代RestTemplate调用服务的写法，实现代码风格统一（伪Http客户端，使得调用远程服务跟调用本地服务一样简单）

1 加入Fegin的依赖

```xml
<!--fegin组件-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2 在主类上添加Fegin的注解

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients//开启Fegin
public class orderApplication{}
```

3 创建一个service,并使用Fegin实现微服务调用

```java
@FeignClient("service-product")//声明调用的提供者的name
public interface ProductService{
    //指定调用提供者的哪个方法
    //@FeignClient+@GetMapping就是一个完整的请求路径http://service-product/product/{pid}
    @GetMapping(value="/product/{pid}")
    Product findByPid(@PathVariable("pid") Integer pid);
}
```

4 修改controller使用创建的service进行服务调用

## Nacos Config--服务配置

首先我们来看一下,微服务架构下关于配置文件的一些问题：

1. 配置文件相对分散。在一个微服务架构下，配置文件会随着微服务的增多变的越来越多，而且分散在各个微服务中，不好统一配置和管理。

2. 配置文件无法区分环境。微服务项目可能会有多个环境，例如：测试环境、预发布环境、生产环境。每一个环境所使用的配置理论上都是不同的，一旦需要修改，就需要我们去各个微服务下手动维护，这比较困难。

3. 配置文件无法实时更新。我们修改了配置文件之后，必须重新启动微服务才能使配置生效，这对一个正在运行的项目来说是非常不友好的。基于上面这些问题，我们就需要 配置中心 的加入来解决这些问题。

配置中心的思路是：

首先把项目中各种配置全部都放到一个集中的地方进行统一管理，并提供一套标准的接口。当各个服务需要获取配置的时候，就来配置中心的接口拉取自己的配置。当配置中心中的各种参数有更新的时候，也能通知到各个服务实时的过来同步最新的信息，使之动态更新。

### Nacos Config入门

使用nacos 作为配置中心，其实就是将nacos 当做一个服务端，将各个微服务看成是客户端，我们将各个微服务的配置文件统一存放在nacos 上，然后各个微服务从nacos 上拉取配置即可。

**1** **搭建nacos环境【使用现有的nacos环境即可】**

**2 在微服务中引入nacos 的依赖**

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**3 在微服务中添加nacos config 的配置**

```
配置文件优先级(由高到低):
bootstrap.properties -> bootstrap.yml -> application.properties -> application.yml
spring:
  application:
    name: server-product
  cloud:
    nacos:
      config:
        # nacos中心地址
        server-addr: localhost:8848
        # 配置文件格式
        file-extension: yaml
  profiles:
    # 环境标识
    active: dev
```

> 注意:不能使用原来的application.yml 作为配置文件，而是新建一个bootstrap.yml作为配置文件

**4 在nacos 中添加配置**

点击配置列表，点击右边+号，新建配置。在新建配置过程中，要注意下面的细节：

1）Data ID不能随便写，要跟配置文件中的对应：

server-product-dev.yaml

2）配置文件格式要跟配置文件的格式对应，且目前仅仅支持YAML和Properties

3）配置内容与原来的application.yml一样

**5 注释本地的application.yam中的内容， 启动程序进行测试**

**配置动态刷新**

在入门案例中，我们实现了配置的远程存放，但是此时如果修改了配置，我们的程序是无法读取到的，因此，我们需要开启配置的动态刷新功能。

在nacos 中的service-product-dev.yaml 配置项中添加下面测试配置:

```
config:
 appName: product
```

方式一: 硬编码方式

```
@RestController
public class NacosConfigController {
    @Autowired
    private ConfigurableApplicationContext applicationContext;
    @GetMapping("/nacos-config-test1")
    public String nacosConfingTest1() {
        return 
applicationContext.getEnvironment().getProperty("config.appName");
    }
}
```

方式二:  注解方式(推荐)

```
@RestController
@RefreshScope//只需要在需要动态读取配置的类上添加此注解就可以
public class NacosConfigController {
    @Value("${config.appName}")
    private String appName;
    //2 注解方式
    @GetMapping("/nacos-config-test2")
    public String nacosConfingTest2() {
        return appName;
    }
}
```

**配置共享**

如果想在同一个微服务的不同环境之间实现配置共享，其实很简单。

只需要提取一个以spring.application.name 命名的配置文件，然后将其所有环境的公共配置放在里面即可。

比如：

- service-product.yaml配置存放商品微服务的公共配置
- service-product-test.yaml配置存放测试环境的配置
- service-product-dev.yaml配置存放开发环境的配置

**不同微服务中间共享配置**

不同为服务之间实现配置共享的原理类似于文件引入，就是定义一个公共配置，然后在当前配置中引入。

1. 在nacos 中定义一个DataID为all-service.yaml的配置，用于所有微服务共享

2. 修改bootstrap.yaml
   
   ```
   spring:
     application:
       name: server-product
     cloud:
       nacos:
         config:
           # nacos中心地址
           server-addr: localhost:8848
           # 配置文件格式
           file-extension: yaml
           # 配置要引入的配置
           shared-dataids: all-service.yaml
           # 配置要动态刷新的配置
           refreshable-dataids: all-service.yaml
     profiles:
       # 环境标识
       active: test
   ```

## Sentinel--服务容错

### 简介

Sentinel (分布式系统的流量防卫兵) 是阿里开源的一套用于 服务容错 的综合性解决方案。它以流量为切入点, 从 流量控制、熔断降级、系统负载保护 等多个维度来保护服务的稳定性。

服务熔断-般有三种状态:

**熔断关闭**状态(Closed)

服务没有故障时，熔断器所处的状态，对调用方的调用不做任何限制

**熔断开启**状态(Open)

后续对该服务接口的调用不再经过网络，直接执行本地的fallback方法

**半熔断**状态(Half-Open)

尝试恢复服务调用,允许有限的流量调用该服务,并监控调用成功率。如果成功率达到预期，则说明服务已恢复，进入熔断关闭状态;如果成功率仍旧很低，则重新进入熔断关闭状态。

### 构成

**核心库**（Java 客户端）不依赖任何框架/库,能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。

**控制台**（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

### 基本概念

**资源**

资源是Sentinel的关键概念（资源就是Sentinel要保护的东西）。它可以是Java应用程序中的任何内容，可以是一个服务，也可以是一个方法,甚至可以是一段代码。比如/order/prod/1接口方法就可以认为是一个资源

**规则**

作用在资源之上，定义以什么样的方式保护资源，主要包括流量控制规则、熔断降级规则以及系统保护规则。

**容错的三个核心思想：**

1 保证自己不被上游服务压垮（对上游限流）

2 保证自己不被下游服务拖垮（对下游熔断）

3 保证外界环境良好（指定最大QPS）

**Sentinel和Hystrix的区别**

两者的原则是一致的, 都是当一个资源出现问题时,让其快速失败,不要波及到其它服务但是在限制的手段上采取了完全不一样的方法:

Hystrix采用的是**线程池隔离**的方式，优点是做到了资源之间的隔离，缺点是增加了线程切换的成本。

Sentinel采用的是通过并发线程的数量和响应时间来对资源做限制。

**系统负载保护**

Sentinel同时提供系统维度的自适应保护能力。当系统负载较高的时候，如果还持续让请求进入可能会导致系统崩溃，无法响应。在集群环境下，会把本应这台机器承载的流量转发到其它的机器上去。如果这个时候其它的机器也处在一个边缘状态的时候， Sentinel 提供了对应的保护机制，让系统的入口流量和系统的负载达到一个平衡，保证系统在能力范围之内处理最多的请求。

**热点规则**

可以单独对资源（方法）上的参数进行限流（参数级别的限流）。

**授权规则**

限制资源的调用者（黑白名单），比如可以在请求参数或者请求头中定义当当前的请求是PC还是App端，自定义类实现RequestOriginParser方法返回当前的流控应用名称后，即可在编辑授权规则中添加对应资源的规则，限制PC或者App访问。

**系统规则**

系统保护规则从应用级别的入口流量进行控制，从单台机器的总体load、RT（平均响应时间）、吞吐量、QPS、CPU使用率五个维度进行监控。

load1/load5/load15 即 系统1分钟/5分钟/15分钟内的负载。

**自定义异常返回**

自定义类实现UrlBlockHandler接口，实现blocked方法（自定义写入response）。

**自定义处理降级/异常**

添加@SentinelResource注解，注解添加blockHandler或者fallback参数，编写对应参数值的同名方法即可（方法参数与资源参数一致）。

@SentinelResource(value = "message2", blockHandler = "myBlockHandler", fallback = "myThrowable")

也可以使用blockHandlerClass等参数指定异常处理类

@SentinelResource(value = "message2", fallbackClass = MyThrowable.class)

**Sentinel规则持久化**

规则持久化是在各个提供服务的项目中添加持久化规则的处理类，而不是在sentinel管理控制台持久化。

- **编写处理类**
  
  直接拷贝已有项目的

- **添加配置**
  
  在微服务的resources目录下创建配置目录META-INF/services',然后添加文件com.alibaba.csp.sentine1.init.InitFunc，文件内容为上面所写的处理类的包名+类名

### 使用

1 只需要在pom.xml中加入下面依赖（上游项目容错）：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2 运行控制台

```shell
java -jar sentinel-dashboard-1.7.0.jar
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.7.0.jar
```

3 配置控制台地址和端口

修改application.yml文件，添加配置：

```
spring:
  cloud:
    sentinel:
      transport:
        port:9999 #跟控制台交流的端口,随意指定一个未使用的端口即可
        dashboard:localhost:8080 #指定控制台服务的地址
```

### Feign整合Sentinel

1 引入sentinel的依赖

```xml
<!--sentine1客户端-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2 在配置文件中开启Feign对Sentinel的支持

```
feign:
  sentine1:
    enabled:
      true
```

3 创建容错类

给feign所对应的远程接口类创建对应的容错实现类。
容错类需要实现feign所在的接口，并实现其中的方法，一旦Feign远程调用异常，就会调用当前类中同名方法。 

4 为Feign修饰的接口类指定对应的容错实现类

fallback参数指定容错类

```
@FeignClient(value = "server-product", fallback = ProductServiceFallBack.class)
```

> 扩展：
> 如果想在容错类中拿到具体的错误，可以使用如下形式：
> 
> ```
> @FeignClient(value = "server-product", 
> // fallback = ProductServiceFallBack.class
> fallbackFactory=ProductServiceFallBackFactory.class
> )
> ```
> 
> 实现类不再直接实现service接口类，而是实现：implements FallbackFactory<ProductService>
> 
> fallback和fallbackFactory只能同时使用其中一种方式。

## Gateway--服务网关

服务网关与nacos不同的地方是：服务网关是用来解决外部客户端等来调用内部微服务的问题。

所谓的API网关（即服务网关），就是指系统的统一入口, 它封装了应用程序的内部结构,为客户端提供统一服务，一些与业务本身功能无关的公共逻辑可以在这里实现，诸如**认证、鉴权、监控、路由转发**等等。

SpringCloudGateway 用以替换Zuul（springCloud 1.x中使用）

SpringCloudGateway本质是一个单独的jar包（微服务），使用的不是传统的web容器技术，所以该微服务不能进入starter-web

### 创建服务网关（基础版）

1 创建api-gateway微服务，引入依赖：

```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
</dependencies>
```

2 创建主类：

```
@SpringBootApplication
public class GateWayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GateWayApplication.class);
    }
}
```

3 添加配置文件：

```
server:
  port: 7000
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      # 路由数组[路由 就是当请求满足什么条件的时候转到哪个微服务]
      routes:
        # 当前路由的标识，要求唯一
        - id: product_route
          # 请求最终转发的地址
          uri: http://localhost:8081
          # 路由的优先级，数字小优先级高
          order: 1
          # 断言：转发请求满足的条件（boolean）
          predicates:
            # 当请求路径满足path指定的规则时，可转发
            - Path=/product_serv/**
          # 过滤器（在请求过程中，可以对请求进行处理：添加参数等）
          filters:
            # 在请求转发之前去掉一层路径（过滤掉：/product_serv）
            - StripPrefix=1
```

### 创建服务网关（增强版）

在基础版的基础上进行如下操作：

**1** **加入nacos依赖**

```
<!--nacos客户端-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**2** **在主类上添加注解**

```java
@SpringBootApp1ication
@EnableDiscoveryClient
public class ApiGatewayApplication{
    public static void main(String[] args){
        SpringApplication.run(ApiGatewayApplication.class,args);
    }
}
```

**3** **添加配置**：

```
server:
  port: 7000
spring:
  application:
    name: api-gateway
  cloud:
    # nacos地址
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      # 路由数组[路由 就是当请求满足什么条件的时候转到哪个微服务]
      routes:
        # 当前路由的标识，要求唯一
        - id: product_route
          # lb指从nacos中按照名称获取微服务，并遵循负载均衡策略
          uri: lb://server-product
          # 路由的优先级，数字小优先级高
          order: 1
          # 断言：转发请求满足的条件（boolean）
          predicates:
            # 当请求路径满足path指定的规则时，可转发
            - Path=/product_serv/**
          # 过滤器（在请求过程中，可以对请求进行处理：添加参数等）
          filters:
            # 在请求转发之前去掉一层路径
            - StripPrefix=1
      discovery:
        locator:
          # 让gateway可以发现nacos中的微服务
          enabled: true
```

**4** **访问**

访问地址：http://localhost:7000/product_serv/product/1

### 创建服务网关（简洁版）

1 基于增强版，**去掉配置中的routes**相关参数即可，访问的时候路径直接用对应的服务名即可：

```
server:
  port: 7000
spring:
  application:
    name: api-gateway
  cloud:
    # nacos地址
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      discovery:
        locator:
          # 让gateway可以发现nacos中的微服务
          enabled: true
```

**2** **访问**

访问地址：http://localhost:7000/server-product/product/1

按照：网关地址/微服务地址/接口 的格式访问即可。

### Gateway核心架构

**基本概念**

路由(Route)是gateway中最基本的组件之一，表示一个具体的路由信息载体。主要定义了下面的几个信息:

**id**,路由标识符，区别于其他Route。

**uri**,路由指向的目的地uri, 即客户端请求最终被转发到的微服务。

**order**, 用于多个Route之间的排序，数值越小排序越靠前，匹配优先级越高。

**predicate**, 断言的作用是进行条件判断，只有断言都返回真，才会真正的执行路由。

**filter**,过滤器用于修改请求和响应信息。

### 断言

断言就是说在什么条件下才进行路由转发

**内置断言**

SpringCloud Gateway包括许多内置的断言工厂,所有这些断言都与HTTP请求的不同属性匹配。具体如下:

- 基于Datetime类型的断言工厂
  
  此类型的断言根据时间做判断，主要有三个:
  
  AfterRoutePredicateFactory:接收一个日期参数，判断请求日期是否晚于指定日期
  
  BeforeRoutePredicateFactory:接收一个日期参数，判断请求日期是否早于指定日期
  
  BetweenRoutePredicateFactory:接收两个日期参数, 判断请求日期是否在指定时间段内
  
  -After=2019-12-31T23:59:59.789+08:00[Asia/Shanghai]

- 基于远程地址的断言厂
  
  RemoteAddrRoutePredicateFactory:接收一个IP地址段,判断请求主机地址是否在地址段中
  
  -RemoteAddr=192.168.1.1/24

- 基于Cookie的断言工厂
  
  CookieRoutePredicateFactory:接收两个参数，cookie 名字和-个正则表达式。判断请求cookie是否具有
  
  给定名称且值与正则表达式匹配。
  
  -Cookie=chocolate, ch.

- 基于Header的断言厂
  
  HeaderRoutePredicateFactory:接收两个参数,标题名称和正则表达式。判断请求Header是否具有给定名
  
  称且值与正则表达式匹配。
  
  -Header=X-Request-Id,\d+

- 基于Host的断言工厂
  
  HostRoutePredicateFactory:接收-个参数,主机名模式。判断请求的Host是否满足匹配规则。
  
  -Host=** .testhost.orgI

- 基于Method请求方法的断言厂
  
  MethodRoutePredicateFactory:接收一个参数, 判断请求类型是否跟指定的类型匹配。
  
  -Method=GET

- 基于Path请求路径的断言工厂
  
  PathRoutePredicateFactory:接收一个参数，判断请求的URI部分是否满足路径规则。
  
  -Path=/foo/{segment}

- 基于Query请求参数的断言工厂
  
  QueryRoutePredicateFactory:接收两个参数,请求param和正则表达式，判断请求参数是否具有给定名称
  
  且值与正则表达式匹配。
  
  -Query=baz, ba.

- 基于路由权重的断言工厂
  
  ```
  WeightRoutePredicateFactory:接收一个[组名,权重],然后对于同一个组内的路由按照权重转发
  routes:
  -id: weight route1
  uri: host1
  predicates:
  -Path=/product/**
  -Weight=group3, 1
  -id: weight route2
  uri: host2
  predicates:
  -Path=/product/**
  -Weight=group3, 9
  ```
  
  上述的例子：针对同一个请求：/product/**，10个中有1个转发到host1上，9个转发到host2上。

**自定义断言**

设定一个场景，假设我们的应用只让age在(min,max)之间的人来访问。

**1** **添加配置**， 添加一个Age的断言配置：

yml文件：

```
前面的代码省略…
          predicates:
            # 当请求路径满足path指定的规则时，可转发
            - Path=/product_serv/**
            # 限制年龄在18到60岁之间的人访问
            - Age=18,60
```

**2** **自定义一个断言工厂，实现断言方法**：

类名字不能随便写，需要以自己所配置的断言单词开头，后面跟上RoutePredicateFactory （AgeRoutePredicateFactory），必须实现或者继承AbstractRoutePredicateFactory。

```java
/**
 * 自定义断言工厂
 * 泛型 用于接收一个配置类，配置类中用于接收配置文件中的配置
 * 直接参考其他工厂的写法即可。（比如：AfterRoutePredicateFactory）
 *
 * @author jjh
 * @date 2020/3/5
 **/
@Component
public class AgeRoutePredicateFactory extends AbstractRoutePredicateFactory<AgeRoutePredicateFactory.Config> {


    public AgeRoutePredicateFactory(){
        super(AgeRoutePredicateFactory.Config.class);
    }

    /**
     * 用于从配置文件中获取参数值赋值到配置类中的属性上
     * @return
     */
    @Override
    public List<String> shortcutFieldOrder() {
        // 这里的顺序与配置文件中的参数顺序一致
        return Arrays.asList("minAge", "maxAge");
    }

    /**
     * 断言处理
     * @param config
     * @return
     */
    @Override
    public Predicate<ServerWebExchange> apply(AgeRoutePredicateFactory.Config config) {
        return new Predicate<ServerWebExchange>() {
            @Override
            public boolean test(ServerWebExchange serverWebExchange) {
                String ageStr = serverWebExchange.getRequest().getQueryParams().getFirst("age");
                if (StringUtils.isNotBlank(ageStr)) {
                    int age = Integer.parseInt(ageStr);
                    return age > config.getMinAge() && age < config.getMaxAge();
                }
                return false;
            }
        };
    }

    @Data
    public static class Config{
        private int minAge;
        private int maxAge;
    }
}
```

**3** **测试**

访问地址：http://localhost:7000/product_serv/product/1?age=19 即可发现已经被拦截。

### 过滤器

作用：请求传递过程中对请求进行加工

生命周期：pre（身份验证/记录调试信息） post（收集信息和指标）

分类：局部过滤器GatewayFilter（作用在某一个路由上）、全局过滤器GlobalFilter（作用在全局范围）

**局部过滤器**

- **内部过滤器**
  
  AddRequestHeader\SetStatus …

- **自定义局部过滤器**
  
  与自定义断言工厂方法类似

**全局过滤器**

自定义全局过滤器需要实现GlobalFilter和Ordered接口，下面为一个简单的统一鉴权示例：

```java
**
 * 自定义全局过滤器
 * 统一鉴权
 *
 * @author jjh
 * @date 2020/3/5
 **/
@Slf4j
@Component
public class AuthGlobelFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 统一鉴权
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        // 认证失败
        if (!StringUtils.equals("admin", token)) {
            log.info("认证失败。。。");
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        // 认证成功
        return chain.filter(exchange);
    }

    /**
     * 用来标志当前过滤器的优先级;值越小优先级越高
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 结合Sentinel进行网关限流

- **route维度：**
  
  即在Spring配置文件（yml）中配置的路由条目，资源名为对应的routeId

- **自定义API维度：**
  
  用户可以利用sentinel提供的API来定义一些API分组。

**实现route维度限流**

**1 导入依赖**

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
</dependency>
```

**2** **编写配置类**

基于Sentinel的Gateway限流是通过其提供的Filter来完成的，使用时只需注入对应的SentinelGatewayFilter

实例以及SentinelGatewayBlockExceptionHandler实例即可。

```java
/**
 * 网关限流
 *基于Sentinel的Gateway限流是通过其提供的Filter来完成的，使用时只需注入对应的SentinelGatewayFilter
 * 实例以及SentinelGatewayBlockExceptionHandler实例即可。
 *
 * https://www.bilibili.com/video/av89898642/?p=44
 *
 * @author jjh
 * @date 2020/3/8
 **/
@Configuration
public class GatewayConfiguration {

    private final List<ViewResolver> viewResolvers;

    private final ServerCodecConfigurer serverCodecConfigurer;

    public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    /** 初始化一个限流过滤器 */
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }

    /** 配置限流参数*/
    @PostConstruct
    public void initGatewayRules() {
        HashSet<GatewayFlowRule> rules = new HashSet<>();
        rules.add(new GatewayFlowRule("product_route")  // 资源名称，对应路由id
                .setCount(1)    // 限流阈值
                .setIntervalSec(1)  // 统计时间窗口，单位是秒，默认是1秒
        );
        GatewayRuleManager.loadRules(rules);
    }

    /** 配置限流的异常处理器*/
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }

    @PostConstruct
    public void initBlockHandlers() {
        BlockRequestHandler blockRequestHandler = new BlockRequestHandler() {
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
                HashMap<String, Object> map = new HashMap<>();
                map.put("code", 0);
                map.put("message", "接口被限流了");
                return ServerResponse.status(HttpStatus.OK)
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .body(BodyInserters.fromObject(map));
            }
        };
        GatewayCallbackManager.setBlockHandler(blockRequestHandler);
    }
}
```

**自定义API维度**

自定义API分组是一种更细粒度的限流规则定义

修改配置类方法initGatewayRules()，添加方法initCustomizedApis()：

```java
/** 配置限流参数*/
@PostConstruct
public void initGatewayRules() {
    HashSet<GatewayFlowRule> rules = new HashSet<>();
    //route限流
//        rules.add(new GatewayFlowRule("product_route")  // 资源名称，对应路由id
//                .setCount(1)    // 限流阈值
//                .setIntervalSec(1)  // 统计时间窗口，单位是秒，默认是1秒
//        );

    // 自定义API分组
    rules.add(new GatewayFlowRule("product_api1").setCount(1).setIntervalSec(1));
    rules.add(new GatewayFlowRule("product_api2").setCount(1).setIntervalSec(1));

    GatewayRuleManager.loadRules(rules);
}
```

```java
/** 自定义API分组 */
@PostConstruct
private void initCustomizedApis() {
    HashSet<ApiDefinition> definitions = new HashSet<>();
    ApiDefinition api1 = new ApiDefinition("product_api1").setPredicateItems(new HashSet<ApiPredicateItem>() {{
        add(new ApiPathPredicateItem().setPattern("/product_serv/product/api1/**")
                .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
    }});
    ApiDefinition api2 = new ApiDefinition("product_api2").setPredicateItems(new HashSet<ApiPredicateItem>() {{
        add(new ApiPathPredicateItem().setPattern("/product_serv/product/api2/demo1"));
    }});
    definitions.add(api1);
    definitions.add(api2);
    GatewayApiDefinitionManager.loadApiDefinitions(definitions);
}
```

## Sleuth--链路追踪

### 链路追踪

分布式链路追踪(DistributedTracing)，就是将一-次分布式请求还原成调用链路，进行日志记录，性能监控并将一次分布式请求的调用情况集中展示。比如各个服务节点上的耗时、请求具体到达哪台机器上、每个服务节点的请求状态等等。

常见的链路追踪技术有下面这些:

**●cat**

由大众点评开源，基于Java开发的实时应用监控平台，包括实时应用监控，业务监控。集成方案是通过代码埋点的方式来实现监控，比如:拦截器，过滤器等。对代码的侵入性很大，集成成本较高。风险较大。

**●zipkin**

由Twitter公司开源，开放源代码分布式的跟踪系统，用于收集服务的定时数据，以解决微服务架构中的延迟问题，包括:数据的收集、存储、查找和展现。该产品结合spring-cloud-sleuth使用较为简单，集成很方便，但是功能较简单。

**●pinpoint**

Pinpoint是韩国人开源的基于字节码注入的调用链分析,以及应用监控分析工具。特点是支持多种插件，UI功能强大，接入端无代码侵入。

**●skywalking**

SkyWalking是本土开源的基于字节码注入的调用链分析，以及应用监控分析工具。特点是支持多种插件，UI功能较强，接入端无代码侵入。目前已加入Apache孵化器。

**●Sleuth**

SpringCloud提供的分布式系统中链路追踪解决方案。

注意：SpringCloudAlibaba技术栈中没有提供自己的链路追踪技术，我们可以采用Sleuth+Zinkin来做链路追踪解决方案。

### 简介

SpringCloudSleuth主要功能就是在分布式系统中提供追踪解决方案。它大量借用了GoogleDapper的设计,先来了解一下Sleuth中的术语和相关概念。

**●Trace**

由一组TraceId相同的Span串联形成一个树状结构。为了实现请求跟踪，当请求到达分布式系统的入口端点时，只需要服务跟踪框架为该请求创建一个唯一的标识(即Traceld)，同时在分布式系统内部流转的时候,框架始终保持传递该唯一值，直到整个请求的返回。那么我们就可以使用该唯一标识将所有的请求串联起来，形成一条完整的请求链路。

**●Span**

代表了一组基本的工作单元。为了统计各处理单元的延迟，当请求到达各个服务组件的时候，也通过一个唯一标识(Spanld)来标记它的开始、具体过程和结束。通过Spanld的开始和结束时间戳，就能统计该span的调用时间，除此之外，我们还可以获取如事件的名称。请求信息等元数据。

●**Annotation**

用它记录一段时间内的事件，内部使用的重要注释:

- cs(ClientSend)客户端发出请求，开始一个请求的生命

- sr(ServerReceived)服务端接受到请求开始进行处理，sr-cs=网络延迟(服务调用的时间)

- ss(ServerSend)服务端处理完毕准备发送到客户端，ss-sr=服务器上的请求处理时间

- cr(ClientReveived)客户端接受到服务端的响应，请求结束。cr-sr=请求的总时间

### 使用

在父工程pom.xml，添加依赖：

```xml
<dependencies>
    <!--链路追踪 sleuth  https://www.bilibili.com/video/av89898642/?p=48-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
</dependencies>
```

 访问微服务，查看控制台日志，可以发现多了如下日志内容：

[api-gateway,1bf115a6ce2e4724,1bf115a6ce2e4724,true]

> 各变量解释：服务名，traceId，spanId，是否上报追踪结果到第三方

### Zipkin集成

Zipkin是Twitter的一个开源项目，它基于GoogleDapper实现，它致力于收集服务的定时数据，以解决微服务架构中的延迟问题，包括数据的**收集、存储、查找**和**展现**。

我们可以使用它来收集各个服务器上请求链路的跟踪数据，并通过它提供的RESTAPI接口来辅助我们查询跟踪数据以实现对分布式系统的监控程序，从而及时地发现系统中出现的延迟升高问题并找出系统性能瓶颈的根源。

除了面向开发的API接口之外，它也提供了方便的UI组件来帮助我们直观的搜索跟踪信息和分析请求链路明细，比如:可以查询某段时间内各用户请求的处理时间等。

Zipkin提供了可插拔数据存储方式:In-Memory、MySql、Cassandra以及Elasticsearch。

 **构成**

Zipkin分为两端，一个是Zipkin服务端，一个是Zipkin客户端，客户端也就是微服务的应用。

客户端会配置服务端的URL地址，一旦发生服务间的调用的时候，会被配置在微服务里面的Sleuth的监听器监听,并生成相应的Trace和Span信息发送给服务端。

#### ZipKin服务端安装

1. 下载ZipKin的jar包
   
   https://search.maven.org/remote_content?g=io.zipkin.java&a=zipkin-server&v=LATEST&c=exec
   
   访问.上面的网址，即可得到一个jar包，这就是ZipKin服务端的jar包

2. 通过命令行，输入下面的命令**启动ZipKinServer**
   
   ```
   java-jarzipkin-server-2.12.9-exec.jar
   ```

3. 通过浏览器访问http://localhost:9411访问

#### Zipkin客户端集成

ZipKin客户端和Sleuth的集成非常简单，只需要在微服务中添加其依赖和配置即可。

1. 在每个微服务上添加依赖
   
   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zipkin</artifactId>
   </dependency>
   ```

2. 添加配置
   
   ```yaml
   spring:
     # 引入zipkin实现链路追踪日志的上报展示
     zipkin:
       # zipkin server的请求地址
       base-url: http://127.0.0.1:9411/
       # 让nacos把它当作URL，而不是服务名
       discovery-client-enabled: false
     sleuth:
       sampler:
         # 采样的百分比 0 ~ 1.0
         probability: 1.0
   ```

3. 访问微服务
   
   http://localhost:700/order-serv/order/prod/1

4. 访问zipkin的Ul界面，观察效果

#### Zipkin数据持久化

Zipkin Server默认会将追踪数据信息保存到内存,但这种方式不适合生产环境。Zipkin支持将追踪数据持久化到mysq|数据库或elasticsearch中。

**使用mysql实现数据持久化**

1. 创建mysql[数据环境](https://github.com/openzipkin/zipkin/blob/master/zipkin-storage/mysql-v1/src/main/resources/mysql.sql)
   
   ```sql
   CREATE TABLE IF NOT EXISTS zipkin_spans (
     `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
     `trace_id` BIGINT NOT NULL,
     `id` BIGINT NOT NULL,
     `name` VARCHAR(255) NOT NULL,
     `remote_service_name` VARCHAR(255),
     `parent_id` BIGINT,
     `debug` BIT(1),
     `start_ts` BIGINT COMMENT 'Span.timestamp(): epoch micros used for endTs query and to implement TTL',
     `duration` BIGINT COMMENT 'Span.duration(): micros used for minDuration and maxDuration query',
     PRIMARY KEY (`trace_id_high`, `trace_id`, `id`)
   ) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;
   
   ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTracesByIds';
   ALTER TABLE zipkin_spans ADD INDEX(`name`) COMMENT 'for getTraces and getSpanNames';
   ALTER TABLE zipkin_spans ADD INDEX(`remote_service_name`) COMMENT 'for getTraces and getRemoteServiceNames';
   ALTER TABLE zipkin_spans ADD INDEX(`start_ts`) COMMENT 'for getTraces ordering and range';
   
   CREATE TABLE IF NOT EXISTS zipkin_annotations (
     `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
     `trace_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.trace_id',
     `span_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.id',
     `a_key` VARCHAR(255) NOT NULL COMMENT 'BinaryAnnotation.key or Annotation.value if type == -1',
     `a_value` BLOB COMMENT 'BinaryAnnotation.value(), which must be smaller than 64KB',
     `a_type` INT NOT NULL COMMENT 'BinaryAnnotation.type() or -1 if Annotation',
     `a_timestamp` BIGINT COMMENT 'Used to implement TTL; Annotation.timestamp or zipkin_spans.timestamp',
     `endpoint_ipv4` INT COMMENT 'Null when Binary/Annotation.endpoint is null',
     `endpoint_ipv6` BINARY(16) COMMENT 'Null when Binary/Annotation.endpoint is null, or no IPv6 address',
     `endpoint_port` SMALLINT COMMENT 'Null when Binary/Annotation.endpoint is null',
     `endpoint_service_name` VARCHAR(255) COMMENT 'Null when Binary/Annotation.endpoint is null'
   ) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;
   
   ALTER TABLE zipkin_annotations ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `span_id`, `a_key`, `a_timestamp`) COMMENT 'Ignore insert on duplicate';
   ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`, `span_id`) COMMENT 'for joining with zipkin_spans';
   ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTraces/ByIds';
   ALTER TABLE zipkin_annotations ADD INDEX(`endpoint_service_name`) COMMENT 'for getTraces and getServiceNames';
   ALTER TABLE zipkin_annotations ADD INDEX(`a_type`) COMMENT 'for getTraces and autocomplete values';
   ALTER TABLE zipkin_annotations ADD INDEX(`a_key`) COMMENT 'for getTraces and autocomplete values';
   ALTER TABLE zipkin_annotations ADD INDEX(`trace_id`, `span_id`, `a_key`) COMMENT 'for dependencies job';
   
   CREATE TABLE IF NOT EXISTS zipkin_dependencies (
     `day` DATE NOT NULL,
     `parent` VARCHAR(255) NOT NULL,
     `child` VARCHAR(255) NOT NULL,
     `call_count` BIGINT,
     `error_count` BIGINT,
     PRIMARY KEY (`day`, `parent`, `child`)
   ) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;
   ```

2. 在启动Zipkin Server的时候，指定数据保存的mysql的信息
   java -jar zipkin-server-2.12.9-exec.jar --STORAGE_TYPE=mysql --MYSQL_HOST=127.0.0.1 --MYSQL_TCP_PORT=3306 --MYSQL_DB=zipkin --MYSQL_USER=root --MYSQL_PASS=root

**使用elasticseach实现数据持久化**

1. 下载elasticSearch
   注意版本的兼容性，目前这里使用6.x版本的elasticSearch，7.x版本的会报错不兼容
   https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-7
2. 启动elasticSearch
   运行：elasticsearch-6.8.7\bin\elasticsearch.bat
3. 在启动Zipkin Server的时候，指定数据保存的elasticSearch的信息
   java -jar zipkin-server-2.12.9-exec.jar --STORAGE_TYPE=elasticsearch --ES-HOST=localhost:9200

## RocketMQ--消息驱动

MQ（Message Queue）是一种跨进程的通信机制，用于传递消息。通俗点说，就是一个先进先出的数据结构。

RocketMQ是阿里巴巴开源的分布式消息中间件，现在是Apache的一个顶级项目。在阿里内部使用非常广泛，已经经过了"双11"这种万亿级的消息流转。

### RocketMQ环境搭建

接下来我们先在linux平台下安装一个RocketMQ的服务

**环境准备**

- RocketMQ：http://rocketmq.apache.org/release_notes/release-notes-4.4.0/
- Java1.8

**Linux环境**

**安装RocketMQ**

上传文件到Linux系统，解压，切换到解压后目录下，运行：

编辑bin/runbroker.sh 和 bin/runserver.sh文件。修改里面的：

```
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"为 JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

```
> nohup ./bin/mqnamesrv &
> bin/mqbroker -n localhost:9876 &
```

**关闭RocketMQ**

```
> bin/mqshutdown broker
> bin/mqshutdown namesrv
```

**Windows**[**环境**](https://www.jianshu.com/p/4a275e779afa)

1 解压压缩包到自定义目录，配置ROCKETMQ_HOME环境变量指定到解压的目录

2 启动NAMESERVER：进入解压目录下的bin目录，执行 start mqnamesrv.cmd

3 启动BROKER：进入解压目录下的bin目录，执行 start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true

> **注意：**假如弹出提示框提示‘错误: 找不到或无法加载主类 xxxxxx’。打开runbroker.cmd，找到JVM Configuration下面的参数，然后将里面的%CLASSPATH%加上英文双引号。保存并重新执行start语句。

### RocketMQ概念

- **Broker**（邮递员）
  
  接收 存储 投递 消息

- **Name Server**（邮局）
  
  管理Broker

- **Producer**（寄件人）
  
  生产消息

- **Consumer**（收件人）
  
  消息消费者

- **Topic**（地区）
  
  区分不同消息

- **Message Queue**（邮件）
  
  一个Topic可以设置一个或多个message queue，这样消息可以并行往各个message queue发送，消费者也可以并行读取消息。

- **Message**（信）
  
  消息载体

### 使用Java发送接收消息

配置文件中添加依赖：

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.0.2</version>
</dependency>
```

消息发送步骤:

1. 创建消息生产者, 指定生产者所属的组名

2. 指定Nameserver地址

3. 启动生产者

4. 创建消息对象，指定主题、标签和消息体

5. 发送消息

6. 关闭生产者

## [Seata]([快速开始-阿里云Spring Cloud Alibaba官网](https://sca.aliyun.com/docs/2022/user-guide/seata/quick-start/))--分布式事务

### 分布式事务解决方案

#### AT 模式

seata 默认为AT模式，是实际项目中比较常用的一种模式，他采用的也是两阶段提交，不过弥补了XA模式中资源锁定周期过长的缺点。相对于 XA 来说，性能更好一些，但缺点就是数据不是强一致，因为它的数据会真实的提交到数据库的，而如果后面做分支事务有问题的话，回滚靠的是日志来实现最终一致。

**使用前提**

- 基于支持本地 ACID 事务的关系型数据库。
- Java 应用，通过 JDBC 访问数据库。

**工作原理**

在 AT 模式下，基于支持本地 ACID 事务的关系型数据库，通过数据源代理类拦截并解析数据库的操作。在业务数据更新前后，记录相应的回滚日志，并在事务提交前，将业务数据的更新和回滚日志一并提交到事务协调器。如果事务正常提交，事务协调器会删除相应的回滚日志；如果事务回滚，事务协调器则根据回滚日志对数据进行逆向补偿更新。

**特点**

1. 对业务无侵入：开发人员可以像使用本地事务一样使用分布式事务，无需额外的代码编写和复杂的配置。
2. 高效性：通过优化事务处理流程，减少了分布式事务带来的性能损耗。
3. 一致性：确保分布式环境下的数据一致性，避免出现数据不一致的情况。

##### AT 模式与 2PC（两阶段提交）模式的区别

**一、实现机制**

1. AT 模式：
   
   - 基于支持本地 ACID 事务的关系型数据库，通过数据源代理类拦截并解析数据库操作，记录业务数据更新前后的回滚日志，并在事务提交前将业务数据更新和回滚日志一并提交到事务协调器。
   - 如果事务正常提交，事务协调器删除回滚日志；若事务回滚，事务协调器根据回滚日志对数据进行逆向补偿更新。

2. 2PC 模式：
   
   - 分为两个阶段，准备阶段和提交阶段。
   - 在准备阶段，事务协调者向所有参与者发送准备请求，参与者执行事务操作但不提交，然后向协调者回复是否可以提交。
   - 在提交阶段，如果所有参与者都回复可以提交，协调者向所有参与者发送提交请求；否则，发送回滚请求。

**二、对业务的侵入性**

1. AT 模式：对业务无侵入，开发人员可以像使用本地事务一样使用分布式事务，无需额外的代码编写和复杂的配置。
2. 2PC 模式：对业务有一定的侵入性，需要参与者实现特定的接口来响应协调者的请求，并且在事务执行过程中可能会出现阻塞等待的情况。

**三、性能表现**

1. AT 模式：通过优化事务处理流程，减少了分布式事务带来的性能损耗，相对性能较好。
2. 2PC 模式：由于需要在两个阶段进行协调和通信，可能会导致较长的事务执行时间和较大的性能开销，特别是在参与者较多或网络延迟较大的情况下。

**四、适用场景**

1. AT 模式：适用于微服务架构、分布式数据库操作以及大规模分布式系统等场景，对性能要求较高且希望对业务代码影响较小。
2. 2PC 模式：适用于对数据一致性要求非常高的场景，如金融交易系统等，但需要考虑性能和可用性方面的挑战。

### 重要组件组成

TC：Transaction Coordinator 事务协调器，管理全局的分支事务的状态，用于全局性事务的提交和回滚。

TM：Transaction Manager 事务管理器，用于开启、提交或者回滚全局事务。

RM：Resource Manager 资源管理器，用于分支事务上的资源管理，向TC注册分支事务，上报分支事务的状态，接受TC的命令来提交或者回滚分支事务。

### 执行流程

1. A服务的TM向TC申请开启一个全局事务，TC就会创建一个全局事务并返回一个唯一的XID

2. A服务的RM向TC注册分支事务，并及其纳入XID 对应全局事务的管辖

3. A服务执行分支事务，向数据库做操作

4. A服务开始远程调用B服务，此时XID 会在微服务的调用链上传播

5. B服务的RM向TC注册分支事务，并将其纳入XID 对应的全局事务的管辖

6. B服务执行分支事务，向数据库做操作

7. 全局事务调用链处理完毕，TM根据有无异常向TC发起全局事务的提交或者回滚

8. TC协调其管辖之下的所有分支事务， 决定是否回滚

### Seata 实现2PC 与传统2PC 的差别

1. 架构层次方面，传统2PC 方案的 RM 实际上是在数据库层，RM本质上就是数据库自身，通过XA协议实现，而 Seata 的RM是以jar包的形式作为中间件层部署在应用程序这一侧的。

2. 两阶段提交方面，传统2PC 无论第二阶段的决议是commit 还是rollback，事务性资源的锁都要保持到Phase2完成才释放。而Seata 的做法是在Phase1 就将本地事务提交，这样就可以省去Phase2持锁的时间，整体提高效率。

### 启动Seata

**下载seata**  
https://github.com/seata/seata/releases/v0.9.0/

**修改配置文件**  
registry.conf

```
registry {
    type = "nacos"
    nacos {
        serverAddr = "localhost"
        namespace = "public"
        cluster = "default"
    }
}
config {
    type = "nacos"
    nacos {
        serverAddr = "localhost"
        namespace = "public"
        cluster = "default"
    }
}
```

nacos-config.txt

第14行添加如下内容：

```
service.vgroup_mapping.service-product=default
service.vgroup_mapping.service-order=default
```

> 这里的语法为：service.vgroup_mapping.${your-service-gruop}=default ，中间的${your-service-gruop} 为自己定义的服务组名称， 这里需要我们在程序的配置文件中配置。

初始化seata 在nacos 的配置

```
# 初始化seata 的nacos配置
# 注意: 这里要保证nacos是已经正常运行的
cd conf
nacos-config.sh 127.0.0.1
```

> 注：如果在window下的话，需要使用sh nacos-config.sh 127.0.0.1命令（如果sh不可用，需要安装git之后，[配置git](https://blog.csdn.net/weixin_42376686/article/details/82391410)的环境变量即可）

### 使用Seata 实现事务控制

**初始化数据表**

在我们的数据库中加入一张undo_log表,这是Seata 记录事务日志要用到的表

```
CREATE TABLE `undo_log`
(
  `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT,
  `branch_id`     BIGINT(20)   NOT NULL,
  `xid`           VARCHAR(100) NOT NULL,
  `context`       VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB     NOT NULL,
  `log_status`    INT(11)      NOT NULL,
  `log_created`   DATETIME     NOT NULL,
  `log_modified`  DATETIME     NOT NULL,
  `ext`           VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = INNODB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```

**添加配置**

在需要进行分布式控制的微服务中进行下面几项配置:

1 添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2 添加配置类：DataSourceProxyConfig

Seata 是通过代理数据源实现事务分支的，所以需要配置 io.seata.rm.datasour ce.DataSourceProxy 的Bean，且是 @Primary 默认的数据源，否则事务不会回滚，无法实现分布式事务

```java
@Configuration
public class DataSourceProxyConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DruidDataSource druidDataSource() {
        return new DruidDataSource();
    }
    @Primary
    @Bean
    public DataSourceProxy dataSource(DruidDataSource druidDataSource) {
        return new DataSourceProxy(druidDataSource);
    }
}
```

3 registry.conf

在resources 下添加Seata 的配置文件 registry.conf

```
registry {
    type = "nacos"
    nacos {
        serverAddr = "localhost"
        namespace = "public"
        cluster = "default"
    }
}
config {
    type = "nacos"
    nacos {
        serverAddr = "localhost"
        namespace = "public"
        cluster = "default"
    }
}
```

4 bootstrap.yaml

```
spring:
  application:
    name: service-product
  cloud:
    nacos:
      config:
        server-addr: localhost:8848 # nacos的服务端地址
        namespace: public
        group: SEATA_GROUP
    alibaba:
      seata:
        tx-service-group: ${spring.application.name}
```

**在order 微服务开启全局事务**

```
@GlobalTransactional//全局事务控制
public Order createOrder(Integer pid) {}
```

再次下单测试

**要点说明：**

1、每个RM使用DataSourceProxy连接数据库，其目的是使用ConnectionProxy，使用数据源和数据连接代理的目的就是在第一阶段将undo_log和业务数据放在一个本地事务提交，这样就保存了只要有业务操作就一定有undo_log。

2、在第一阶段undo_log中存放了数据修改前和修改后的值，为事务回滚作好准备，所以第一阶段完成就已经将分支事务提交，也就释放了锁资源。

3、TM开启全局事务开始，将XID 全局事务id 放在事务上下文中，通过feign调用也将XID 传入下游分支事务，每个分支事务将自己的Branch ID分支事务ID与XID 关联。

4、第二阶段全局事务提交，TC会通知各各分支参与者提交分支事务，在第一阶段就已经提交了分支事务，这里各各参与者只需要删除undo_log即可，并且可以异步执行，第二阶段很快可以完成。

5、第二阶段全局事务回滚，TC会通知各各分支参与者回滚分支事务，通过 XID 和 Br anch ID 找到相应的回滚日志，通过回滚日志生成反向的 SQL 并执行，以完成分支事务回滚到之前的状态，如果回滚失败则会重试回滚操作。
