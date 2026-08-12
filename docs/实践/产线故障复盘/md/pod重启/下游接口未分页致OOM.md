# 下游接口未分页致OOM

> **涉及服务**: 业务服务A
> **故障类型**: Pod重启

## 故障现象

### 子案例一
早上 09:03，业务服务A 同 Deployment 下的一个 pod（业务服务A-pod-xxx）发生重启，另一个 pod 正常未重启。grafana 监控显示异常 pod 的内存 used 在 09:05 前后明显波动并接近 3GB limit；同期下游数据服务B 收到自动告警"响应体大于 10MB"，时间集中在 09:05 前后。k8s 事件显示 pod 因 liveness probe 失败被重启。

当时未找到真实原因，曾推测为"月初大数据量查询 → GC 停顿 → liveness 超时"。事后结合另外两起同类事故（子案例二、子案例三）确认，真实根因相同：下游指标接口未实现真分页，导报表等大批量查询时全量数据堆积致内存压力接近 limit，服务变慢致 liveness 超时。

### 子案例二
早上 8:40，业务服务A 的一个 pod（业务服务A-pod-xxx）发生重启，另一个 pod 正常未重启。同一时段 ELK 日志出现大量调用算法服务的 ERROR，算法接口调用量异常偏高、单次响应体约 1.4MB，异常期间共堆积约 254 条响应到内存，纯文本即约 355MB，反序列化为 Map 后占用更大，最终触发 OOM。

### 子案例三
晚上 22:45，生产命名空间一个 pod 重启。grafana 内存监控显示 22:45-22:50 used 内存曲线急速上升接近 6GB（接近 max），定位为 OOM 导致。由于进行了回滚，从运维拿到的重启前日志中可见 Feign 调用下游指标接口的分页参数已递增到 `current:8148`、`size:1000`，说明下游接口未实现真分页，客户端持续累加全量结果导致内存溢出。

## 现场分析

### 子案例一
- k8s 事件：pod 因 liveness probe 失败被 k8s 重启。livenessProbe 为 `httpGet path: /actuator port: 9099`，`timeoutSeconds: 3`、`periodSeconds: 5`、`failureThreshold: 3`；readinessProbe 为 `tcpSocket port: 8081`，`timeoutSeconds: 1`、`failureThreshold: 3`。
- grafana 监控：异常 pod（业务服务A-pod-xxx）内存 used 接近 3GB limit，正常 pod 同期平稳，说明故障窗口内内存持续增长接近上限。
- 下游告警：数据服务B 收到"响应体大于 10MB"自动告警，与内存高峰、响应变慢时间窗吻合——大响应体是下游未分页的典型信号。
- Skywalking 链路：系统数据订正任务（定时任务）线程链路耗时显著拉长，伴随 HTTP、HikariCP、PostgreSQL 节点，符合月初跑批/导报表等大批量查询特征。
- 关键认知：当时推断"GC 停顿致 liveness 超时"是未找到真因时的推测；结合另外两起同类事故，"响应体大于 10MB"告警与内存接近 limit 共同指向"下游未分页、全量数据堆积"，这才是真实根因。

