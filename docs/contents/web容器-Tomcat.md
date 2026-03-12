## 概述

### 什么是 Tomcat

Tomcat 是 Apache 软件基金会（Apache Software Foundation）的 Jakarta 项目中的一个核心项目，由 Apache、Sun 和其他一些公司及个人共同开发而成。由于有了 Sun 的参与和支持，最新的 Servlet 和 JSP 规范总是能在 Tomcat 中得到体现。

Tomcat 是一种轻量级的 Web 应用服务器（Servlet 容器），完全支持 Java Servlet、JavaServer Pages（JSP）、Java EL 和 WebSocket 规范，是开发和部署 Java Web 应用的常用选择。

### Tomcat 架构

Tomcat 的核心架构包括以下组件：

- **Server**：顶级容器，代表整个 Tomcat 实例
- **Service**：包含一个 Engine 和一个或多个 Connector
- **Engine**：处理请求的容器，一个 Service 只有一个 Engine
- **Host**：表示虚拟主机
- **Context**：表示 Web 应用
- **Connector**：负责处理网络连接
- **Executor**：线程池

### Spring Boot 与 Tomcat

Spring Boot 默认使用 Tomcat 作为嵌入式 Web 容器。通过 spring-boot-starter-web 依赖自动引入 Tomcat。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 版本对应关系

| Spring Boot 版本 | Tomcat 版本 | Servlet API | 最低 Java 版本 |
|-----------------|-------------|-------------|---------------|
| 1.x | 8.x | javax.servlet | Java 7 |
| 2.x | 9.x | javax.servlet | Java 8 |
| 3.x | 10.x | jakarta.servlet | Java 17 |
| 3.1+ | 11.x | jakarta.servlet | Java 21 |

> **注意**：Spring Boot 3.x 之后升级到 Jakarta EE 9+（jakarta.servlet），与之前的 javax.servlet 不兼容。

#### 替换嵌入式容器

可以替换为其他嵌入式容器：

```xml
<!-- 排除 Tomcat，添加 Undertow -->
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

#### Spring Boot 中的请求限制

Spring Boot 默认配置下，Tomcat 的请求处理能力由以下参数决定：

- **最大线程数（`maxThreads`）**：200
  - 当并发请求数超过 200 时，新请求会被放入队列
- **队列容量（`acceptCount`）**：100
  - 当队列满时，后续请求会被拒绝（返回 503 Service Unavailable）

**总拒绝阈值 = 最大线程数 + 队列容量 = 200 + 100 = 300**

即当并发请求数达到 300 时，服务开始拒绝新请求。

```properties
# 通用优化配置
server.tomcat.max-threads=300          # 最大线程数
server.tomcat.min-spare-threads=50     # 最小空闲线程数
server.tomcat.accept-count=200         # 队列容量
server.tomcat.connection-timeout=60000 # 连接超时时间（毫秒）
```

---

## 核心概念

### ServletContext

ServletContext 是 Web 容器的上下文对象，代表当前 Web 应用。每个 Web 应用在部署时，容器会为其创建一个 ServletContext 对象，并被该应用内的所有 Servlet 共享。

#### 特性

- **创建时机**：Web 容器启动时为每个 Web 应用程序创建
- **共享范围**：被 Web 应用内的所有程序共享
- **生命周期**：随 Web 应用关闭、Tomcat 关闭或 Web 应用 reload 时销毁

#### 获取方式

```java
// 通过 ServletConfig 获取
ServletContext context = getServletConfig().getServletContext();

