# Spring Boot 默认并发上限压测实践

> 一个 Spring Boot 服务不改任何配置，能扛多少并发？用最小 echo 服务 + JMeter 阶梯加压实测：**默认 200 工作线程、单请求阻塞 500ms 时，吞吐封顶 ~390 RPS；线程翻倍，上限精确翻倍**。全文先讲结论与原理，可复制的服务代码与压测脚本见[附录](#附录复现材料)。

**环境**：Windows 10 / Java 11 / Spring Boot 2.7.18 / Maven 3.9.9 / JMeter 5.6.3（非 GUI 模式）

## 1. 结论

**默认配置的并发上限不是一个固定数字，而是由公式决定：**

> **吞吐上限（RPS）≈ maxThreads × (1000 ÷ 单请求阻塞毫秒数)**
>
> 实测验证：默认 200 线程 + 阻塞 500ms → **~390 RPS**，拐点在并发 200；线程翻倍到 400 → **~780 RPS**，拐点同步移到 400

### 上限由三个因素决定

| 因素 | 规律 | 实测证据 |
|------|------|---------|
| **工作线程数**（`server.tomcat.threads.max`，默认 200） | 线程翻倍，上限翻倍，拐点同步移动 | 200 线程 → 390 RPS；400 线程 → 780 RPS |
| **单请求阻塞时长**（DB/下游调用耗时） | 阻塞越长上限越低；无阻塞时此瓶颈消失 | delay=500ms → 390 RPS；delay=0 → **10,490 RPS** |
| **下游容量**（DB 连接池等） | 下游不足时，调大线程只是把排队挪位置 | HikariCP 默认仅 10 连接，常先于线程池成为瓶颈（常见经验，本文未实测） |

**超限的表现：不报错，只是变慢。** NIO 连接器 `maxConnections` 默认 8192，超额请求被接受后排在线程池队列等待，延迟按 `并发数 ÷ 上限` 线性膨胀（500 并发时排队 1.2 秒）。压测必须盯延迟曲线，不能只看错误率。

### 该怎么配置

1. **先测下游容量**：线程数不超过下游能同时承载的请求数（DB 连接池、依赖服务）
2. **按公式反推线程数**：`目标 RPS × 平均阻塞毫秒 ÷ 1000`；产线常见 200~400，盲目上千只烧内存（每线程 ~1MB 栈）
3. **配过载保护**：超时 + 熔断限流，快速失败优于让用户慢慢等
4. **能绕开就绕开**：Java 21 虚拟线程（Spring Boot 3.2+ `spring.threads.virtual.enabled=true`）让阻塞不再独占线程

> 各因素的展开分析见第 3 节（原理），配置建议的详细说明见第 4 节（生产调参）。

## 2. 实测现象

方法一句话：echo 接口带 `delay` 参数模拟业务阻塞，JMeter 从 50 到 1000 逐档加压找拐点（详见附录）。

### 2.1 为什么并发 300 时吞吐不涨了？

**默认配置（maxThreads=200，delay=500ms）：**

| 并发 | 稳态 RPS | 平均延迟 | p99 | 错误率 |
|-----:|---------:|---------:|----:|-------:|
| 50   | ~98      | ~512ms   | 516ms | 0% |
| 100  | ~195     | ~513ms   | 516ms | 0% |
| **200（拐点）** | **~390** | ~512ms | 516ms | 0% |
| 300  | ~390 不再涨 | ~739ms | 845ms | 0% |
| 500  | ~400     | ~1193ms  | 1389ms | 0% |
| 对照：500 并发 delay=0 | **10,490**（全程均值） | ~40ms | 68ms | 0% |

三个现象：

1. **并发 ≤200**：RPS 随并发线性增长（98 → 195 → 390），延迟稳定 ~512ms，零排队
2. **并发 >200**：吞吐钉死 ~390，延迟按 `并发 ÷ 390 × 1000ms` 膨胀（300 并发 739ms、500 并发 1193ms）
3. **全程零错误**：超载的表现是「排队变慢」，不是「请求被拒」

### 2.2 调大线程池，上限怎么变？

`server.tomcat.threads.max=400` 重启重压：

| 并发 | 稳态 RPS | 平均延迟 | p99 | 错误率 | 说明 |
|-----:|---------:|---------:|----:|-------:|------|
| 300  | ~584     | ~512ms   | 516ms | 0% | 从排队反转为不排队 |
| 500  | **~780** | ~640ms   | 681ms | 0% | 吞吐 +95% |
| 800  | ~782     | ~1024ms  | 1101ms | 0% | 到顶，纯排队 |
| 1000 | ~780     | ~1281ms  | 1418ms | 0% | 平台期确认 |

**上限随线程数线性翻倍**（390 → 780），拐点同步从并发 200 移到 400。最有说服力的是 **c300 的反转**：同样 300 并发，默认配置下超限排队（390 RPS、延迟 +44%），调参后轻装直行（584 RPS、延迟 = 纯处理时间 512ms）——证明瓶颈唯一来源就是工作线程池大小。

## 3. 原理

### 3.1 上限公式从哪来

每个请求占用一个工作线程「阻塞 500ms + 处理」，线程 1 秒最多完成 `1000/500 = 2` 个请求。200 个线程 → `200 × 2 = 400 RPS`。两轮实测吻合：`200 线程 → 390`、`400 线程 → 780`。反过来，已知目标 RPS 与平均阻塞时长，可反推所需线程数。

### 3.2 超限为什么零报错

Tomcat NIO 连接器 `maxConnections` 默认 **8192**，远大于工作线程数：

- 超出线程数的 TCP 连接**照常被接受**，只是请求在线程池队列等空闲线程
- `acceptCount=100` 只在连接数达到 `maxConnections` 后才生效（新连接直接被拒）
- 所以上限的表现是**延迟恶化而非请求失败**——压测要盯延迟曲线，不能只看错误率

### 3.3 对照组：瓶颈不是 Java

`delay=0`、同样 200 线程池、500 并发 → **10,490 RPS**（延迟 40ms）。说明：

- Java/Tomcat 处理 HTTP 并不慢，瓶颈是「**阻塞式请求 × 平台线程池**」的组合
- 线程阻塞期间被独占「睡觉」，线程池大小决定「允许多少个请求同时在等」
- 这正是 Java 21 虚拟线程要解决的问题：阻塞不再独占平台线程，此类调参随之失效

## 4. 生产调参建议

| 建议 | 要点 |
|------|------|
| **先看下游，再调 Tomcat** | 线程池调到 400 但 HikariCP 默认只有 10 个 DB 连接，瓶颈只是挪了个地方。顺序：下游容量 → 反推线程数（≤ 下游连接池总容量）→ 压测验证拐点 |
| **产线常见值 200~400** | 每线程 ~1MB 栈，800 线程光栈就 800MB；线程多了上下文切换、下游压力同步上升 |
| **快速失败优于慢慢排队** | 默认行为是延迟膨胀（用户干等 1.2s）。配超时 + 熔断限流（如 Sentinel），让过载流量快速失败，保住整体可用性 |
| **新项目可直接绕开** | Java 21 虚拟线程（Spring Boot 3.2+ `spring.threads.virtual.enabled=true`）或 WebFlux，让「阻塞独占线程」的前提不复存在 |

## 5. 延伸：大厂为何仍用 Java

看到「默认才 400 RPS」很容易得出「Java 并发不行」——这把**模型默认参数**误当成了**语言能力**。

- **不是 Java 慢**：同样服务 `delay=0` 打到 10,490 RPS；差成绩是平台线程（每个 ~1MB 栈）在陪阻塞调用睡觉。Node.js 用事件循环回避此问题（CPU 密集卡死整个进程）、Go 用 goroutine 解决（阻塞极便宜）、Java 21 虚拟线程才补齐
- **大厂瓶颈不在单机**：百万级 RPS 靠「网关 → 集群 → 缓存 → 分库分表」水平扩展；第一瓶颈几乎永远在下游（DB/缓存/依赖服务），单机 5000 RPS 而 DB 只接 500 没有意义
- **选型算总账**：生态（Spring 全家桶 + 中间件 + Dubbo，新业务一周上线）、人才（几十万 Java 工程师）、工程性（强类型 + JIT/GC 二十年优化，百万行代码几百人协作可维护）——单机性能只是其中一项
- **大厂自己也分化**：核心电商仍是 Java，基础设施/云原生大量用 Go，异步化（Netty/Reactor）用得很早——业务生态层用 Java、性能敏感层用别的补

> 一句话：**单机并发上限是部署密度问题（多加一台机器的事），生态、人才、可维护性才是选型主导项**。压测看到的「400 RPS」真实含义是线程在睡觉、CPU 在空转——账记在线程模型头上，由虚拟线程/响应式来还。

## 附录：复现材料

只想看结论到这里就够了；以下为动手复现用的完整代码与脚本，全部可直接复制。

### A.1 被测服务

单模块 Maven 项目：1 个启动类 + 1 个 Controller。**刻意不配置任何 Tomcat 参数——默认值就是被测对象**（Spring Boot 2.7 默认：`maxThreads=200`、`acceptCount=100`、NIO `maxConnections=8192`）。

`pom.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>echo-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

`EchoController.java`（`delay` 参数用 `Thread.sleep` 模拟业务阻塞，是整个实验的关键设计）：

```java
package com.example.echo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.time.Instant;
import java.util.LinkedHashMap;
import java.util.Map;

@RestController
public class EchoController {

    @GetMapping("/api/echo")
    public Map<String, Object> echoGet(
            @RequestParam(required = false) String msg,
            @RequestParam(defaultValue = "0") long delay) throws InterruptedException {
        return echo("GET", msg, delay);
    }

    @PostMapping("/api/echo")
    public Map<String, Object> echoPost(
            @RequestBody String body,
            @RequestParam(defaultValue = "0") long delay) throws InterruptedException {
        return echo("POST", body, delay);
    }

    private Map<String, Object> echo(String method, String message, long delay) throws InterruptedException {
        Thread.sleep(Math.max(0, delay));
        Map<String, Object> result = new LinkedHashMap<>();
        result.put("method", method);
        result.put("message", message);
        result.put("delay", delay);
        result.put("timestamp", Instant.now().toString());
        return result;
    }
}
```

`application.properties`（端口按需调整，本文 8080 被占用故用 18701）：

```properties
server.port=18701
```

启动与冒烟：

```bash
mvn spring-boot:run
curl -s "http://localhost:18701/api/echo?msg=world&delay=100"
# {"method":"GET","message":"world","delay":100,"timestamp":"..."} 约 100ms 后返回
```

### A.2 JMeter 测试计划

线程数、时长、delay 全部用 `__P()` 属性占位——一个 jmx 跑所有档位，无需 GUI。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.6.3">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Echo Load Test" enabled="true">
      <stringProp name="TestPlan.comments">Staged load test. Params: -Jconcurrency -Jduration -Jramp -Jdelay_ms</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.tearDown_on_shutdown">true</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments" guiclass="ArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
      <stringProp name="TestPlan.user_define_classpath"></stringProp>
    </TestPlan>
    <hashTree>
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Echo Users" enabled="true">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController" guiclass="LoopControlPanel" testclass="LoopController" testname="Loop Controller" enabled="true">
          <boolProp name="LoopController.continue_forever">true</boolProp>
          <intProp name="LoopController.loops">-1</intProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">${__P(concurrency,200)}</stringProp>
        <stringProp name="ThreadGroup.ramp_time">${__P(ramp,10)}</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <stringProp name="ThreadGroup.duration">${__P(duration,50)}</stringProp>
        <stringProp name="ThreadGroup.delay">0</stringProp>
        <boolProp name="ThreadGroup.same_user_on_next_iteration">true</boolProp>
      </ThreadGroup>
      <hashTree>
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="GET /api/echo" enabled="true">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments" guiclass="HTTPArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
            <collectionProp name="Arguments.arguments">
              <elementProp name="msg" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">false</boolProp>
                <stringProp name="Argument.value">load-test</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
                <boolProp name="HTTPArgument.use_equals">true</boolProp>
                <stringProp name="Argument.name">msg</stringProp>
              </elementProp>
              <elementProp name="delay" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">false</boolProp>
                <stringProp name="Argument.value">${__P(delay_ms,500)}</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
                <boolProp name="HTTPArgument.use_equals">true</boolProp>
                <stringProp name="Argument.name">delay</stringProp>
              </elementProp>
            </collectionProp>
          </elementProp>
          <stringProp name="HTTPSampler.domain">localhost</stringProp>
          <stringProp name="HTTPSampler.port">18701</stringProp>
          <stringProp name="HTTPSampler.protocol">http</stringProp>
          <stringProp name="HTTPSampler.path">/api/echo</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
          <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
          <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
          <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
          <boolProp name="HTTPSampler.DO_MULTIPART_POST">false</boolProp>
          <boolProp name="HTTPSampler.postBodyRaw">false</boolProp>
          <stringProp name="HTTPSampler.embedded_url_re"></stringProp>
          <stringProp name="HTTPSampler.connect_timeout">5000</stringProp>
          <stringProp name="HTTPSampler.response_timeout">60000</stringProp>
        </HTTPSamplerProxy>
        <hashTree/>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

### A.3 运行命令与步骤

```bash
# 目录约定：loadtest/{echo-load-test.jmx, results/, reports/}
# 注意：-o 的报告目录必须不存在、其父目录必须存在，否则报错
mkdir -p results reports

HEAP="-Xms512m -Xmx1g" "D:/Programs/apache-jmeter-5.6.3/bin/jmeter.bat" \
  -n -t echo-load-test.jmx \
  -Jconcurrency=200 -Jramp=10 -Jduration=50 -Jdelay_ms=500 \
  -l results/c200.jtl -e -o reports/c200
```

- `-n` 非 GUI；`-l` 原始 jtl；`-e -o` 生成 HTML 仪表盘
- `HEAP` 给 JMeter 自身 JVM 调内存（客户端 1000 并发建议 `-Xmx1536m`）
- 每档 **10s ramp-up + 50s 稳态**（对照组 delay=0 为 30s），取 summary 尾部窗口的稳态 RPS

完整流程：

1. 搭建 A.1 节服务（三个文件 + `mvn spring-boot:run`）
2. 保存 A.2 节 jmx 为 `loadtest/echo-load-test.jmx`
3. 逐档执行：默认配置压 50/100/200/300/500，加 `server.tomcat.threads.max=400` 重启后压 300/500/800/1000，另跑一组 `delay=0` 对照
4. 打开 `loadtest/reports/<档位>/index.html` 看仪表盘（响应时间曲线、分位数、吞吐随时间变化）

> **判断拐点的标志**：并发增加而稳态 RPS 不再增加、平均延迟按 `并发 ÷ 上限RPS` 比例膨胀。拐点前的最大并发即该配置的有效并发上限。
