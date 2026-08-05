# LangGraph

## LangGraph 概览

LangGraph 是一个用于构建**有状态、多角色（multi-actor）** LLM 应用的编排框架，由 LangChain 团队推出。它把一次 Agent 的执行过程建模成一张**图（Graph）**：节点（Node）执行具体逻辑，边（Edge）定义流转方向，状态（State）在节点之间流转。

与大多数"链（Chain）"式框架不同，LangGraph 的图**允许包含环（Cycle）**，因此天然适合表达 Agent 的"思考—行动—观察"循环、以及多 Agent 之间的反复协作。

```
┌─────────────────────────────────────────────────┐
│                  LangGraph                       │
│                                                 │
│   ┌────────┐    ┌────────┐    ┌────────┐       │
│   │ Node A │───▶│ Node B │───▶│ Node C │       │
│   └────────┘    └───▲────┘    └───┬────┘       │
│        ▲           │             │            │
│        │           └─────────────┘            │
│        │           条件边（环形回流）           │
│        └─────────────────────────────────────┘
│              State 在节点间流转                    │
└─────────────────────────────────────────────────┘
```

> 本文配套阅读：[AI Agent（智能体）](/contents/ai/ai-agent.md)，其中对 Agent 的基础概念、工具、MCP、记忆等做了系统梳理；LangGraph 正是把这些能力"编排成工作流"的引擎。

---

### 什么是 LangGraph？

一句话：**LangGraph = 用图来编排有状态的 LLM 工作流**。

- 它解决的核心问题是：当任务不再是"线性调用一次 LLM"，而是需要**循环、分支、人工介入、断点续跑、多 Agent 协作**时，纯 Chain 难以表达。
- 它不是一个全新的 LLM 框架，而是**建立在 LangChain 之上**的编排层：你依然用 LangChain 的 LLM、Tool、Prompt、Runnable，只是把它们的执行顺序交给"图"来定义。
- 它强调**显式可控**：流程由你用代码声明，而不是黑盒自动执行，便于调试、干预和复现。

### 核心概念

LangGraph 的世界有且仅有四个核心抽象：**State（状态）、Node（节点）、Edge（边）、Graph（图）**。

| 概念 | 角色 | 说明 |
|------|------|------|
| **State** | 贯穿全流程的共享数据 | 一个 TypedDict / Pydantic 模型，节点读写它来传递信息 |
| **Node** | 处理逻辑单元 | 一个函数或 Runnable，接收 State、返回对 State 的"增量更新" |
| **Edge** | 流转路径 | 普通边（固定走向）和条件边（按 State 动态路由） |
| **Graph** | 整体编排容器 | 由 StateGraph 构建、编译后变成可运行的图 |

此外有两个**特殊节点** `START`（入口）和 `END`（出口），用于标记图的开始与结束。

### 架构特点

LangGraph 区别于普通链式框架的关键，在于它的几个架构设计取向：

| 特点 | 说明 | 带来的收益 |
|------|------|-----------|
| **基于图且支持环** | 图不是纯 DAG，允许节点回流 | 原生支持 ReAct 循环、反思、重试 |
| **有状态（Stateful）** | 内置 Checkpointer 持久化每一步状态 | 可记忆、可断点续跑、可做"时间旅行" |
| **显式可控** | 流程由代码声明，逐节点定义 | 易调试、可人工干预、行为可复现 |
| **流式输出** | 支持 token 级 / 节点级流式 | 长任务下用户体验更好 |
| **容错与续跑** | 崩溃后从最近检查点恢复 | 适合长时间运行的任务 |

对比视角：如果说 LangChain 的 LCEL 擅长把"输入→模型→输出"串成**线性管道**，那么 LangGraph 擅长描述**带循环、带分支、带记忆**的**有状态工作流**。

### 主要组件与构建方式

#### State（状态）

State 是整张图共享的数据结构，**定义了节点之间传递什么信息**。最常见的写法是 `TypedDict`，也可以用 Pydantic 模型。

关键点在于**更新函数（reducer）**：默认情况下，节点返回的同名字段会**整体覆盖**旧值；但你可以使用 `Annotated[类型, reducer]` 声明"如何合并"，而不是覆盖。最典型的例子是消息列表——希望新消息**追加**而非替换：

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages
from langchain_core.messages import HumanMessage

class State(TypedDict):
    # messages 字段会用 add_messages 把新消息"追加"到旧列表
    messages: Annotated[list, add_messages]
    # 普通字段：节点返回同名值时会整体覆盖
    step: int
