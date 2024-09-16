## 概述

Spring Boot （Boot 顾名思义，是引导的意思）[框架](https://www.zhihu.com/question/53729800)是用于简化 Spring 应用从搭建到开发的过程。应用**开箱即用**，只要通过一个指令，包括命令行 java -jar 、SpringApplication 应用启动类 、 Spring Boot Maven 插件等，就可以启动应用了。另外，Spring Boot 强调只需要很少的配置文件，所以在开发生产级 Spring 应用中，让开发变得更加高效和简易。目前，Spring Boot 版本是 2.x 版本。Spring Boot 包括 WebFlux。

## Spring Boot 核心特性

以下链接内容将介绍 Spring Boot 相关的细节内容。 在这里，您可以学习到可能需要使用和自定义的主要功能。

> [Spring Boot 核心特性_学习-阿里云Spring Cloud Alibaba官网](https://sca.aliyun.com/learn/spring-boot/core/)

## 作用

- **自动配置**：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置
- **起步依赖**：告诉Spring Boot需要什么功能，它就能引入需要的库。
- 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- Actuator：让你能够深入运行中的Spring Boot应用程序，一探究竟。

## Spring Boot 特性

- 支持运行期内嵌容器，如 Tomcat、Jetty
- 强大的开发包，支持热启动
- 自动管理依赖
- 自带应用监控
- 非常简洁的安全策略集成



### 优点

1. **独立运行**
   
   内置 Web 服务器，Spring Boot 默认集成了 Tomcat、Jetty 或 Undertow 等内嵌 Web 服务器，使得应用程序可以独立运行，不需要外部的 Web 容器。

2. **简化配置**
   
   约定优于配置：遵循 “约定优于配置” 的原则，减少了开发者需要进行的配置工作。例如，默认的配置文件`application.properties`或`application.yml`中已经定义了很多常用的配置项，开发者只需要根据项目需求修改其中的部分配置即可。

3. **自动配置**
   
   Spring Boot 能够根据项目依赖自动配置 Spring 应用。例如，当在项目中添加了`spring-boot-starter-web`依赖时，Spring Boot 会自动配置 Tomcat 服务器、Spring MVC 框架等相关组件，无需开发者手动进行繁琐的配置工作。

4. **应用监控**
   
   Spring Boot Actuator 提供了健康检查的功能，可以监控应用程序的状态。还提供了监控指标的功能，可以收集和展示应用程序的运行数据。



### 自动装配的流程

**@SpringBootApplication 注解**是 Spring Boot 应用的核心注解，它是一个组合注解，其中包含了`@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan`。其中 **@EnableAutoConfiguration 注解** 是实现自动装配的关键。它通过`@Import(AutoConfigurationImportSelector.class)`导入了`AutoConfigurationImportSelector`类。在`AutoConfigurationImportSelector`的`selectImports()`方法中，会从类路径下的`META-INF/spring.factories`文件中加载自动配置类的全限定名。每个自动配置类都使用了`@Conditional`系列注解来指定在满足特定条件时才进行自动装配。Spring 容器会根据这些自动配置类来进行 bean 的装配，从而实现了根据项目依赖自动配置 Spring 应用的功能。

1. **加载META-INF/spring.factories文件**：
   - 在Spring Boot应用程序启动时，首先加载`META-INF/spring.factories`文件，获取所有自动配置类的全限定名。该文件位于各个Starter依赖的jar包中，其中定义了自动配置类的全限定名。
2. **条件评估**：
   - Spring Boot根据条件注解对自动配置类进行条件评估。只有当条件满足时，相应的自动配置类才会被选中进行自动配置。
3. **创建和注册Bean**：
   - 对于被选中的自动配置类，Spring Boot会根据其中的配置信息创建相应的Bean，并将其注册到Spring容器中。
4. **应用配置**：
   - Spring Boot会遍历所有的自动配置类，将满足条件的配置都应用到应用程序中。

#### 什么是 DeferredImportSelector

`DeferredImportSelector` 是一个接口，它继承自 `ImportSelector`。它的主要用途是在某些情况下延迟配置类的导入，直到特定条件得到满足为止。这通常用于处理一些依赖于其他配置类的情况，或者当需要在运行时动态决定导入哪些类时。

##### 使用场景

- **场景 1：条件性导入配置类**
  
  假设我们需要根据一些条件来决定是否导入某个配置类。在这种情况下，我们可以使用 `DeferredImportSelector` 来实现条件性的导入。

- **场景 2：避免循环依赖**
  
  有时候，两个配置类之间可能会出现相互依赖的情况。在这种情况下，可以使用 `DeferredImportSelector` 来延迟其中一个配置类的导入，以避免循环依赖的问题。

### 什么是 bootstrap.yml

`bootstrap.yml` 是 Spring Boot 应用程序中用于配置应用程序启动时的一些核心配置文件之一。它主要用于存储应用程序启动时的一些基础配置，如 Spring Cloud 的服务发现、配置中心等相关的配置。这些配置在应用程序启动之初就需要加载，因此它们通常位于 `bootstrap.yml` 文件中。

- 在 Spring Boot 应用中，`bootstrap.yml`的加载顺序比`application.yml`（或`application.properties`）更早。这使得它适合配置一些在应用程序上下文创建之前就需要被加载的配置信息。
- 例如，在使用 Spring Cloud Config 时，`bootstrap.yml`**用于**配置**从配置中心获取配置文件**的相关信息（如配置中心的地址、配置文件的名称等），这样在应用程序启动的早期就能从配置中心获取到配置信息并进行应用。

#### bootstrap.yml 与 application.yml 的区别

1. **加载时机不同**：
   
   - `bootstrap.yml`：在 Spring Boot 应用程序启动时最先加载，主要用于初始化 Spring 的环境变量，如激活特定的配置中心或服务发现客户端等。
   - `application.yml`：在应用程序启动之后加载，主要用于配置应用程序的具体行为，如数据库连接、日志配置等。

2. **配置优先级**：
   
   - `bootstrap.yml` 中的配置优先级高于 `application.yml` 中的配置。这意味着 `bootstrap.yml` 中的配置会覆盖 `application.yml` 中的同名配置项。

#### 注意事项

- 如果没有特殊的需求，可以不使用 `bootstrap.yml` 文件，而将所有配置放在 `application.yml` 或 `application.properties` 文件中。
- 如果使用了 `bootstrap.yml` 文件，确保其中的配置不会与 `application.yml` 中的配置产生冲突。

#### 总结

Spring Cloud 的一些核心功能，如从配置中心（如 Spring Cloud Config Server）获取配置信息，依赖于`bootstrap.yml`来配置与外部配置中心交互的相关属性，例如配置中心的地址、配置文件的名称、环境等信息。这些信息需要在应用启动的早期阶段就被加载和处理，以便在应用上下文创建之前就能够从配置中心获取到配置信息。这是 Spring Cloud 特有的需求，Spring Boot 本身没有这样的机制。

## SpringApplication

**1. 基本概念**

SpringApplication 是 Spring Boot 应用程序的引导类，用于启动一个 Spring Boot 应用。它封装了启动 Spring 应用程序所需的复杂配置和初始化过程。

**2. 主要作用**

- **配置加载**：自动加载和配置应用程序的环境，包括从多个源（如 application.properties、application.yml 文件，系统环境变量，命令行参数等）读取配置信息，并将这些信息整合到 Spring 的 Environment 中。
- **应用上下文创建**：根据应用的类型和配置，选择合适的应用程序上下文（如 AnnotationConfigApplicationContext 或 ServletWebServerApplicationContext 等）来管理应用程序中的 bean。
- **启动流程管理**：负责整个启动流程的管理，包括触发各种启动相关的事件（如 ApplicationStartingEvent、ApplicationEnvironmentPreparedEvent 等），让开发者可以在不同的启动阶段进行自定义操作。

### run方法做的事情

Spring Boot 的 `run` 方法是启动 Spring Boot 应用程序的入口点。这个方法通常是通过调用 `SpringApplication.run()` 来实现，主要的关键步骤如下：

1. **推断应用程序类型**：首先，`SpringApplication` 会尝试推断应用程序的类型（例如，是传统的 web 应用程序、反应式 web 应用程序还是其他类型）。这通常是通过检查类路径上的特定库（如 Spring MVC 或 Spring WebFlux）来完成的。

2. **加载 Spring 应用上下文**：接下来，`SpringApplication` 会创建一个 `ApplicationContext`（应用上下文），这是 Spring 框架的核心接口，用于提供配置信息给应用程序中的对象。根据应用程序的类型，可能会创建不同类型的 `ApplicationContext`（如 `AnnotationConfigApplicationContext`、`AnnotationConfigReactiveWebApplicationContext` 等）。

3. **加载 Spring Boot 自动配置**：Spring Boot 的自动配置是 Spring Boot 的核心特性之一。在这一步，Spring Boot 会尝试自动配置你的应用程序。它通过检查类路径上的项目依赖、属性设置和其他因素来推断你可能需要的配置，并自动应用这些配置。自动配置是通过 `@EnableAutoConfiguration` 注解触发的，该注解通常包含在 `@SpringBootApplication` 注解中。

4. **加载应用程序的 Bean**：在创建并配置好 `ApplicationContext` 之后，Spring Boot 会加载应用程序中定义的 Bean。这包括通过 `@Component`、`@Service`、`@Repository`、`@Controller` 等注解标记的类，以及通过 Java 配置类（使用 `@Configuration` 和 `@Bean` 注解）定义的 Bean。

5. **启动嵌入式服务器（如果适用）**：如果你的 Spring Boot 应用程序是一个 web 应用程序，并且你选择了使用 Spring Boot 的嵌入式服务器（如 Tomcat、Jetty 或 Undertow），那么在这一步，Spring Boot 会启动这个服务器。这允许你的应用程序作为一个独立的 web 服务器运行，而无需部署到外部容器中。

6. **运行应用程序**：最后，一旦所有必要的配置和 Bean 都已加载，并且（如果适用）嵌入式服务器也已启动，Spring Boot 应用程序就会开始运行。这通常意味着你的应用程序现在可以接受请求（对于 web 应用程序）或执行其他类型的任务（对于非 web 应用程序）。

**总结**

`SpringApplication.run(...)` 方法是 Spring Boot 应用程序启动的核心，它负责初始化和启动整个 Spring 应用程序上下文。这一过程包括了解析命令行参数、创建和配置 `ApplicationContext`、注册 Bean 定义、刷新上下文、启动 Web 服务器（如果是 Web 应用程序）、执行启动后的初始化任务等。通过这一系列的操作，Spring Boot 应用程序能够快速启动并准备好接收请求。

### 详细启动过程

- SpringBoot的[启动过程](https://www.jianshu.com/p/cb5cb5937686)，实际上就是对ApplicationContext的初始化过程。
- ApplicationContext创建后立刻为其设置Environment，并由**ApplicationContextInitializer**对其进一步封装。
- 通过*SpringApplicationRunListener*在ApplicationContext初始化过程中各个时点发布各种广播事件，并由*ApplicationListener*负责接收广播事件。
- 初始化过程中完成IoC的注入，包括通过@EnableAutoConfiguration导入的各种自动配置类。
- 初始化完成前调用ApplicationRunner和CommandLineRunner的实现类。

启动流程主要分为三个部分：

1. 第一部分进行SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器
2. 第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块
3. 第三部分是自动化配置模块

![img](../_images/f3ee7c15-e09b-4378-a184-c50eeb5930b7.png)

## Spring Boot扩展机制

[java spi](https://blog.csdn.net/lldouble/article/details/80690446)的具体约定为:当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。 基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。

在Spring中也有一种类似与Java SPI的加载机制。它在META-INF/spring.factories文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。这种自定义的SPI机制是Spring Boot Starter实现的基础。

## 重要注解

**@ComponentScan** 注解的作用是扫描@SpringBootApplication所在的Application类（即spring-boot项目的入口类）所在的包（basepackage）下所有的@component注解（或拓展了@component的注解：`@Service`、`@Repository`、`@Controller`）标记的bean，并注册到spring容器中。在没有`@ComponentScan`之前，需要在 XML 配置文件中通过`<context:component - scan>`元素来指定包扫描路径。`@ComponentScan`注解提供了一种基于 Java 的配置方式，使得配置更加简洁、直观，并且与 Java 代码的结合更加紧密。

  **@EnableAutoConfiguration** 自动配置：从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中EnableAutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

> 出处：[Springboot 启动原理详细解析](https://www.cnblogs.com/jstarseven/p/11087157.html)

## 配置读取

Spring Boot 读取配置的方式主要有以下几种：

- **使用 @Value 注解**
  
  @Value`注解可以直接将配置文件中的值注入到 Spring 管理的 bean 的成员变量中。可以指定默认值：例如：
  
  `@Value("${myapp.description:Default description}")`
  
  ```java
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.stereotype.Component;
  
  @Component
  public class MyComponent {
      @Value("${myapp.name}")
      private String appName;
  
      // 可以在其他方法中使用appName变量
  }
  ```

- **使用 @ConfigurationProperties 注解**
  
  当需要将一组相关的配置项映射到一个 Java 类时，可以使用`@ConfigurationProperties`注解。
  
  ```properties
  myapp.database.url=jdbc:mysql://localhost:3306/mydb
  myapp.database.username=root
  myapp.database.password=secret
  ```
  
  可以创建一个对应的类来接收这些配置：
  
  ```java
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.stereotype.Component;
  
  @Component
  @ConfigurationProperties(prefix = "myapp.database")
  public class DatabaseConfig {
      private String url;
      private String username;
      private String password;
  
      // 提供getter和setter方法
  }
  ```

- **通过 Environment 对象**
  
  可以在 Spring 管理的 bean 中通过构造函数或者`@Autowired`注解注入`Environment`对象。使用`Environment`对象的`getProperty`方法来读取配置值。例如：
  
  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.core.env.Environment;
  import org.springframework.stereotype.Component;
  
  @Component
  public class MyComponent {
      private final Environment environment;
  
      @Autowired
      public MyComponent(Environment environment) {
          this.environment = environment;
      }
  
      public void doSomething() {
          String appName = environment.getProperty("myapp.name");
          // 使用appName进行操作
      }
  }
  ```

## 内嵌服务器

Spring Boot 支持使用内嵌的 Web 服务器来快速构建和运行 Web 应用程序。内嵌服务器是指直接嵌入到应用程序中的 Web 服务器，而不是像传统 Java Web 应用那样需要单独部署到外部的 Web 服务器（如 Tomcat 或 Jetty）上。使用内嵌服务器可以简化开发流程，提高开发效率，并且使得部署更加简单。

### 支持的内嵌服务器

Spring Boot 默认支持以下几种内嵌服务器：

1. **Tomcat**：Spring Boot **默认**使用**的内嵌服务器**。适合大多数传统的 Java Web 应用场景。
2. **Jetty**：轻量级的内嵌服务器，适合高性能场景。
3. **Undertow**：轻量级的内嵌服务器，由 Red Hat 开发，适合高并发场景。

### 使用 Spring Boot 内嵌服务器的优势

1. **简化开发流程**：不需要单独安装和配置外部 Web 服务器。
2. **快速启动**：内嵌服务器启动速度快，适合开发环境。
3. **简化部署**：打包后的应用程序可以直接运行，不需要额外的部署步骤。
4. **轻量级**：内嵌服务器占用资源少，适合部署在资源受限的环境中。

#### 更换为 Jetty 或 Undertow

如果你想要更换内嵌服务器，可以通过添加相应的依赖来实现：

Jetty：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

Undertow：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

## SpringBoot 打包的 jar

Spring Boot 打包的 jar 和普通 jar 的区别简要如下：

**内容方面**

- Spring Boot jar：包含程序代码、**依赖库**、内嵌服务器、启动配置相关类。
- 普通 jar：主要包含程序类文件、资源文件和明确依赖的库。

**运行方式**

- Spring Boot jar：可直接通过`java - jar`命令运行，Spring Boot 会自动配置并启动内嵌服务器。
- 普通 jar：需在已有 Java 运行环境（如 Web 容器）中引用和运行。

**目录结构**

- Spring Boot jar：有独特结构，如依赖在`BOOT-INF/lib`，类文件在`BOOT-INF/classes`等。
- 普通 jar：遵循标准格式，类和资源直接放在根目录下。

## 程序发布（Maven build打包程序）

**eclipse****：** Run->Maven build

或者直接在项目的路径下面运行：mvn package

如果需要跳过单元测试可以使用：mvn package -DskipTests

启动命令：java -jar ./Application-0.0.1-SNAPSHOT.jar --server.port=8888 start

运行命令(输出到日志文件)：java -jar spring-boot01-1.0-SNAPSHOT.jar > log.file 2>&1 &

## [日志管理](https://blog.csdn.net/xudan1010/article/details/52890102)

在创建Spring Boot工程时，我们引入了spring-boot-starter，其中包含了spring-boot-starter-logging，该依赖内容就是Spring Boot默认的日志框架Logback，所以我们在引入log4j2之前，需要先排除该包的依赖，再引入log4j2的依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

> **注意：**spring boot 1.3版本支持log4j，在spring boot 1.4的版本中，就需要使用log4j2，否则会出现如下错误：Project build error: 'dependencies.dependency.version' for org.springframework.boot:spring-boot-starter-log4j:jar is missing.

## 定时任务

1 定时任务的开启，首先需要在项目的启动类上加入@EnableScheduling注解来启用定时任务。

2 在运行定时任务的类上加上@Component注解，表示该类可以作为定时任务类运行来交给spring运行。

3 在需要运行的方法上加上@Scheduled(fixedRate=1000)来定义执行的方式，这里表示每1秒运行一次。

## Spring Boot 2 [新特性](https://blog.csdn.net/yalishadaa/article/details/79400916)

- **默认connection pool变了**

默认的连接池已经由Tomcat切换到了HikariCP。

- **软件版本**

要求Tomcat最低版本为8.5、要求Hibernate最低版本为5.2、要求Jetty最低版本为9.4

- **Spring Framework 5.0**

Spring Boot 2.0 是建立在Spring Framework 5.0之上的（最低要求）

- **松绑定改善**

下面的属性最终都会被映射为 spring.jpa.databaseplatform=mysql

- spring.jpa.database-platform=mysql

- spring.jpa.databasePlatform=mysql

- spring.JPA.database_platform=mysql

## Spring Boot Admin

Spring Boot Admin 是一个管理和[监控Spring Boot 应用程序](https://www.cnblogs.com/shihaiming/p/8488939.html)的开源软件。每个应用都认为是一个客户端，通过HTTP或者使用 Eureka注册到admin server中进行展示，Spring Boot Admin UI部分使用AngularJs将数据展示在前端。

Spring Boot Admin 是一个针对spring-boot的actuator接口进行UI美化封装的监控工具。他可以：在列表中浏览所有被监控spring-boot项目的基本信息，详细的Health信息、内存信息、JVM信息、垃圾回收信息、各种配置信息（比如数据源、缓存列表和命中率）等，还可以直接修改logger的level。

springboot2.0使用admin可以参考[本文](https://www.jianshu.com/p/2b66433bd373)。1.5.x参考[该文](https://www.cnblogs.com/shihaiming/p/8488939.html)。

### Spring Boot Admin的使用

springboot-admin分为**客户端**和**服务器端**，使用服务

客户端即为自己的应用；

服务端则为引入了spring-boot-admin-starter-server依赖的springboot应用。

**客户端**

客户端需要引入spring-boot-admin-starter-client依赖；如果需要端点保护，则还需引入spring-boot-starter-security依赖，

并添加配置文件和SecurityConfig.java；

**application.properties**

```properties
server.port=8001
## 是否开启security 拦截
security.basic.enabled=true
security.user.name=root
security.user.password=root
## 应用名称
spring.application.name=demo
## 监控服务端地址
spring.boot.admin.url=http://192.168.1.101:9191
## client访问服务端端点的认证信息
spring.boot.admin.username=root
spring.boot.admin.password=root
## 服务端访问client端点的地址信息（本应用访问地址）
spring.boot.admin.client.service-url=http://192.168.1.164:8001
## 服务端访问client端点的认证信息
spring.boot.admin.client.metadata.user.name=${security.user.name}
spring.boot.admin.client.metadata.user.password=${security.user.password}
## 安全认证
management.security.enabled=true
#可在线查看日志
endpoints.logfile.enabled=true
```

**SecurityConfig.java**

```java
@Configuration
publicclassSecurityConfigextendsWebSecurityConfigurerAdapter{
    @Override
    protectedvoidconfigure(HttpSecurityhttp)throwsException{
        http.csrf().disable();//配置跨域请求
        http.httpBasic().and()
        .authorizeRequests().antMatchers("/actuator/**").authenticated()
        .anyRequest().permitAll();//配置springboot-admin-client
        http.csrf().ignoringAntMatchers("/**");//忽略拦截所有地址（顺序不可变）
    }
}
```

**服务端**

客户端需要引入spring-boot-admin-starter-server依赖；如果需要端点保护，则还需引入spring-boot-starter-security依赖，

并添加配置文件和SecurityConfig.java；

**application.properties**

```properties
server.port=8000
# 配置SBA Client连接的安全账号密码
security.user.name=root
security.user.password=root
#开启安全验证
security.basic.enabled=true
```

**SecurityConfig.java**

```java
@Configuration
publicclassSecurityConfigextendsWebSecurityConfigurerAdapter{
    @Override
    protectedvoidconfigure(HttpSecurityhttp)throwsException{
        //Pagewithloginformisservedas/login.htmlanddoesaPOSTon/login
        http.formLogin().loginPage("/login.html").loginProcessingUrl("/login").permitAll();
        //TheUIdoesaPOSTon/logoutonlogout
        http.logout().logoutUrl("/logout");
        //Theuicurrentlydoesn'tsupportcsrf
        http.csrf().disable();

        //Requestsfortheloginpageandthestaticassetsareallowed
        http.authorizeRequests()
        .antMatchers("/login.html","/**/*.css","/img/**","/third-party/**")
        .permitAll();
        //...andanyotherrequestneedstobeauthorized
        http.authorizeRequests().antMatchers("/**").authenticated();

        //EnablesothattheclientscanauthenticateviaHTTPbasicforregistering
        http.httpBasic();
    }
}
```

**参考：**

> https://blog.csdn.net/lqm1991/article/details/83788978
> 
> https://www.jianshu.com/p/921387db847e
> 
> https://www.jianshu.com/p/d29663c1bddd
> 
> http://codecentric.github.io/spring-boot-admin/1.5.0/#securing-spring-boot-admin

## README文件

一份好的README可以给人以项目全景[概览](https://mp.weixin.qq.com/s/M2FihtQmvMgX-o7JD8edzA)，可以使新人快速上手项目，可以降低沟通成本。同时，README应该简明扼要，条理清晰，建议包含以下方面：

- 项目简介：用一两句话简单描述该项目所实现的业务功能；
- 技术选型：列出项目的技术栈，包括语言、框架和中间件等；
- 本地构建：列出本地开发过程中所用到的工具命令；
- 领域模型：核心的领域概念，比如对于示例电商系统来说有Order、Product等；
- 测试策略：自动化测试如何分类，哪些必须写测试，哪些没有必要写测试；
- 技术架构：技术架构图；
- 部署架构：部署架构图；
- 外部依赖：项目运行时所依赖的外部集成方，比如订单系统会依赖于会员系统；
- 环境信息：各个环境的访问方式，数据库连接等；
- 编码实践：统一的编码实践，比如异常处理原则、分页封装等；
- FAQ：开发过程中常见问题的解答。

需要注意的是，README中的信息可能随着项目的演进而改变（比如引入了新的技术栈或者加入了新的领域模型），因此也是需要持续更新的。虽然我们知道，软件文档的一个痛点便是无法与项目实际进展保持同步，但是就README这点信息来讲，还是建议开发者们不要吝啬那一点点敲键盘的时间。

此外，除了保持README的持续更新，一些重要的架构决定可以通过示例代码的形式记录在代码库中，新开发者可以通过直接阅读这些示例代码快速了解项目的通用实践方式以及架构选择，请参考ThoughtWorks的技术雷达。