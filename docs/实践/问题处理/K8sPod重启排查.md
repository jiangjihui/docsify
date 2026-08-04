# Kubernetes Pod 重启排查

在 Kubernetes 中，排查 Pod 重启原因通常遵循“由表及里”的顺序。以下是几种最常用且有效的方法，按推荐程度排序：

### 1. 查看 Pod 事件（Events）
`kubectl describe pod <pod-name>` 是最核心的命令。重点查看底部的 **Events** 部分和顶部的 **Containers** 状态。

*   **Last State**: 在 `Containers` 区域下，如果 Pod 发生过重启，会显示 `Last State`。其中的 `Reason` 字段是关键：
    *   `OOMKilled`: 内存溢出被杀（最常见）。
    > 多见于内存泄漏或 `memory.limit` 设得过小；真实案例可参考 [容器与 JVM 内存配置不一致致 OOMKiller](/实践/产线故障复盘/md/pod重启/容器与JVM内存配置不一致致OOMKiller.md) 与 [下游接口未分页致 OOM](/实践/产线故障复盘/md/pod重启/下游接口未分页致OOM.md)。
    *   `Error`: 进程异常退出。
    *   `Completed`: 正常执行完毕退出（对于长期运行的服务来说属于异常）。
*   **Events**: 查看最近的事件记录，常见的重启相关事件包括：
    *   `BackOff` / `CrashLoopBackOff`: 容器反复崩溃重启。
    *   `Unhealthy`: Liveness Probe 或 Readiness Probe 检查失败导致 Kubelet 杀掉容器。
    *   `Evicted`: 节点资源不足导致 Pod 被驱逐。
    *   `Preempting`: 被高优先级 Pod 抢占。

```bash
kubectl describe pod <pod-name> -n <namespace>
```

动手前先快速确认重启次数、落点节点，以及「从何时开始重启」：

```bash
# 一眼看 RestartCount（重启次数）与 NODE（落在哪个节点）
kubectl get pod <pod-name> -n <namespace> -o wide
# 只看重启次数
kubectl get pod <pod-name> -n <namespace> -o jsonpath='{.status.containerStatuses[*].restartCount}'
```

`describe` 输出的 `Containers.<name>.Last State.Terminated` 还带 `StartedAt` / `FinishedAt` 时间戳，可据此定位重启开始的时间点。

### 2. 查看上一次运行的日志
如果容器已经重启，当前 `kubectl logs` 只能看到新容器的日志。**必须加 `--previous` (或 `-p`) 参数**才能看到崩溃前的日志：

```bash
# 查看上一次崩溃前的日志
kubectl logs <pod-name> -n <namespace> --previous

# 如果Pod有多个容器，指定容器名
kubectl logs <pod-name> -c <container-name> --previous
```

> ⚠️ 注意：如果 Pod 重启次数过多，`--previous` 可能只保留倒数第二次运行的日志。

### 3. 检查退出码（Exit Code）
在 `describe pod` 的 `Last State` 中会显示 `Exit Code`，这是判断根因的重要线索：

| Exit Code | 含义 | 常见原因 |
| :--- | :--- | :--- |
| **0** | 正常退出 | 主进程执行完毕，但 Pod 期望长期运行 |
| **1** | 通用错误 | 应用代码异常、配置错误、依赖缺失 |
| **137** | SIGKILL (9) | OOMKilled、手动 kill、节点资源回收 |
| **139** | SIGSEGV (11) | 段错误，通常是代码 bug 或二进制损坏 |
| **143** | SIGTERM (15) | 优雅终止信号，可能是探针失败或调度触发 |
| **255** | 超出范围 | 通常是启动脚本错误或 shell 异常 |

### 4. 检查探针（Probe）配置
如果 Events 中出现 `Unhealthy`，说明是探针导致的重启：
*   **Liveness Probe 失败** → Kubelet 直接杀掉并重启容器
*   **Startup Probe 失败** → 容器启动超时被杀

检查方法：在 `describe pod` 输出中查看 `Liveness`、`Readiness`、`Startup` 的配置是否合理（初始延迟、超时时间、阈值等）。

### 5. 检查节点级别的问题
如果多个不同应用的 Pod 在同一节点上同时重启，问题可能在节点侧：

```bash
# 查看节点事件
kubectl describe node <node-name>

# 查看 kubelet 日志（需要登录到节点）
journalctl -u kubelet --since "1 hour ago" | grep -i "kill\|evict\|oom"

# 查看系统级 OOM Killer 记录
dmesg | grep -i "oom\|killed"
```

### 6. 使用高级工具辅助排查

| 工具 | 用途 |
| :--- | :--- |
| **stern / kubectl-logs** | 实时追踪多容器/Pod 日志流 |
| **kubespy** | 实时观察 K8s 资源变化 |
| **Prometheus + Grafana** | 查看重启前的 CPU/内存曲线，确认是否有资源突增 |
| **kubectl-debug / ephemeral containers** | 向运行中的 Pod 注入调试容器进行现场排查 |

### 🔍 快速排查流程图

```
Pod 重启了
    │
    ├── kubectl describe pod → 看 Last State.Reason + Events
    │       ├── OOMKilled → 调大 memory limit / 排查内存泄漏
    │       ├── Unhealthy → 检查探针配置和应用健康端点
    │       ├── Evicted → 检查节点资源和 Pod QoS 等级
    │       └── Error/其他 → 继续 ↓
    │
    ├── kubectl logs --previous → 看崩溃前应用日志
    │       ├── 有明确报错 → 修复应用代码/配置
    │       └── 无有用信息 → 继续 ↓
    │
    ├── 检查 Exit Code → 对照上表定位方向
    │
    └── 检查节点日志(dmesg/kubelet) → 排查系统级问题
```


**💡 排查小贴士：**
*   **如果是 OOMKilled (137)**：检查你的应用是否存在内存泄漏，或者适当调大 Deployment 中的 `resources.limits.memory` 配置。
*   **如果是应用报错 (1)**：在 `--previous` 的日志中搜索 `Error`、`Exception`、`Connection refused` 等关键字，通常能直接看到是连不上数据库、缺配置文件还是代码逻辑报错。
*   **如果是健康检查失败**：有时候应用启动较慢，但 `livenessProbe`（存活探针）配置的时间太短，K8s 会误以为应用卡死而强制重启它。可以检查 `describe` 输出中是否有 `Liveness probe failed` 的警告。

**最佳实践建议**：在生产环境中，务必配置好 **Prometheus 监控 Pod 重启次数** (`kube_pod_container_status_restarts_total`) 和 **资源使用率告警**，这样可以在用户感知之前主动发现问题。


> **延伸阅读**：
> - Kubernetes 官方文档 —— [容器探针（Probes）](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
> - Kubernetes 官方文档 —— [Pod 生命周期与终止](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
> - kube-state-metrics 指标说明 —— [kube_pod_container_status_restarts_total](https://github.com/kubernetes/kube-state-metrics/blob/main/docs/pod-metrics.md)