// 通过继承 HttpServlet 直接获取
ServletContext context = this.getServletContext();
```

#### 应用场景

1. **网站计数器** - 记录网站访问量
2. **在线用户显示** - 维护当前在线用户列表
3. **数据缓存** - 将不经常更改的数据缓存到内存，减少磁盘 I/O
4. **跨 Servlet 通信** - 通过 setAttribute 和 getAttribute 在不同 Servlet 间共享数据

### 线程池

Tomcat 使用线程池来处理并发请求，这与 Java 标准线程池有所不同。

#### 线程池参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| 核心线程数 | 10 | 线程池保持的最小活跃线程数 |
| 最大线程数 | 200 | 线程池可创建的最大线程数 |
| 空闲超时 | 60秒 | 线程空闲超时回收时间 |

#### 工作原理

1. 请求到达 Tomcat 时，从线程池分配一个线程处理
2. 若无空闲线程且未达最大线程数，则创建新线程
3. 请求处理完成后，线程返回线程池等待下一个请求
4. 空闲线程超过超时时间可能被回收

> **注意**：Tomcat 的线程池主要用于处理网络请求，与 Java 标准线程池的队列机制有所不同。

---

## 请求处理机制

### 资源类型

在 Tomcat 中，所有资源访问都通过 Servlet 实现。Tomcat 将请求资源分为三类：

| 类型 | 处理 Servlet | 说明 |
|------|--------------|------|
| 静态资源 | DefaultServlet | 处理 CSS、HTML、JS、图片等 |
| Servlet | InvokerServlet | 处理用户注册的 Servlet |
| JSP | JspServlet | 处理 JSP 页面 |

### 处理流程

Tomcat 通过 Mapper 类进行路由判断，流程如下：

1. 首先判断是否是 Servlet
2. 然后判断是否是 JSP
3. 最后由 DefaultServlet 处理静态资源

#### DefaultServlet 处理流程

```
请求 → doPost/doGet → serveResource → copy → 输出流
```

DefaultServlet 最终通过 `copy()` 方法将静态资源的输入流读取并写入输出流，完成资源响应。

---

## 运行模式

Tomcat 支持三种 I/O 模式：

### BIO（阻塞 I/O）

- **全称**：Blocking I/O
- **特点**：传统的 Java I/O 操作，阻塞式处理
- **性能**：三种模式中最低
- **适用场景**：低并发、简单应用

### NIO（非阻塞 I/O）

- **全称**：New I/O / Non-blocking I/O
- **特点**：基于 Java.nio 包，支持非阻塞操作
- **性能**：优于 BIO，适合高并发场景
- **适用场景**：高并发 Web 应用（Tomcat 8+ 默认模式）

> **说明**：NIO 优化了网络 I/O 读写，但如果系统瓶颈不在网络 I/O（如每次读取字节数较少），BIO 和 NIO 性能差异不明显。

### APR（Apache Portable Runtime）

- **全称**：Apache Portable Runtime
- **特点**：通过 JNI 调用 Apache 核心动态链接库
- **性能**：最高性能，适合高并发生产环境
- **适用场景**：对静态文件处理性能要求高的场景

> **注意**：
> - Tomcat 7 默认 BIO 模式，需要手动优化
> - Tomcat 8+ 默认 NIO 模式，无需修改
> - Windows 版 Tomcat 默认携带 tcnative-1.dll，默认使用 APR 模式

### 配置运行模式

修改 `conf/server.xml` 中的 Connector 节点：

```xml
<!-- BIO 模式 -->
<Connector port="8080" protocol="org.apache.coyote.http11.Http11Protocol" />

<!-- NIO 模式 -->
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" />

<!-- APR 模式 -->
<Connector port="8080" protocol="org.apache.coyote.http11.Http11AprProtocol" />
```

### AJP 协议

当使用 Nginx + Tomcat 集群时，建议禁用 AJP 协议：

```xml
<!-- 注释掉 AJP 连接器 -->
<!-- <Connector protocol="AJP/1.3" address="::1" port="8009" redirectPort="8443" /> -->
```

---

## 配置优化

### 连接器配置

打开 `conf/server.xml`，配置 Connector 节点：

```xml
<Connector
    port="8080"
    protocol="HTTP/1.1"
    URIEncoding="UTF-8"
    minSpareThreads="25"
    maxSpareThreads="75"
    maxThreads="300"
    acceptCount="200"
    connectionTimeout="20000"
    enableLookups="false"
    disableUploadTimeout="true"
    compression="on"
    compressionMinSize="2048"
    compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"
    redirectPort="8443" />
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `URIEncoding` | URL 编码格式，建议设置为 UTF-8 |
| `minSpareThreads` | 最小备用线程数，Tomcat 启动时初始化 |
| `maxSpareThreads` | 最大空闲线程数，超出时回收 |
| `maxThreads` | 最大线程数，即最大并发数 |
| `acceptCount` | 等待队列大小，满队列时拒绝连接 |
| `connectionTimeout` | 连接超时时间（毫秒） |
| `enableLookups` | 是否开启 DNS 反向查询，建议关闭 |
| `disableUploadTimeout` | 禁用上传超时 |
| `compression` | 启用 gzip 压缩 |
| `compressionMinSize` | 启用压缩的最小文件大小 |
| `compressableMimeType` | 支持压缩的 MIME 类型 |

