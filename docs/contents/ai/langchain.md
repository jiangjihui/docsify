# LangChain 框架总览

## 概览

LangChain 是当前最流行的 LLM 应用开发框架，它把"模型调用、提示、检索、记忆、工具、Agent"等能力封装成可组合的抽象，让你用更少的胶水代码搭出 RAG、对话、Agent 等应用。

它与本分类其他文章的关系：LangChain 是**积木与管道**，LangGraph（见 [LangGraph](/contents/ai/langgraph.md)）是架在其上的**有状态编排引擎**；Function Calling（见 [Function Calling / Tool Use](/contents/ai/function-calling.md)）是它驱动 Agent 的底层机制。

```
 你写的 LCEL 链
 ┌─────────────────────────────────────────┐
 │ Prompt → Model → OutputParser → …        │
 └─────────────────────────────────────────┘
         │
         ▼
   统一 Runnable 接口（invoke/stream/batch）
         │
         ▼
   LLM / 工具 / 检索 真正执行
```

> 前置阅读：[LLM 基础](/contents/ai/llm-basics.md)、[提示词工程](/contents/ai/prompt-engineering.md)。后续见 [Function Calling / Tool Use](/contents/ai/function-calling.md) 与 [LangGraph](/contents/ai/langgraph.md)。

---

### 什么是 LangChain？

LangChain 不是一个模型，而是一套**围绕 LLM 的应用开发库与生态**。它解决的核心问题是：把"调一次 API"升级成"组织多次调用 + 外部数据 + 工具 + 状态"的工程化流程。

它已发展为一组分工明确的包：

| 包 | 职责 |
|----|------|
| **langchain-core** | 基础抽象（Runnable、Prompt、Message、Document 等） |
| **langchain** | 链（Chains）、Agent、检索等高层组合 |
| **langchain-community** | 社区第三方集成（向量库、加载器等） |
| **langchain-openai** 等 partner 包 | 各家模型/服务的官方适配 |
| **LangGraph** | 有状态、多角色的工作流编排（见 [LangGraph](/contents/ai/langgraph.md)） |
| **LangServe** | 把链部署成 HTTP 服务 |
| **LangSmith** | 追踪、评估、可观测性 |

### 核心抽象

LangChain 把 LLM 应用拆成可复用的零件：

| 抽象 | 作用 | 对应文章 |
|------|------|----------|
| **Model I/O** | Prompt（模板）+ Model（聊天模型）+ Output Parser（解析输出） | [提示词工程](/contents/ai/prompt-engineering.md) |
| **Retrieval** | Document / TextSplitter / Embedding / VectorStore / Retriever | [RAG](/contents/ai/rag.md)、[Embeddings](/contents/ai/embeddings-vector-db.md) |
| **Memory** | 对话历史等状态 | [LLM 基础](/contents/ai/llm-basics.md) |
| **Tools** | 模型可调用的外部函数 | [Function Calling](/contents/ai/function-calling.md) |
| **Chains** | 把上述零件用管道串起来（LCEL） | 本文 |
| **Agents** | 让模型自主决定调用哪些工具、几步完成 | [AI Agent](/contents/ai/ai-agent.md) |

### LCEL：用管道组合 LLM 调用

**LCEL（LangChain Expression Language）** 是 LangChain 推荐的"组合语言"：用 `|` 把多个 **Runnable** 串成一条链，整条链本身也是一个 Runnable，统一支持 `invoke / stream / batch / async`。

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("把下面句子翻译成英文：{text}")
model = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

chain = prompt | model | parser   # 三个 Runnable 串成一条链
print(chain.invoke({"text": "今天天气真好"}))
```

关键点：

- 每个零件都是 `Runnable`，可单独测、可复用。
- 链可以嵌套、并行、分支，复杂度随需求增长而增长。
- `stream` 支持逐字输出，适合长文本。

### 提示与模型 I/O

- **PromptTemplate**：把变量安全注入提示（见 [提示词工程](/contents/ai/prompt-engineering.md)）。
- **ChatModel**：封装各家聊天模型，消息用 `SystemMessage / HumanMessage / AIMessage / ToolMessage` 表示。
- **OutputParser**：把模型文本解析成结构化对象（如 JSON、Pydantic）。

### 检索（Retrieval）

LangChain 把 RAG 需要的零件标准化：

- `Document`：带元数据的文本片段。
- `TextSplitter`：切分策略（见 [RAG](/contents/ai/rag.md)）。
- `Embeddings` / `VectorStore`：向量化与存储（见 [Embeddings](/contents/ai/embeddings-vector-db.md)）。
- `Retriever`：统一的"给问题、返回相关文档"接口。

### 记忆（Memory）

对话类应用需要跨轮记住历史。现代写法多用 **消息列表** 直接拼进 prompt，或用 `RunnableWithMessageHistory` 把历史管理外包；有状态、可续跑的场景则交给 [LangGraph](/contents/ai/langgraph.md) 的 Checkpointer。

### Agents 与 Tools

Agent = 模型 + 工具 + 循环决策。LangChain 提供预置 Agent（`create_react_agent` 等），底层就是 [Function Calling](/contents/ai/function-calling.md)：模型决定调哪个工具、传什么参，运行时执行后把结果喂回模型继续。

### 周边生态

| 组件 | 用途 |
|------|------|
| **LangServe** | 一条链 → 一个 FastAPI 服务，便于部署 |
| **LangSmith** | 记录每次调用的链路、做评估与回归（见 [AI 安全与防护](/contents/ai/ai-safety.md) 的监控） |
| **LangGraph** | 需要循环、分支、状态、人工介入时上它 |

### 与 LangGraph 的关系

二者**互补分层**：

- **LangChain**：负责"怎么调用模型、提示、检索、工具"（积木与管道）。
- **LangGraph**：负责"按什么顺序、在什么状态下、循环多少次调用"（编排引擎）。

简单线性链用 LCEL 即可；一旦需要**环、分支、多 Agent、断点续跑、人工介入**，就交给 LangGraph（见 [LangGraph](/contents/ai/langgraph.md)）。

### 何时用 LangChain？

- ✅ 想快速搭 RAG / 对话 / 简单 Agent，不想自己写大量胶水代码。
- ✅ 需要统一的多模型、多向量库、多工具适配层。
- ⚠️ 若只是偶尔调一次 API、逻辑很简单，直接用官方 SDK 可能更轻。
- ⚠️ 复杂有状态工作流，优先 LangGraph 而非手写链。

### 相关资源

- [LLM 基础](/contents/ai/llm-basics.md) / [提示词工程](/contents/ai/prompt-engineering.md) — 前置概念
- [Function Calling / Tool Use](/contents/ai/function-calling.md) — Agent 的底层机制
- [RAG（检索增强生成）](/contents/ai/rag.md) — LangChain 的检索零件
- [LangGraph](/contents/ai/langgraph.md) — 其上的编排引擎
- [LangChain 文档](https://python.langchain.com/)
