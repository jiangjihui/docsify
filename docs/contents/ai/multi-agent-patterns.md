# 多智能体协作模式

## 概览

当任务复杂到单 Agent 难以胜任（需要多种专长、长流程、质量把关），就把工作**拆分给多个各司其职的 Agent**，用不同拓扑组织它们协作。本文系统梳理主流多 Agent 协作模式。

```
        Supervisor（监督者）
       /      |       \
  WorkerA   WorkerB   WorkerC      ← 各管一摊
       \      |       /
        Orchestrator（汇总）
```

> 前置阅读：[AI Agent（智能体）](/contents/ai/ai-agent.md)、[Function Calling / Tool Use](/contents/ai/function-calling.md)。实现见 [LangGraph](/contents/ai/langgraph.md)、[DeerFlow](/contents/ai/deerflow.md)；安全见 [AI 安全与防护](/contents/ai/ai-safety.md)。

---

### 为什么要多 Agent？

- **专长分离**：不同 Agent 用不同提示/工具/模型，各管一摊（研究、写代码、审稿）。
- **模块化**：单个 Agent 更可控、更易测，改动互不影响。
- **可扩展**：加新能力 = 加新 Agent，而非把逻辑塞进一个超长提示。
- **质量提升**：用"生成 + 评审"分工提升产出质量。

> 经验法则（Anthropic）：**多数情况单 Agent + 好工具就够了**；只有真正需要并行专长或强隔离时才上多 Agent，否则复杂度反噬。

### 模式一：Supervisor（监督者 / 路由）

一个中心 **Supervisor** Agent 负责理解任务、**把子任务路由**给专门的 Worker，再汇总。Worker 之间不直接通信。

```
User → Supervisor → 路由 → WorkerA / WorkerB / WorkerC → 汇总 → User
```

- **优点**：结构清晰、易控、易观测。
- **缺点**：Supervisor 是瓶颈，路由错误会传导。
- **适用**：任务可清晰分派给不同专长的场景。
- **实现**：LangGraph 里 Supervisor 是一个节点，`add_conditional_edges` 按 LLM 路由结果连到对应 Worker（见 [LangGraph](/contents/ai/langgraph.md)）。

### 模式二：Orchestrator-Workers（编排者-工人）

**Orchestrator** 不只是路由，还会**动态拆解任务、把子任务派给 Worker、收集结果并综合**。Worker 可并行执行不同子任务，最后由 Orchestrator 汇总成最终产出。

- **与 Supervisor 的区别**：Orchestrator 主动"规划+聚合"，而非仅做分类路由；Worker 常并行。
- **适用**：需要把大任务拆成多个可并行部分的复杂生成/分析（如一次生成多份材料再整合）。
- **示例**：DeerFlow 的 Planner 即编排者，把研究计划派给 Researcher/Coder，再交 Reporter 汇总（见 [DeerFlow](/contents/ai/deerflow.md)）。

### 模式三：分层（Hierarchical）

在 Supervisor/Orchestrator 之上再叠一层管理者，形成多级团队（组→部门→总控）。适合超大规模、组织化极强的系统；代价是链路长、调试难。

### 模式四：Swarm / Network（去中心化协作）

Agent 之间**直接互转（handoff）**：A 觉得自己搞不定某子任务，就把上下文交给更合适的 B，B 处理完再交回或转给 C。没有固定中枢。

```
AgentA ⇄ AgentB ⇄ AgentC   （按需交接，无固定层级）
```

- **优点**：灵活、自然、贴近人类团队协作。
- **缺点**：流程难预测、易死循环、难观测。
- **适用**：开放式、难以预先规划步骤的任务。
- **实现**：OpenAI Swarm / AutoGen 的 handoff 机制；LangGraph 可用节点间条件边模拟。

### 模式五：辩论 / 评审（Debate / Review）

让两个（或多个）Agent **对同一问题给出方案并互相批评**，由裁判（人或模型）选定/综合最优。

```
Generator 提案 → Critic 批评 → Generator 修订 → … → Judge 定稿
```

- **优点**：显著提升正确性、减少幻觉（尤其数学、代码、事实）。
- **缺点**：多轮调用，成本高、慢。
- **适用**：对质量极敏感的场景（代码评审、方案论证、安全审查）。

### 模式对比

| 模式 | 结构 | 优点 | 缺点 | 适用 |
|------|------|------|------|------|
| Supervisor | 中心路由 | 清晰、易控 | 中枢瓶颈 | 可分派专长任务 |
| Orchestrator-Workers | 规划+并行+汇总 | 并行、质量高 | 编排复杂 | 大而可拆的任务 |
| Hierarchical | 多级 | 组织化强 | 链路长、难调 | 超大规模 |
| Swarm/Network | 去中心交接 | 灵活自然 | 难预测、易循环 | 开放式任务 |
| Debate/Review | 对抗评审 | 质量高 | 慢、贵 | 质量敏感场景 |

### 实现要点

- **状态传递**：用结构化状态在 Agent 间传递上下文（[LangGraph](/contents/ai/langgraph.md) 的 State）。
- **终止条件**：设定最大轮数/完成信号，防止无限循环。
- **成本控制**：多 Agent = 多次模型调用，需限轮次、选模型档位（[LLM 基础](/contents/ai/llm-basics.md)）。
- **可观测**：记录每次交接与决策（LangSmith，见 [LangChain](/contents/ai/langchain.md)）。
- **安全**：Agent 有工具即有能力，需护栏与确认（[AI 安全与防护](/contents/ai/ai-safety.md)）。

### 与本分类的关系

- **DeerFlow** 是 Orchestrator-Workers 的现成实现（Planner + research_team + Reporter，见 [DeerFlow](/contents/ai/deerflow.md)）。
- **LangGraph** 把这些模式都建模成"图 + 条件边"，便于声明与调试（见 [LangGraph](/contents/ai/langgraph.md)）。

### 相关资源

- [AI Agent（智能体）](/contents/ai/ai-agent.md) — 单 Agent 基础
- [Function Calling / Tool Use](/contents/ai/function-calling.md) — Agent 的行动机制
- [LangGraph](/contents/ai/langgraph.md) — 多 Agent 的图编排
- [DeerFlow](/contents/ai/deerflow.md) — Orchestrator-Workers 实例
- [AI 安全与防护](/contents/ai/ai-safety.md) — 多 Agent 的风险
- [Building Effective Agents (Anthropic)](https://www.anthropic.com/research/building-effective-agents)