```

| 写法 | 默认行为 | 适用场景 |
|------|---------|---------|
| `field: type` | 覆盖 | 单值结果（如最终答案） |
| `Annotated[list, add_messages]` | 追加消息 | 多轮对话历史 |
| `Annotated[list, operator.add]` | 拼接列表 | 累积收集多条结果 |

#### Node（节点）

节点就是**一个接收 State、返回 State 增量更新的函数**（或任意 LangChain Runnable）。它不能就地修改 State，只能返回需要更新的字段：

```python
def agent_node(state: State):
    # 读取当前状态
    last_msg = state["messages"][-1]
    # ...调用 LLM、工具等...
    reply = "这是模型的回复"
    # 返回"增量"：只给需要更新的字段
    return {"messages": [("ai", reply)], "step": state.get("step", 0) + 1}
```

注意：返回的是**部分状态（partial state）**，框架会按需合并（按 reducer 或覆盖规则）。

#### Edge（边）

边定义了"执行完一个节点后去哪"。有三种常见形态：

| 边类型 | 写法 | 说明 |
|--------|------|------|
| 普通边 | `add_edge(A, B)` | 固定从 A 到 B |
| 入口/出口 | `add_edge(START, A)` / `add_edge(B, END)` | 标记起点与终点 |
| 条件边 | `add_conditional_edges(A, router, {"分支名": 目标})` | 由 `router(state)` 的返回值决定走向 |

条件边是 Agent 行为分支的核心：

```python
from typing import Literal

def router(state: State) -> Literal["tool", "finish"]:
    last = state["messages"][-1]
    # 如果模型要求调用工具，则走向 tool 节点；否则结束
    if last.additional_kwargs.get("tool_calls"):
        return "tool"
    return "finish"

builder.add_conditional_edges("agent", router, {"tool": "call_tool", "finish": END})
```

#### Graph（图）的构建方式

构建一个可运行的图遵循固定套路：**定义 State → 加节点 → 加边 → 编译**。

```
定义 State（数据结构）
      │
      ▼
StateGraph(State)            # 创建构建器
      │
      ├─ add_node("名", 函数)  # 添加节点（可多个）
      │
      ├─ add_edge(START, "名") # 连接入口
      ├─ add_edge("名", "名")  # 普通边
      └─ add_conditional_edges(...)  # 条件边
      │
      ▼
builder.compile()            # 编译为可运行图
      │
      ▼
graph.invoke(input)          # 执行
```

```python
from langgraph.graph import StateGraph, START, END

builder = StateGraph(State)
builder.add_node("agent", agent_node)
builder.add_node("call_tool", tool_node)

builder.add_edge(START, "agent")
builder.add_conditional_edges(
    "agent",
    router,
    {"tool": "call_tool", "finish": END},
)
builder.add_edge("call_tool", "agent")  # 工具调用完后回到 agent（环）

