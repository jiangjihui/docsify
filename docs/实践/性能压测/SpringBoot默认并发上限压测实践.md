# Spring Boot 默认并发上限压测实践（JMeter 实测）

> 用一个最小 echo 服务 + JMeter 阶梯加压，实测 Spring Boot 默认配置下的并发请求上限：**默认 200 工作线程、单请求阻塞 500ms 时，吞吐封顶 ~390 RPS，拐点恰好在并发 200**；调大 `server.tomcat.threads.max=400` 后上限线性翻倍到 ~780 RPS。本文所有数据均为实测，服务代码、JMeter 脚本、运行命令完整可复制，跟着做即可复现。

**环境**：Windows 10 / Java 11 / Spring Boot 2.7.18 / Maven 3.9.9 / JMeter 5.6.3（CLI 非 GUI 模式）

## 1. 压测目标与思路

**目标**：摸清 Spring Boot（内嵌 Tomcat）在**不改任何配置**时能扛多少并发请求。

**思路**：

- 写一个 echo 接口，带 `delay` 参数（服务端 `Thread.sleep` 模拟业务阻塞，如 DB 查询、下游调用）
- 用 JMeter 从 50 → 100 → 200 → 300 → 500 逐档加压，观察吞吐与延迟的**拐点**
- 拐点出现的并发数 ≈ Tomcat 工作线程数（默认 `maxThreads=200`）
- 另跑一组 `delay=0` 的对照组，区分「线程池瓶颈」与「服务器本身瓶颈」

**为什么 delay 参数是关键设计**：真实业务的耗时几乎都阻塞在等下游（DB / Redis / HTTP）上，线程在阻塞期间被独占。`delay=500` 让每个请求占用一个工作线程 500ms，才能稳定复现「线程池占满 → 后续请求排队」的产线典型现象；`delay=0` 则几乎不占线程，测出来的是机器本身的处理能力。

## 2. 被测服务搭建

单模块 Maven 项目，共 1 个启动类 + 1 个 Controller，**刻意不配置任何 Tomcat 线程池参数——默认值就是被测对象**（Spring Boot 2.7 默认：`maxThreads=200`、`acceptCount=100`、NIO `maxConnections=8192`）。

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

`EchoController.java`：

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

`application.properties`（端口按需调整，本文 8080 被本机其它服务占用故用 18701）：

```properties
server.port=18701
```

启动与冒烟验证：

```bash
mvn spring-boot:run
curl -s "http://localhost:18701/api/echo?msg=world&delay=100"
# {"method":"GET","message":"world","delay":100,"timestamp":"..."} 约 100ms 后返回
```

## 3. JMeter 压测方案

### 3.1 参数化测试计划（jmx）

线程组与 delay 参数全部用 `__P()` 属性占位，一个 jmx 跑所有档位，无需 GUI：

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

### 3.2 CLI 运行命令

```bash
# 目录约定：loadtest/{echo-load-test.jmx, results/, reports/}
# 注意：-o 的报告目录必须不存在、其父目录必须存在，否则报错
mkdir -p results reports

HEAP="-Xms512m -Xmx1g" "D:/Programs/apache-jmeter-5.6.3/bin/jmeter.bat" \
  -n -t echo-load-test.jmx \
  -Jconcurrency=200 -Jramp=10 -Jduration=50 -Jdelay_ms=500 \
  -l results/c200.jtl -e -o reports/c200
```

- `-n` 非 GUI 模式；`-l` 输出原始 jtl；`-e -o` 生成 HTML 仪表盘报告
- `HEAP` 环境变量给 JMeter 自己的 JVM 调内存（客户端 1000 并发建议 `-Xmx1536m`）
- 每档跑 **10s ramp-up + 50s 稳态**，看 summary 输出尾部窗口取稳态 RPS

## 4. 第一轮：默认配置压测结果（delay=500ms）

| 并发 | 稳态 RPS | 平均延迟 | p90 | p99 | 错误率 |
|-----:|---------:|---------:|----:|----:|-------:|
| 50   | ~98      | ~512ms   | 516ms | 516ms | 0% |
| 100  | ~195     | ~513ms   | 515ms | 516ms | 0% |
| **200** | **~390** | ~512ms | 516ms | 516ms | 0% |
| 300  | ~390（不再涨） | ~739ms | 829ms | 845ms | 0% |
| 500  | ~400     | ~1193ms  | 1376ms | 1389ms | 0% |
| 对照组：500 并发 delay=0 | **10,490** | ~40ms | 57ms | 68ms | 0% |

**现象解读**：

1. 并发 ≤200：RPS 随并发线性增长（98 → 195 → 390），延迟稳定在 ~512ms，零排队
2. 并发 >200：**吞吐钉死在 ~390 RPS 不再上升**，延迟开始膨胀——300 并发 739ms、500 并发 1193ms，正好符合排队公式 `延迟 ≈ 并发数 ÷ 390 × 1000ms`
3. 全程零错误：超载不是「请求被拒绝」，而是「排队变慢」

