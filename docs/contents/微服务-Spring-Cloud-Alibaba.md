> 参考：https://www.bilibili.com/video/av89898642



## 介绍

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服

务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接

入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。



## 主要功能

**服务限流降级** ：默认支持 WebServlet、WebFlux， OpenFeign、RestTemplate、Spring Cloud

Gateway， Zuul， Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修

改限流降级规则，还支持查看限流降级 Metrics 监控。

**服务注册与发现** ：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。

**分布式配置管理** ：支持分布式系统中的外部化配置，配置更改时自动刷新。

**消息驱动能力** ：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。

**分布式事务** ：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。

**阿里云对象存储** ：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任

何时间、任何地点存储和访问任意类型的数据。

**分布式任务调度** ：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有

Worker（schedulerx-client）上执行。

**阿里云短信服务** ：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建

客户触达通道。



## 组件

**Sentinel** ：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳

定性。

**Nacos** ：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

**RocketMQ** ：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠

的消息发布与订阅服务。

**Dubbo** ：Apache Dubbo? 是一款高性能 Java RPC 框架。

**Seata** ：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

**Alibaba Cloud ACM** ：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心

产品。

**Alibaba Cloud OSS** : 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提

供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和

访问任意类型的数据。

**Alibaba Cloud SchedulerX** : 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精

准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

**Alibaba Cloud SMS** : 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速

搭建客户触达通道。



## Nacos Discovery--服务治理

### 简介

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速

实现动态服务发现、服务配置、服务元数据及流量管理。

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



## Sentinel--服务容错

### 简介

Sentinel (分布式系统的流量防卫兵) 是阿里开源的一套用于 服务容错 的综合性解决方案。它以流量

为切入点, 从 流量控制、熔断降级、系统负载保护 等多个维度来保护服务的稳定性。



服务熔断-般有三种状态:

**熔断关闭**状态(Closed)

服务没有故障时，熔断器所处的状态，对调用方的调用不做任何限制

**熔断开启**状态(Open)

后续对该服务接口的调用不再经过网络，直接执行本地的fallback方法

**半熔断**状态(Half-Open)

尝试恢复服务调用,允许有限的流量调用该服务,并监控调用成功率。如果成功率达到预期，则说明服务

已恢复，进入熔断关闭状态;如果成功率仍旧很低，则重新进入熔断关闭状态。



### 构成

**核心库**（Java 客户端）不依赖任何框架/库,能够运行于所有 Java 运行时环境，同时对 Dubbo /

Spring Cloud 等框架也有较好的支持。

**控制台**（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等

应用容器。



### 基本概念

**资源**

资源是Sentinel的关键概念（资源就是Sentinel要保护的东西）。它可以是Java应用程序中的任何内容，可以是一个服务，也可以是一个方法,

甚至可以是一段代码。比如/order/prod/1接口方法就可以认为是一个资源

**规则**

作用在资源之上，定义以什么样的方式保护资源，主要包括流量控制规则、熔断降级规则以及系统保护规则。

**容错的三个核心思想：**

1 保证自己不被上游服务压垮（对上游限流）

2 保证自己不被下游服务拖垮（对下游熔断）

3 保证外界环境良好（指定最大QPS）

 

**Sentinel和Hystrix的区别**

两者的原则是一致的, 都是当一个资源出现问题时,让其快速失败,不要波及到其它服务

但是在限制的手段上采取了完全不一样的方法:

Hystrix采用的是**线程池隔离**的方式，优点是做到了资源之间的隔离，缺点是增加了线程切换的成本。

Sentinel采用的是通过并发线程的数量和响应时间来对资源做限制。

 

**系统负载保护**

Sentinel同时提供系统维度的自适应保护能力。当系统负载较高的时候，如果还持续让请求进入可能会导

致系统崩溃，无法响应。在集群环境下，会把本应这台机器承载的流量转发到其它的机器上去。如果这个时候

其它的机器也处在一个边缘状态的时候， Sentinel 提供了对应的保护机制，让系统的入口流量和系统的负载达

到一个平衡，保证系统在能力范围之内处理最多的请求。

 

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

  在微服务的resources目录下创建配置目录META-INF/services',然后添加文件

  com.alibaba.csp.sentine1.init.InitFunc，文件内容为上面所写的处理类的包名+类名





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





## **Gateway--服务网关**

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

### **链路追踪**

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
	
1. 通过命令行，输入下面的命令**启动ZipKinServer**
    ```
    java-jarzipkin-server-2.12.9-exec.jar
    ```
    
1. 通过浏览器访问http://localhost:9411访问




#### Zipkin客户端集成

ZipKin客户端和Sleuth的集成非常简单，只需要在微服务中添加其依赖和配置即可。

1. 在每个微服务上添加依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zipkin</artifactId>
    </dependency>
    ```

1. 添加配置

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

1. 访问微服务

   http://localhost:700/order-serv/order/prod/1

1. 访问zipkin的Ul界面，观察效果



#### **Zipkin数据持久化**

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
	
1. 在启动Zipkin Server的时候，指定数据保存的mysql的信息
   java -jar zipkin-server-2.12.9-exec.jar --STORAGE_TYPE=mysql --MYSQL_HOST=127.0.0.1 --MYSQL_TCP_PORT=3306 --MYSQL_DB=zipkin --MYSQL_USER=root --MYSQL_PASS=root



**使用elasticseach实现数据持久化**

1. 下载elasticSearch
   注意版本的兼容性，目前这里使用6.x版本的elasticSearch，7.x版本的会报错不兼容
   https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-7
2. 启动elasticSearch
   运行：elasticsearch-6.8.7\bin\elasticsearch.bat
3. 在启动Zipkin Server的时候，指定数据保存的elasticSearch的信息
   java -jar zipkin-server-2.12.9-exec.jar --STORAGE_TYPE=elasticsearch --ES-HOST=localhost:9200