### Spring Boot 配置

在 `application.properties` 中配置：

```properties
# 线程池配置
server.tomcat.max-threads=300
server.tomcat.min-spare-threads=50
server.tomcat.accept-count=200
server.tomcat.connection-timeout=60000

# 连接配置
server.tomcat.max-connections=10000
server.tomcat.accept-count=100

# 压缩配置
server.compression.enabled=true
server.compression.mime-types=text/html,text/xml,text/javascript,text/css,text/plain
```

或在 `application.yml` 中：

```yaml
server:
  tomcat:
    max-threads: 300
    min-spare-threads: 50
    accept-count: 200
    connection-timeout: 60000
    max-connections: 10000
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/javascript,text/css,text/plain
```

---

## JVM调优

### 启动参数优化

Tomcat 的启动参数在 `bin/catalina.sh`（Linux）或 `bin/catalina.bat`（Windows）中配置。

#### JDK 8+ 推荐配置

```bash
export JAVA_OPTS="-server \
-Xms2G -Xmx2G \
-Xmn512m \
-XX:MetaspaceSize=512M \
-XX:MaxMetaspaceSize=512M \
-XX:+UseG1GC \
-XX:MaxGCPauseMillis=200 \
-XX:+HeapDumpOnOutOfMemoryError \
-verbose:gc \
-XX:+PrintGCDetails \
-XX:+PrintGCTimeStamps \
-XX:+PrintGCDateStamps \
-Xloggc:/appl/gc.log \
-Djava.awt.headless=true"
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `-server` | 生产环境必须启用，使用 Server JVM |
| `-Xms` `-Xmx` | 堆初始/最大内存，建议设为相同值避免动态调整 |
| `-Xmn` | 年轻代大小，推荐为堆内存的 3/8 |
| `-XX:MetaspaceSize` | 元空间初始大小（JDK 8+） |
| `-XX:MaxMetaspaceSize` | 元空间最大大小（JDK 8+） |
| `-XX:+UseG1GC` | 使用 G1 垃圾收集器（JDK 9+ 默认） |
| `-XX:MaxGCPauseMillis` | 最大 GC 暂停时间目标 |
| `-XX:+HeapDumpOnOutOfMemoryError` | 内存溢出时生成堆转储 |
| `-XX:+PrintGCDetails` | 打印详细 GC 日志 |
| `-XX:+PrintGCTimeStamps` | 打印 GC 时间戳 |
| `-Xloggc` | GC 日志文件路径 |
| `-Djava.awt.headless=true` | 无头模式，用于服务器端图形处理 |

#### JDK 17+ 额外参数

```bash
export JAVA_OPTS="-server \
-Xms2G -Xmx2G \
-XX:+UseZGC \
-XX:+HeapDumpOnOutOfMemoryError \
-XX:HeapDumpPath=/appl/heapdump.hprof \
-Djava.awt.headless=true \
--add-opens=java.base/java.lang=ALL-UNNAMED \
--add-opens=java.base/java.util=ALL-UNNAMED"
```

### 内存设置建议

| 服务器内存 | 堆内存 | 年轻代 | 元空间 |
|-----------|--------|--------|--------|
| 4GB | 2GB | 512MB - 768MB | 256MB - 512MB |
| 8GB | 4GB | 1GB - 1.5GB | 512MB - 1GB |
| 16GB | 8GB | 2GB - 3GB | 1GB |

---

## Gzip压缩

### 配置方式

修改 `conf/server.xml`：

```xml
<Connector port="8080" protocol="HTTP/1.1"
   connectionTimeout="20000"
   redirectPort="8443"
   compression="on"
   compressionMinSize="2048"
   compressableMimeType="text/html,text/xml,text/javascript,application/javascript,text/css,text/plain,text/json,application/json" />
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `compression` | 压缩开关：on/off/force |
| `compressionMinSize` | 启用压缩的最小文件大小（字节），默认 2048 |
| `compressableMimeType` | 支持压缩的 MIME 类型列表 |
| `noCompressionUserAgents` | 不进行压缩的用户代理（可选） |