## 5. 第二轮：maxThreads=400 调参对比

`application.properties` 加一行后重启服务重压：

```properties
server.tomcat.threads.max=400
```

| 并发 | 稳态 RPS | 平均延迟 | p90 | p99 | 错误率 | 对比默认配置 |
|-----:|---------:|---------:|----:|----:|-------:|------|
| 300  | ~584     | ~512ms   | 516ms | 516ms | 0% | 390→584，从排队变为不排队 |
| 500  | ~780     | ~640ms   | 678ms | 681ms | 0% | 400→780，吞吐 +95% |
| 800  | ~782     | ~1024ms  | 1091ms | 1101ms | 0% | 到顶，纯排队 |
| 1000 | ~780     | ~1281ms  | 1404ms | 1418ms | 0% | 平台期确认 |

**调参结论**：

1. **上限随 maxThreads 线性翻倍**：200 线程 → ~390 RPS，400 线程 → ~780 RPS，拐点同步从并发 200 移到 400
2. **c300 的反转最有说服力**：同样 300 并发，默认配置下是「超限排队」（390 RPS、延迟 +44%），调参后变为「轻装直行」（584 RPS、延迟 = 纯处理时间 512ms）——证明瓶颈唯一来源就是工作线程池大小
3. c800/c1000 上探确认平台期：吞吐钉死 ~780，延迟按 `并发 ÷ 780 秒` 膨胀，依然零错误

## 6. 原理分析

### 6.1 上限公式

```
吞吐上限（RPS） ≈ maxThreads × (1000 / 单请求阻塞毫秒数)
```

两轮实测完美吻合：`200 × 1000/500 = 400 ≈ 实测 390`；`400 × 1000/500 = 800 ≈ 实测 780`。反之，已知目标 RPS 与平均阻塞时长，可反推需要的线程数。

### 6.2 为什么超限了却零错误

Spring Boot 2.7 内嵌 Tomcat 用 NIO 连接器，`maxConnections` 默认 **8192**，远大于工作线程数。超过 200 个并发后：

- 多出来的 TCP 连接**照常被接受**（只要没到 8192），只是请求在线程池队列里等空闲线程
- `acceptCount=100` 只在连接数达到 `maxConnections` 之后才生效（此时新连接直接被拒）
- 所以本场景的「上限」表现为**延迟恶化而非请求失败**——压测时要盯延迟曲线，不能只看错误率

### 6.3 对照组证明瓶颈不是 Java

`delay=0` 时同样的 200 线程池、500 并发，冲到 **10,490 RPS**（延迟 40ms、零错误）。说明：

- Java/Tomcat 本身处理 HTTP 并不慢，瓶颈是「**阻塞式请求 × 平台线程池**」这个组合
- 平台线程阻塞的代价是线程被独占睡觉；线程池大小决定了「允许多少个请求同时在等」
- 这正是 Java 21 虚拟线程要解决的问题：阻塞不再独占平台线程，此类调参随之失效

## 7. 生产环境调参建议

1. **先看下游，再调 Tomcat**。把线程池调到 400，但 HikariCP 默认只有 10 个 DB 连接——瓶颈只是从 Tomcat 排队挪到连接池排队，RPS 不会变。正确顺序：确认下游能承载的并发 → 反推线程数（一般 ≤ 下游连接池总容量）→ 压测验证拐点
2. **常见产线值 200~400，不建议盲目上千**。每线程默认 ~1MB 栈，800 线程光栈就 800MB；且线程数上去后 CPU 上下文切换成本、下游压力同步上升
3. **过载时「快速失败」优于「慢慢排队」**。默认行为是延迟膨胀（500 并发时用户等 1.2s），产线应配连接/响应超时 + 熔断限流（如 Sentinel），让过载流量快速失败，保住整体可用性
4. **新项目考虑绕开这个问题**：Java 21 虚拟线程（Spring Boot 3.2+ `spring.threads.virtual.enabled=true`）或 WebFlux 响应式，让「阻塞独占线程」的前提不复存在

## 8. 复现指引

完整步骤速览：

1. 搭建第 2 节的服务（三个文件 + `mvn spring-boot:run`）
2. 保存第 3.1 节 jmx 为 `loadtest/echo-load-test.jmx`，`mkdir -p results reports`
3. 按第 3.2 节命令逐档执行（50/100/200/300/500），每档看 summary 尾部稳态值
4. 加 `server.tomcat.threads.max=400` 重启，重压 300/500/800/1000
5. 打开 `loadtest/reports/<档位>/index.html` 看 HTML 仪表盘（响应时间曲线、分位数、吞吐随时间变化）

**判断拐点的标志**：并发增加而稳态 RPS 不再增加、平均延迟按 `并发 ÷ 上限RPS` 比例膨胀——拐点前的最大并发即该配置的有效并发上限。
