# 容器与JVM内存配置不一致致OOMKiller

> **涉及服务**: 数据服务B
> **故障类型**: Pod重启

## 故障现象

### 子案例一
早上 8:00:36 数据服务B 的 pod 发生重启。重启后约 5 秒（8:00:41）应用日志开始报数据库连接池已满，抛出 PostgreSQL PSQLException：`remaining connection slots are reserved for non-replication superuser connections`，即数据库可用连接槽已被耗尽，新请求无法获取连接。异常持续约半小时后，8:29:00 之后未再出现新的连接异常，系统自行恢复。

### 子案例二
早上 8:00:05 数据服务B 的 pod 发生重启。应用的 ELK 日志中没有任何 error 级别记录，运维结合容器监控判定为 OOM。日志线索显示，服务端最后处理的是一个较大的查询请求，其 `node_ids` 参数多达 3250 个，响应体大小约 7.1MB（7466972 bytes），处理完该请求后服务再无新日志，进程被系统杀死。

## 现场分析

### 子案例一
- k8s 事件与监控表明 pod 在 8:00:36 因 OOMKiller 被重启（`describe pod` 的 Reason 为 OOMKilled）。
- 重启后约 8:00:41 起，应用抛出 PostgreSQL 连接池满异常：`org.postgresql.util.PSQLException: ERROR: pooler: Connect to FE failed, remaining connection slots are reserved for non-replication superuser connections`，说明数据库侧连接数已达到上限。
- 数据库的连接数上限为 512，重启后业务恢复期并发请求集中涌入，获取数据库连接的代码未加锁，多个线程同时申请连接，瞬时连接数超过 512，新请求无法再获取连接，形成连接池雪崩。
- 8:29:00 后未再出现新的连接异常，连接逐渐释放、池恢复。

### 子案例二
- ELK 日志中无 error，但 k8s 事件与容器监控显示 pod 因 OOMKiller 被重启。
- 容器内存 limit 为 4G，JVM 最大堆（-Xmx）同样配置为 4G，两者等高，未给 JVM 堆外的系统/元空间/线程栈等开销预留空间。
- 服务端最后处理的请求体量大：`node_ids` 参数 3250 个，日志推测最后停留在数据服务B 的序列化调用上（将结果对象通过 Jackson 序列化为 JSON 字符串），响应体大小约 7.1MB（7466972 bytes）。序列化大对象需要申请较多内存，JVM 在堆内存接近上限时无法从容器获取足够内存。
- 当容器内存达到 cgroup 限制，Linux 内核 OOMKiller 机制被触发，强制杀掉进程（类似 `kill -9`），`describe pod` Reason 显示 OOMKilled。

## 根因
主根因是 pod 内存 limit 与 JVM 堆配置不一致（pod 等于或不足 JVM 堆 + 系统开销），容器内存先于 JVM 堆达到上限，cgroup 触发 OOMKiller 杀死进程。两个子案例触发与次生效应不同：子案例二由大请求（3250 个 node_ids、响应体约 7.1MB）直接触发内存申请失败；子案例一在 pod 重启后引发连接池雪崩——获取数据库连接的代码未加锁，并发请求瞬时获取过多连接，超过数据库 512 的连接限制。

## 解决方案
- 调整 k8s 容器配置：pod 内存 limit 必须大于 JVM 最大堆，预留系统/堆外开销（示例：JVM 6G、pod 7G）。
- 优化业务代码，减少不必要的序列化与反序列化，大请求（数千 ID）需分批或流式处理。
- JVM 的 OOM 日志落盘到文件，并把日志路径同步给运维，便于事后排查。
- 指标接口在出现异常时应显式抛出/返回错误给调用方，不应吞掉异常。
- 获取数据库连接的代码加锁，或在项目启动阶段就完成连接初始化，避免重启后并发请求引发连接池雪崩。

## 复盘要点
- pod limit 必须大于 JVM 堆，预留系统/堆外开销，避免容器先于 JVM 触发 OOMKiller。
- 大请求（数千 ID、MB 级响应体）需分批或流式处理，避免单次序列化耗尽堆内存。
- 异常应显式抛出而非吞掉，否则日志无 error，定位困难。
- 获取连接代码需加锁或预初始化，避免重启后连接池雪崩。
- OOM 日志应落盘，便于事后追溯。

## 关键命令与指标
```bash
# k8s describe pod（Reason=OOMKiller / OOMKilled）
# ELK 日志（最后一条为序列化大对象，无 error）
# PostgreSQL PSQLException: remaining connection slots are reserved for non-replication superuser connections
# 关键指标：
#   node_ids 参数个数：3250
#   响应体大小：Response size 7466972 bytes（约 7.1MB）
#   容器内存 limit：4G
#   JVM 最大堆（-Xmx）：4G
#   数据库连接限制：512
#   修复后配置示例：JVM 6G、pod 7G
```

## 时间线

### 子案例一
| 时间 | 事件 |
|------|------|
| 08:00:36 | 数据服务B 的 pod 因 OOMKiller 重启 |
| 08:00:41 | 应用报 PostgreSQL 连接池满（PSQLException: remaining connection slots are reserved for non-replication superuser connections） |
| 08:29:00 | 之后未再出现新的连接异常，系统自行恢复 |

### 子案例二
| 时间 | 事件 |
|------|------|
| 08:00:05 | 数据服务B 的 pod 因 OOMKiller 重启 |
| 重启前 | 服务端处理 3250 个 node_ids 大请求，响应体约 7.1MB（7466972 bytes），序列化期间内存申请失败 |
| 事后 | ELK 无 error，运维结合 k8s 事件判定 OOMKilled |