### Spring Boot 配置

```properties
server.compression.enabled=true
server.compression.mime-types=text/html,text/xml,text/javascript,text/css,text/plain,application/json
server.compression.min-response-size=2048
```

### 验证压缩

```bash
# 使用 curl 验证
curl -I -H "Accept-Encoding: gzip" http://localhost:8080/

# 查看响应头
# Content-Encoding: gzip 表示已启用压缩
```

---

## 编码设置

### server.xml 配置

```xml
<Connector port="8080" protocol="HTTP/1.1"
    redirectPort="8443"
    URIEncoding="UTF-8" />
```

### 启动参数配置

在 `bin/catalina.sh` 中添加：

```bash
JAVA_OPTS="${JAVA_OPTS} -Dfile.encoding=UTF-8 -Duser.timezone=Asia/Shanghai"
```

### Spring Boot 配置

```properties
server.tomcat.uri-encoding=UTF-8
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
```

---

## 部署方式

### 第一种：修改 server.xml

在 `conf/server.xml` 的 `<Host>` 节点中添加 Context：

```xml
<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context path="/hello"
        docBase="D:/myapp/webroot"
        debug="0"
        privileged="true" />
</Host>
```

- `path`：访问路径
- `docBase`：应用实际路径
- `debug`：调试级别
- `privileged`：是否拥有特权

### 第二种：创建 Context 配置文件

在 `conf/Catalina/localhost` 目录下创建 XML 文件（文件名即为访问路径）：

```xml
<!-- conf/Catalina/localhost/hello.xml -->
<Context path="/hello"
    docBase="D:/myapp/webroot"
    debug="0"
    privileged="true" />
```

**优点**：
- 无需修改 server.xml
- 可隐藏真实项目名称
- 支持热部署

### 第三种：部署到 webapps 目录

将 war 包或解压后的应用拷贝到 `webapps` 目录：

```bash
# 方式一：直接拷贝
cp -r /myapp /path/to/tomcat/webapps/

# 方式二：部署 war 包
cp myapp.war /path/to/tomcat/webapps/
```

Tomcat 会自动解压和部署。

### 第四种：Tomcat Manager

使用 Tomcat 管理界面部署：

1. 访问 `http://localhost:8080/manager/html`
2. 在 "WAR file to deploy" 部分上传 war 包

---

## 常见问题

### 1. 启动报错：端口被占用

```bash
# 查找占用端口的进程
netstat -ano | findstr 8080

# Windows 结束进程
taskkill /PID <PID> /F

# Linux 结束进程
kill -9 <PID>
```

### 2. 内存溢出（OOM）

```properties
# 添加 JVM 参数
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/heapdump.hprof
```

分析堆转储文件定位问题。

### 3. 响应缓慢

- 检查线程池配置是否合理
- 检查是否存在数据库连接泄漏
- 检查是否启用 Gzip 压缩
- 查看 GC 日志分析是否频繁 Full GC

### 4. 部署后 404

- 确认 war 包名称与访问路径一致
- 检查 web.xml 配置是否正确
- 确认应用已成功启动，查看日志

### 5. JSP 无法解析

确认 web.xml 中已配置 JSP Servlet：

```xml
<servlet>
    <servlet-name>jsp</servlet-name>
    <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>jsp</servlet-name>
    <url-pattern>*.jsp</url-pattern>
</servlet-mapping>
```

---

## 参考资料

- [Apache Tomcat 官方文档](https://tomcat.apache.org/)
- [Spring Boot 官方文档](https://spring.io/projects/spring-boot)
- [Tomcat 架构详解](https://tomcat.apache.org/tomcat-10.1-doc/architecture/overview.html)