### 子案例二
- k8s 事件：pod 的 readiness probe 失败，HTTP probe failed with statuscode 500，content deadline exceeded（Client Timeout exceeded while streaming headers），来源节点为 内网IP。
- grafana 内存监控：业务服务A 的堆内 used 内存与 max 内存在 08:20-08:50 间多次出现冲高波动，与内存飙升时段吻合；堆外内存相对平稳。GC 次数与 GC 暂停时间图呈现对应时段的暂停时长，直观反映异常时段内存及垃圾回收相关的运行状态。
- Skywalking 链路追踪：按响应时长（Duration）排序，最长响应时间的请求为业务服务A 的 QueryDataTask 服务调用，响应时间约 440991ms（约 7.3 分钟），与内存飙升时段吻合；调用链路下游包含多个 history-data-query 子请求（响应时间约 570ms）。
- ELK 日志：异常时段 08:27:14-08:44:33 的日志与 grafana 异常时间吻合；ERROR 日志在异常时段较多，且基本都是调用算法服务的 error。算法接口（getSystemHistoryTimeDataQueryTheoryDay）在异常时段调用量特别高、耗时都比较长。
- 数据量分析：单条响应日志大小约 1.37MB（source 为内网IP 的 HTTP 响应），异常期间共约 254 条响应堆积在内存中，纯文本约 355MB，反序列化为 Map 后占用更大。
- 上游调用方请求业务服务A 的参数：`dimension=day`、`startDt=2026-04-13T00:00:00`、`endDt=2026-04-13T23:59:59.999`、`thingIdList` 包含 2000+ 个系统 ID、`tags` 包含 10 个 30 日综合效率相关指标（理论发电量、系统效率、等效利用小时数等）。

### 子案例三
- grafana 内存监控：21:50-23:00 期间，used 内存曲线呈波动上升趋势，max 相对平稳；22:45-22:50 期间 used 急速上升接近 6GB，定位为 OOM 导致的 pod 重启。
- Feign 调用日志（回滚后从运维日志获得）：`2026-07-02 22:44:57.233 INFO [Thread-44] 业务服务A 的 Feign 日志组件 [InnerServiceClient#getData] ---> POST 数据服务B 的指标数据接口`，请求体 `current=8148, size=1000, counted=false`，`identityCodes=["assetsManage"]`、`startDt=2026-04-30 00:00:00`、`endDt=2026-06-30 23:59:59`、`dimension=dayHours`、`tags=["total_energy","rolling_energy"]`。
- `current:8148` 表明分页页码已递增到 8148，说明进行了多次全量查询；与下游确认后，注册的下游接口未实现分页查询，每次"分页"实际都返回全量数据，客户端将所有返回累加到 list 中，最终内存溢出。

## 根因

下游指标接口"有分页参数"但未实现真分页：接口对外暴露 `current`/`size` 等分页参数，但内部并未真正按分页截取数据，每次请求都返回全量结果。业务服务A 在请求大批量系统ID（2000+ 个）的日级/小时级指标数据时（子案例一：月初导报表；子案例二：日级综合效率；子案例三：小时级数据），客户端按"分页"循环调用，把每次返回的全量数据都 add 到内存中的 list，数据堆积无法释放，最终触发 OOM 或内存压力致 liveness 超时。

三个子案例触发场景不同（子案例一为系统数据订正任务/导报表、子案例二为算法服务指标接口、子案例三为数据服务B 的指标数据接口），但根因相同：下游接口未实现真分页，客户端误把全量响应当分页结果累加。其中子案例一当时误判为"GC 停顿致 liveness 超时"，实际为下游未分页大响应体堆积致内存压力。

## 解决方案

- 推动下游接口实现真分页：在数据库查询或内存计算层真正按 `current`/`size` 截取数据，避免每次返回全量结果；联调用例必须覆盖"翻页到末页"场景，验证返回数据量递减至空。
- 业务服务A 侧增加防御：限制单次查询的系统ID数量与时间跨度，超限时分批请求；对下游响应做大小校验，单条响应超过阈值时熔断告警。
- 增加 pod 内存上限：短期可将 pod 内存 limit 与 JVM 堆适当上调（子案例一曾将 3GB→4GB 并放宽 liveness `timeoutSeconds: 3→5`），作为缓冲，但根因仍需下游真分页修复。
- OOM 时保留 heapdump 与日志到文件：便于事后定位堆积对象类型与调用栈。
- 下游接口注册时增加分页契约校验：注册到内部服务调用注册表时，强制要求声明是否支持分页，并在网关/客户端层面增加拦截。

## 复盘要点

