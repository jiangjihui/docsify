# GC停顿致liveness超时

> **故障日期**: 2026-04-01
> **涉及服务**: 业务服务A
> **故障类型**: Pod重启

## 故障现象
2026-04-01 早上 09:03，业务服务A 同 Deployment 下的一个 pod（业务服务A-pod-xxx）发生重启，另一个 pod（业务服务A-pod-yyy）正常未重启。grafana 监控显示异常 pod 的内存 used 在 09:05 前后出现明显波动，并接近 3GB limit（max 红线）；同期正常 pod 的内存曲线平稳。下游数据服务B 收到自动生成的异常告警，告警名称为"响应体大于 10MB"，时间集中在 09:05 前后。

## 现场分析
- **k8s 事件**：通过 KubePi 查看 pod 的 k8s 事件，pod 因 liveness probe 失败被 k8s 重启。
- **grafana 监控**：异常 pod（业务服务A-pod-xxx）的内存 used（绿色曲线）在 09:05 前后出现明显波动，接近 3GB 的 max（红色曲线）上限；正常 pod 同期内存平稳。说明异常 pod 在故障窗口内内存持续增长接近 limit。
- **下游告警**：下游数据服务B 收到自动告警"响应体大于 10MB"，时间集中在 09:05 前后，与异常 pod 内存高峰、响应变慢的时间窗吻合，说明大响应体在故障窗口内集中产生。
- **Skywalking 链路**：Trace 中出现业务服务A 的系统数据订正任务（某定时任务）相关线程，伴随 HTTP、HikariCP、PostgreSQL 等节点，链路耗时显著拉长，与月初跑批/导报表等大数据量查询特征一致。
- **k8s 测活配置**：livenessProbe 为 `httpGet path: /actuator port: 9099`，`timeoutSeconds: 3`、`periodSeconds: 5`、`failureThreshold: 3`；readinessProbe 为 `tcpSocket port: 8081`，`timeoutSeconds: 1`、`failureThreshold: 3`；`terminationGracePeriodSeconds: 60`。

推断链路：月初（4 月 1 日）导报表等大数据量查询 → 内存持续增长接近 3GB limit → 内存本就在高位的 pod 触发 JVM GC 频繁/停顿 → 服务响应变慢/卡死 → liveness probe（`/actuator` 端口 9099）超时无响应 → k8s 按 failureThreshold 重启 pod。

## 根因
月初（4 月 1 日）系统数据订正任务等大数据量查询导致内存持续增长，接近 3GB limit；内存本就在高位的 pod 触发 JVM GC 频繁/停顿，服务响应变慢/卡死；liveness probe 走 `/actuator:9099`，`timeoutSeconds=3s` 偏短，无法覆盖 GC 停顿窗口，探针连续超时达到 `failureThreshold=3` 后 k8s 重启 pod。同 Deployment 另一个 pod 因内存未冲高、未触发持续 GC 停顿，故正常存活。

## 解决方案
- pod 内存 3GB → 4GB（参考下游数据服务B 的 4GB 配置）；同步调整业务服务A 的 Dockerfile `-Xms` 等相关配置与 Deployment.yaml 内存配置。
- liveness `timeoutSeconds: 3` → `timeoutSeconds: 5`，使超时阈值与 GC 停顿预期匹配，避免短停顿即触发探针失败。
- 月初/周期性大数据查询提前扩容或限流，避免单 pod 内存长时冲高、挤压探针响应窗口。

## 复盘要点
- 月初/周期性大数据查询需提前扩容或限流，避免内存长时高位运行。
- liveness 超时阈值需与 GC 停顿预期匹配，`/actuator` 走 HTTP 时 3s 偏短，应预留 GC 停顿余量。
- pod 内存 limit 不宜卡在常规峰值，应留出 GC 与突发查询的缓冲。
- 同 Deployment 多 pod 不一定同时故障，定位时需对照正常 pod 的监控曲线快速锁定异常实例。

## 关键命令与指标
```bash
# livenessProbe httpGet /actuator:9099 timeout 3s period 5s failure 3
# readinessProbe tcpSocket:8081 timeout 1s failure 3
# terminationGracePeriodSeconds 60
# 关键指标：
#   异常 pod 内存 used：接近 3GB limit
#   pod 内存 limit：3GB（修复后 4GB）
#   liveness timeoutSeconds：3s（修复后 5s）
#   下游告警：响应体大于 10MB
#   k8s 重启原因：liveness probe 失败达 failureThreshold
```

## 时间线
| 时间 | 事件 |
|------|------|
| 09:03 | 业务服务A 一个 pod（业务服务A-pod-xxx）发生重启，另一个 pod 正常 |
| 09:05 前后 | 异常 pod 内存 used 明显波动，接近 3GB limit；下游数据服务B 收到"响应体大于 10MB"告警 |
| 故障窗口 | Skywalking Trace 显示业务服务A 系统数据订正任务等链路耗时拉长，伴随 HTTP/HikariCP/PostgreSQL 节点 |
| 故障窗口 | liveness probe `/actuator:9099` 超时无响应，k8s 按 failureThreshold=3 重启 pod |
