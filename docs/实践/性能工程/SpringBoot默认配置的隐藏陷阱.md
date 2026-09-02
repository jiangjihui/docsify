# Spring Boot 默认配置的隐藏陷阱

> Spring Boot 的「开箱即用」代价是：大量默认值在低负载下毫无存在感，压力一上来才现形。本文是压测/上线前的检查清单：每条一个默认值、什么时候咬人、怎么配、怎么验证。与 [SpringBoot默认并发上限压测](SpringBoot默认并发上限压测实践.md) 配套阅读——那篇讲怎么测，这篇讲测之前该知道什么。

**基线版本**：Spring Boot 2.7.18。3.x 的差异在各条目内以「3.x」标注（配置属性名与默认值均已经 2.7.18 官方元数据核对）。

## 速查表

| 分组 | 陷阱 | 默认值 | 咬人场景 | 关键配置 |
|------|------|--------|---------|---------|
| 请求链路 | Tomcat 线程池 | maxThreads=200 | 阻塞型接口并发 >200 | `server.tomcat.threads.max` |
| 请求链路 | HikariCP 连接池 | 10 | 并发请求数远超 DB 连接数 | `spring.datasource.hikari.maximum-pool-size` |
| 请求链路 | connectionTimeout | 30s | 连接池耗尽时干等 30s 才报错 | `spring.datasource.hikari.connection-timeout` |
| 容器与部署 | 优雅停机 | immediate（关闭） | K8s 滚动更新偶发 502 | `server.shutdown: graceful` |
| 容器与部署 | JVM 堆 | 容器内存 1/4 | OOMKilled 或内存浪费 | `MaxRAMPercentage` |
| 异步与观测 | @Async 线程池 | 随版本漂移 | 高并发异步任务线程爆炸 | 自定义 `ThreadPoolTaskExecutor` |
| 异步与观测 | 监控指标 | 无 | 线上瓶颈只能靠日志猜 | actuator + micrometer |
| 连接细节 | KeepAlive 请求数 | 100 | 长连接周期性断开、重连毛刺 | `server.tomcat.max-keep-alive-requests` |
| 连接细节 | maxSwallowSize | 2MB | 大请求体被静默掐断 | `server.tomcat.max-swallow-size` |

## 请求链路的排队墙

同一请求路径上三堵墙，从外到内：Tomcat 线程池 → DB 连接池 → 连接等待超时。瓶颈永远在最窄的那堵，且越靠内越静默。

### Tomcat 线程池：maxThreads=200

阻塞型接口（慢 SQL、下游调用）的吞吐天花板 = `200 × (1000/单请求阻塞ms)`。并发超过 200 后不报错、只排队，延迟按 `并发 ÷ 上限` 膨胀。

- **什么时候咬人**：单请求阻塞 500ms 时 200+ 并发即触顶（吞吐钉死 ~390 RPS，实测数据见 [压测实践](SpringBoot默认并发上限压测实践.md)）
- **怎么配**：先看下游容量再调线程数（≤ 下游可承载并发），单调 Tomcat 只是挪排队位置
- **怎么验证**：压测加压过程中看 `tomcat.threads.busy` 指标是否长期贴住 200——贴住即线程池瓶颈实锤

### HikariCP 连接池：默认只有 10

Tomcat 200 线程 vs Hikari 10 连接——4:1 的错配。高并发时 190 个请求在等连接，且等满 30s（connectionTimeout）才抛异常，用户感知是「接口极慢」而非报错。

- **什么时候咬人**：DB 单查询 200ms 时，10 连接供给上限 = 50 TPS——压测吞吐恰卡在这种位置时优先怀疑它
- **怎么配**：连接数 ≈ 目标 TPS × 单查询平均耗时（Little's law）。注意 DB 实例总连接上限是所有服务共享的，留 30~50% 余量
- **怎么验证**：看 `hikaricp.connections.pending` 指标——大于 0 持续出现即连接池不足；恒为 0 而 TPS 上不去则瓶颈在 DB 内部或查询本身
- **已踩坑案例**：[数据库锁表致连接池耗尽](../产线故障复盘/md/连接超时/数据库锁表致连接池耗尽.md)

### connectionTimeout：干等 30 秒

连接池耗尽时 HikariCP 等 `connectionTimeout`（默认 30s）才放弃——而上游（网关/客户端）超时往往只有几秒，用户早就超时走了，服务端还在傻等。