- 下游接口"有分页参数"不等于"真分页"，需联调验证：翻页到末页时返回应为空或递减，而不是每页都返回全量。
- 现象链相似时优先怀疑同一根因：三起事故（子案例一、二、三）均表现为"内存接近 limit + 下游大响应体告警"，真实根因均为下游未分页，避免重复误判。
- 未找到真因前的推测必须标注"推测"并持续验证："GC 停顿致 liveness 超时"只是表面现象，需追查内存为何冲高（数据量/响应体），再调探针参数。
- liveness 超时未必是探针阈值问题，先查"为什么响应慢"（数据量、内存、下游接口），确认根因后再调整探针配置。
- 大数据量查询应做内存预算与限流：单次请求的系统ID数量、时间跨度需有上限，超限分批；响应体大小需监控，超阈值熔断。
- OOM 时保留 heapdump 与日志到文件便于定位：heapdump 可揭示堆积对象类型，文件日志在 pod 重启后仍可追溯。
- `current` 参数递增到异常大值（如 8148）是下游未真分页的强信号，监控应针对此告警。

## 关键命令与指标

```bash
# k8s describe pod（liveness/readiness probe 失败，HTTP probe failed with statuscode 500）
# grafana 内存 used 接近 limit（子案例一: 3GB；子案例二: max 6GB；子案例三: 接近 6GB）
# livenessProbe httpGet /actuator:9099 timeout 3s period 5s failure 3（子案例一曾放宽至 5s）
# 下游告警：响应体大于 10MB（子案例一）
# Skywalking 最长 Duration 约 440991ms（业务服务A 的 QueryDataTask，子案例二）
# 单条响应约 1.4MB × 254 条 ≈ 355MB（纯文本，Map 化后更大，子案例二）
# Feign 日志：current=8148, size=1000（分页页码异常递增，表明下游未真分页，子案例三）
# 上游请求 thingIdList 约 2000 个系统ID × 30 天日级数据
```

## 时间线

### 子案例一
| 时间 | 事件 |
|------|------|
| 09:03 | 业务服务A 一个 pod（业务服务A-pod-xxx）重启，另一个 pod 正常 |
| 09:05 前后 | 内存 used 明显波动接近 3GB limit；下游数据服务B 收到"响应体大于 10MB"告警 |
| 故障窗口 | Skywalking 显示系统数据订正任务链路耗时拉长，伴随 HTTP/HikariCP/PostgreSQL 节点 |
| 故障窗口 | liveness probe `/actuator:9099` 超时无响应，k8s 按 failureThreshold=3 重启 pod |
| 事后 | 当时误判为 GC 停顿致 liveness 超时；与另外两起事故对照后确认真实根因为下游接口未分页 |

### 子案例二
| 时间 | 事件 |
|------|------|
| 08:20 | grafana 堆内 used 内存开始多次冲高波动 |
| 08:27:14 | ELK 开始出现异常时段日志 |
| 08:27:54 | 算法接口（getSystemHistoryTimeDataQueryTheoryDay）开始大量调用且耗时较长 |
| 08:36:00 | ELK 出现多条 ERROR 日志，均为调用算法服务的 error |
| 08:40 | 业务服务A 的 pod（业务服务A-pod-xxx）发生重启 |
| 08:44:33 | ELK 最后一条异常日志，单条响应大小 1.37MB |
| 08:50 | grafana 内存冲高波动结束 |
| 事后 | 测试确认算法接口虽有分页参数但未实现分页，数据量大于 1000 时无限查询存入 list，引发 OOM |

### 子案例三
| 时间 | 事件 |
|------|------|
| 21:50 | grafana 内存监控开始，used 内存呈波动上升趋势 |
| 22:42 | Feign 日志开始记录下游指标接口调用 |
| 22:44:57.233 | Feign 日志记录 `current=8148, size=1000` 的全量查询请求 |
| 22:45 | 生产命名空间 pod 因 OOM 重启，used 接近 6GB |
| 22:50 | grafana 内存 used 达到峰值后 pod 已重启 |
| 事后 | 与下游确认数据服务B 的指标接口未实现分页查询，注册的第三方接口未分页导致持续全量查询，内存溢出 |