graph = builder.compile()      # 编译
result = graph.invoke({"messages": [("user", "北京天气？")]})
```

`compile()` 之后的 `graph` 是一个标准的 LangChain Runnable，支持 `invoke` / `stream` / `ainvoke` 等统一接口。

### 与 LangChain 的关系

两者不是替代关系，而是**互补的分层关系**。

| 维度 | LangChain | LangGraph |
|------|-----------|-----------|
| 定位 | 组合 LLM 调用的"积木与管道" | 编排有状态、循环、多角色的"工作流引擎" |
| 抽象 | Chain / Runnable / LCEL | Graph / Node / Edge / State |
| 流程形态 | 偏向线性（DAG） | 支持环、分支、循环 |
| 状态管理 | 无内建持久化 | 内建 Checkpointer（记忆/续跑） |
| 可控性 | 高层封装，黑盒执行 | 显式声明每一步，便于干预 |
| 适用 | 一次性问答、RAG、简单链 | Agent 循环、多 Agent、人工介入 |

一句话总结：**LangChain 负责"怎么调用模型与工具"，LangGraph 负责"按什么顺序、在什么状态下、循环多少次地去调用它们"。** LangGraph 完全复用 LangChain 的 LLM、Tool、Prompt、Memory 等原语，并在其上叠加图编排能力。

LangGraph 也提供了**预置 Agent**：`langgraph.prebuilt.create_react_agent` 可以零样板代码地搭建一个 ReAct 式工具调用 Agent（见下文示例）。

### 典型使用场景

| 场景 | 为什么用 LangGraph |
|------|-------------------|
| **多步 Agent 循环（ReAct）** | 思考→行动→观察的循环天然是"环"，图能直接表达 |
| **多 Agent 协作** | 监督者（supervisor）、分层、群（swarm）等拓扑用节点/边清晰建模 |
| **人工介入（Human-in-the-loop）** | 用 `interrupt` 在关键节点暂停，等人工确认后继续 |
| **长时任务 / 断点续跑** | Checkpointer 保存每一步，崩溃或重启后可恢复 |
| **带分支与反思的流程** | 条件边实现"判断→重试 / 通过"的质量门禁 |
| **RAG 精炼循环** | 检索→生成→评估→不足的再检索，形成闭环 |

### 简单示例

下面给出两个由浅入深的例子。

#### 示例一：自定义带"环"的图（生成 → 评判 → 不通过则改写）

这个例子刻意展示 LangGraph 的招牌能力——**条件边形成的环**：笑话生成后由评判节点决定"通过"或"重写"，重写后再次评判，直到通过为止。

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    topic: str
    joke: str
    verdict: str   # "funny" / "not_funny"

def generate(state: State):
    return {"joke": f"关于《{state['topic']}》的笑话：……"}

def judge(state: State):
    # 真实场景此处调用 LLM 评判；这里简化为始终需要先重写一次
    return {"verdict": "not_funny" if "改版" not in state["joke"] else "funny"}

def rewrite(state: State):
    return {"joke": state["joke"] + "（改版加强版）"}

def route(state: State) -> Literal["approved", "retry"]:
    return "approved" if state["verdict"] == "funny" else "retry"

builder = StateGraph(State)
builder.add_node("generate", generate)
builder.add_node("judge", judge)
builder.add_node("rewrite", rewrite)

builder.add_edge(START, "generate")
builder.add_edge("generate", "judge")
builder.add_conditional_edges("judge", route, {"approved": END, "retry": "rewrite"})
builder.add_edge("rewrite", "judge")   # 改写后回到评判（环）

graph = builder.compile()
result = graph.invoke({"topic": "程序员"})
print(result["joke"], "->", result["verdict"])
```

#### 示例二：用预置 Agent 快速搭建带工具的 ReAct Agent

实际项目里多数工具调用型 Agent 不需要手写图，直接用 `create_react_agent`：

```python
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """查询指定城市的天气。"""
    return f"{city} 今天晴，25°C"

model = ChatOpenAI(model="gpt-4o-mini")
agent = create_react_agent(model, tools=[get_weather])

result = agent.invoke({"messages": [("user", "北京天气怎么样？")]})
print(result["messages"][-1].content)
```

### 进阶要点：持久化与人工干预

LangGraph 真正区别于"普通循环代码"的，是内建的**持久化与干预**能力。

**1. Checkpointer（检查点）实现记忆与续跑**

通过给编译后的图传入 Checkpointer，每一步状态都会被按 `thread_id` 保存，从而支持跨会话记忆与崩溃恢复：

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()
agent = create_react_agent(model, tools=[get_weather], checkpointer=checkpointer)

config = {"configurable": {"thread_id": "conv-001"}}
agent.invoke({"messages": [("user", "北京天气？")]}, config)
# 同一 thread_id 再次调用，模型能"记得"上一轮对话
```

**2. interrupt 实现人工介入（Human-in-the-loop）**

在任意节点中用 `interrupt` 暂停，等待人工输入后通过 `Command(resume=...)` 继续，常用于敏感操作前的审批：

```python
from langgraph.types import interrupt, Command

def human_approval(state):
    decision = interrupt({"question": "确认要执行删除操作吗？"})
    return {"approved": decision == "yes"}

# 恢复执行
agent.invoke(Command(resume="yes"), config)
```

**3. 时间旅行（Time Travel）**

因为每一步都被持久化，你可以 replay 任意历史检查点、或从某个中间状态分支探索不同走向——这对调试复杂 Agent 非常有用。

### 部署与可视化

- **LangGraph Studio**：官方提供的桌面/Web IDE，可可视化调试图、逐步执行、查看 State 变化。
- **LangGraph Platform**：托管服务，提供持久化、任务队列、流式 API 与监控，适合生产环境部署。
- 自托管：编译后的图就是普通 Python 对象，也能直接放进 FastAPI / 任意服务中运行。

### 相关资源

- [LangGraph 官方文档](https://langchain-ai.github.io/langgraph/)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)
- [AI Agent（智能体）](/contents/ai/ai-agent.md) — 本文的 Agent 基础概念前置阅读
- [LLM 基础](/contents/ai/llm-basics.md) / [提示词工程](/contents/ai/prompt-engineering.md) / [RAG](/contents/ai/rag.md) — 使用 LangGraph 前建议先掌握的基础