```yaml
spring:
  datasource:
    hikari:
      connection-timeout: 5000
```

宁可 5 秒快速失败，别 30 秒慢慢等。

## 容器与部署

### 优雅停机：默认 immediate

K8s 滚动更新发 SIGTERM 后，Spring Boot 默认立即开始关闭，排队中的请求被切断——偶发 502，极难排查（问题窗口只在滚动更新瞬间出现）。

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

配套 K8s 侧：`preStop sleep 10` + `terminationGracePeriodSeconds` ≥ 优雅停机窗口 + 排空时间。

> 3.x 变化：Spring Boot 3.0 起 `server.shutdown` 默认已是 `graceful`——新项目不用配，2.7 必配。

- **怎么验证**：滚动更新期间观察 access log 有无 5xx；日志应出现 `Commencing graceful shutdown` 字样

### JVM 堆：容器内存的 1/4

不显式设置时，JVM 按容器内存的 1/4 分配堆。容器 limit 512MB 时堆只有 128MB，峰值直接 OOMKilled；limit 8GB 时堆 2GB 剩 6GB 闲着，浪费资源。

```dockerfile
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0"
```

容器内推荐用 `MaxRAMPercentage`（如 75%）而非固定 `-Xmx`：前者随 limit 自动缩放，且留出元空间/线程栈/堆外内存的余量。

- **怎么验证**：`kubectl exec <pod> -- java -XX:+PrintFlagsFinal -version 2>/dev/null | grep MaxHeapSize`，或看启动日志 GC 配置行

## 异步与观测

### @Async 线程池：随版本漂移

`@Async` 不显式指定 executor 时，Spring 老版本用 `SimpleAsyncTaskExecutor`——**每任务新建线程不复用**，高并发异步任务线程数失控直接把 Pod 打挂；较新版本改为复用 `applicationTaskExecutor`，但队列容量仍是小默认值，高吞吐下任务堆积。

```java
@Configuration
public class AsyncConfig {
    @Bean("asyncExecutor")
    public ThreadPoolTaskExecutor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(8);
        executor.setMaxPoolSize(32);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-");
        return executor;
    }
}
```

使用时 `@Async("asyncExecutor")` 显式指定。

- **怎么验证**：压测异步场景时看线程数（`jstack` / `kubectl top pod`）——线程数随请求数线性增长 = 中招

### 监控指标：默认裸奔

没有 metrics 时，瓶颈定性只能靠推理——本文所有「怎么验证」都依赖这一条。压测报告里「TPS 上不去」的归因（线程池？连接池？DB？）若有 HikariCP active/pending 指标，当场闭环，不用事后猜。

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```

重点暴露：`hikaricp.connections.active/pending`、`tomcat.threads.busy/current`、JVM GC 指标。

- **怎么验证**：`curl localhost:8080/actuator/metrics/hikaricp.connections.pending` 能返回数值即生效

## 连接细节

### KeepAlive：100 请求后断开

`server.tomcat.max-keep-alive-requests` 默认 100——客户端长连接每 100 个请求被服务端主动断开一次。

- **什么时候咬人**：LB 后端不均匀时表现为偶发延迟毛刺；压测客户端若不复用连接，还会放大连接建立开销
- **怎么配**：长连接高频调用场景调大到 10000
- **怎么验证**：`curl -v http://... 2>&1 | grep -c "Re-using"` 之类观察连接复用；或压测期看 TIME_WAIT 数量

> 3.x 变化：该属性更名为 `server.tomcat.keep-alive-max-requests`——2.7 用 `max-keep-alive-requests`。

```properties
server.tomcat.max-keep-alive-requests=10000
```

### maxSwallowSize：2MB 静默截断

请求体超 2MB 时 Tomcat 不会读完剩余 body 就掐断连接——客户端看到的是**连接重置而非 413**，报错信息完全误导排查方向。上传场景必配：

```properties
server.tomcat.max-swallow-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

- **怎么验证**：`curl -v -F file=@10mb.bin http://.../upload`，返回 413/正常处理 = 配置生效；连接被 reset = 未生效

## 与其他文档的分工

- **怎么压测找上限** → [SpringBoot默认并发上限压测](SpringBoot默认并发上限压测实践.md)
- **已经出事了怎么排查** → 故障排查与复盘（CPU 飙升、OOM、锁表、Pod 重启等事后案例）
- 本文定位：**还没出问题时的事前检查**，上线/压测前过一遍速查表
