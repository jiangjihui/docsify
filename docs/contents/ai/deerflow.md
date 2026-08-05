# DeerFlow

## DeerFlow 概览

DeerFlow（**D**eep **E**xploration and **E**fficient **R**esearch **F**low）是字节跳动开源的、社区驱动的**深度研究（Deep Research）框架**，采用 MIT 许可证。它把大语言模型与搜索、抓取、Python 代码执行、RAG 知识库检索、MCP 工具调用整合在一起，通过**基于 LangGraph 的有状态图工作流**进行编排，将复杂的研究任务拆解成并行化的子任务流水线，最终由单一自然语言指令产出结构化报告、PPT 乃至 AI 播客音频等交付物。

> 本文配套阅读：[LangGraph](/contents/ai/langgraph.md)（DeerFlow 的编排底座）与 [AI Agent（智能体）](/contents/ai/ai-agent.md)（Agent 基础概念前置）。

```
┌──────────────────────────────────────────────────────┐
│                     DeerFlow                          │
│                                                      │
│   一个自然语言指令                                     │
│        │                                             │
│        ▼                                             │
│   ┌─────────┐   ┌────────┐   ┌──────────────────┐   │
│   │Coordinator├──▶│Planner │──▶│ Research Team    │   │
│   └─────────┘   └────────┘   │ ├ Researcher     │   │
│                              │ └ Coder          │   │
│                              └──────────────────┘   │
│                                       │             │
│                                       ▼             │
│                              ┌──────────────────┐   │
│                              │    Reporter      │   │
│                              └──────────────────┘   │
│                                       │             │
│                 交付物：报告 / PPT / 播客音频        │
└──────────────────────────────────────────────────────┘
```

---

### 什么是 DeerFlow？

一句话：**DeerFlow 是一个"研究型超级智能体（SuperAgent）编排框架"**——它由多个专职 Agent 协作完成长周期、多步骤的研究与执行任务，而不是像普通聊天机器人那样"问完即止"。

- **定位**：不是聊天界面，而是把 LLM 与外部工具组合成研究与执行系统；2.0 主线进一步演进为 full-stack Super Agent。
- **出身**：字节跳动开源，社区驱动；1.x 为经典 Deep Research 图，2.0 是一次从头重写（主线与 1.x 不共享代码，1.x 仍保留在单独分支）。
- **底座**：编排层建立在 [LangGraph](/contents/ai/langgraph.md) 之上，复用 LangChain 的 LLM / Tool / Prompt 原语。

### 核心架构与工作原理

#### 分层技术栈

DeerFlow 采用清晰的分层架构，每一层职责单一：

| 层级 | 技术 | 作用 |
|------|------|------|
| 编排层 | **LangGraph** | 有状态多 Agent 工作流管理（图、Checkpoint、Human-in-the-loop） |
| LLM 交互 | LangChain + **litellm** | 模型调用、链式处理、模型无关抽象 |
| 执行环境 | **Docker 隔离沙箱** | 真实执行命令、文件系统、装包跑代码 |
| 后端 | Python | 核心逻辑与工作流图定义 |
| 前端 | Next.js（Node.js 22+） | 可视化交互界面 |

```
┌─────────────────────────────────────────────┐
│  前端 Next.js  │  后端 Python (src/)         │
│  (Web UI)     │  ┌───────────────────────┐  │
│               │  │   LangGraph 编排层    │  │
│               │  │  ┌───┐ ┌───┐ ┌───┐   │  │
│               │  │  │Coord│Planner│...│  │  │
│               │  │  └───┘ └───┘ └───┘   │  │
│               │  └───────────────────────┘  │
│               │         │         │         │
│               │    litellm    Docker 沙箱    │
│               │   (多模型)   (执行/文件)     │
└─────────────────────────────────────────────┘
```

#### 层级化多 Agent 角色系统

DeerFlow 把复杂研究拆成职责清晰的流水线，内置五类专职角色。每类角色**只访问与自身职责相关的工具**，符合最小权限原则：

| 角色 | 职责 | 典型工具 |
|------|------|---------|
| **Coordinator（协调器）** | 入口，管理生命周期，分类请求并路由（闲聊直接回复，研究问题转交 Planner） | 路由逻辑 |
| **Planner（规划器）** | 任务拆解，生成结构化多步计划；判断上下文是否充足，决定是否重新规划 | 计划校验（Pydantic） |
| **Researcher（研究员）** | 信息检索：搜索、抓取、私有知识库检索、MCP | Tavily/Brave/Arxiv、Jina、RAG、MCP |
| **Coder（编码员）** | 代码分析、Python REPL 执行、技术任务 | Python 解释器、文件读写 |
| **Reporter（报告员）** | 汇总 findings，生成带引用的结构化报告 | 报告模板、tiptap 编辑 |

#### 有状态图工作流（Workflow）

经典 DeerFlow 1.x 的状态图流转如下，整体是一张 LangGraph 有状态图：

```
coordinator
    │
    ▼
(background_investigator)   # 可选：先做一次初步搜索为规划打底
    │
    ▼
planner  ──▶ human_feedback  # 中断，让用户接受/自然语言修改/自动接受计划
    │                              │ (EDIT_PLAN 则回流 planner)
    ▼                              │
research_team  ── 调度每一步 ──▶ researcher / coder（各自 ReAct 循环）
    │           收集结果回共享 State
    ▼
reporter  ──▶  最终报告
```

关键机制：
- **Plan-and-Execute**：Planner 先产出显式、类型化的多步计划，再分步骤执行，规划与执行解耦。
- **Orchestrator-Workers**：`research_team` 是监督者，把每个计划步骤派发给专职 worker，结果回收进共享 State。
- **Human-in-the-loop**：`human_feedback` 节点会中断运行，用户可 `[ACCEPTED]` 接受、`[EDIT_PLAN]` 自然语言改写（回流 Planner）或开启自动接受。
- **ReAct**：Researcher / Coder 均为 ReAct Agent，推理与工具调用交替。
- **有界迭代**：受 `max_plan_iterations`、`max_step_num` 约束；上下文不足时 Planner 会重新规划。
- **Checkpoint**：LangGraph 的检查点机制让任务可在任意节点暂停/恢复，是 HITL 改计划的底层基础。

> 2.0 进一步演进为 **"单一 Lead Agent + 动态子代理"** 架构：主代理动态拉起各自独立上下文的子代理，支持串行（有依赖）与并行（无依赖）执行，最后汇总。

#### 关键设计要点

- **litellm 模型无关抽象层**：统一接入 100+ 模型（GPT-4、Claude、Qwen、Ollama 等），并支持**多档模型**——把强推理任务路由给更强模型、轻量任务用更便宜模型，兼顾质量与成本。
- **三级沙箱隔离**：`Local`（主机直接执行，低安全，本地开发）/ `Docker`（容器隔离，日常使用）/ `Kubernetes`（Pod 隔离，企业生产）。
- **持久化文件系统**：沙箱状态跨步骤保留，中间产物可自然复用；开启审批闸门后更贴合企业安全要求。

### 主要功能特性

| 特性 | 说明 |
|------|------|
| **LLM 集成（litellm）** | 统一接入 GPT-4 / Claude / Qwen 等百余模型，支持按 Agent 分配不同模型（多档） |
| **搜索与检索** | Tavily、Brave、DuckDuckGo、Arxiv、SearxNG；Jina 抓取与高级内容抽取 |
| **RAG 集成** | 对接 RAGFlow、VikingDB、Milvus、Qdrant 等私有知识库，输入框可直接 @ 文档 |
| **MCP 无缝接入** | 通过 MCP Server 扩展私有域访问、知识图谱、网页浏览等能力 |
| **Human-in-the-loop** | 支持用自然语言实时修订执行中的计划，也可开启自动接受 |
| **报告后编辑** | 基于 tiptap 的 Notion 式块编辑，支持 AI 润色、扩写、缩写 |
| **多模态交付物** | 原生产出结构化报告、PowerPoint 演示文稿、TTS 播客音频 |
| **Skills 技能系统** | Markdown 定义工作流，按需渐进加载（节省 Token），支持自定义 |
| **长时记忆** | 跨会话持久记忆用户偏好、写作风格、技术栈 |
| **智能澄清** | 多轮对话澄清模糊主题，提升研究精度（可配置开关） |
| **LangGraph Studio 调试** | 自带 `langgraph.json`，可实时可视化、调试工作流 |

### 典型使用场景

| 场景 | 说明 | 示例 |
|------|------|------|
| **深度研究与竞品分析** | 最强用例：自动生成检索词、并行抓取、交叉比对矛盾说法，产出带来源的报告 | "对比 2026 年主流 AI 代码编辑器的定价与定位" |
| **多步数据分析** | 沙箱 Python + Coder，加载数据集→清洗→统计→可视化→总结 | "分析 Titanic 数据集，找出影响生还率的关键因素并出图" |
| **自动化内容生产** | 研究 + 编码 + 报告协作，自动产出带图表/对比表的文章或 PPT | 技术对比长文草稿 |
| **代码审查与重构** | 一个 Agent 读代码、一个找问题、一个改并跑测试 | 对 Python/TS 代码库做预置审查 |
| **网页/应用搭建** | 在沙箱中装依赖、跑代码、生成可部署网页 | 从指令直接产出可运行 Web 应用 |

### 与同类工具的差异对比

| 维度 | DeerFlow | OpenAI Deep Research | Perplexity | Manus | LangGraph | AutoGen / CrewAI |
|------|----------|----------------------|------------|-------|-----------|------------------|
| 开源 / 可自托管 | ✅ MIT | ❌ 闭源 | ❌ 闭源 | ❌ 闭源 | ✅ | ✅ |
| 多智能体协作 | ✅ 5 角色/动态子代理 | 内部多 Agent（不可见） | ❌ 单轮检索 | ✅ | 需自建 | ✅ |
| MCP 协议 | ✅ | ❌ | ❌ | 部分 | ✅ | 部分 |
| RAG / 私有知识 | ✅ 多后端 | ❌ | ❌ | 部分 | ✅ 需自建 | 有缺口 |
| 人工介入（HITL） | ✅ 自然语言改计划 | ❌ | ❌ | 少量 | ✅ | 需自建 |
| 代码执行沙箱 | ✅ Docker 三级隔离 | ✅（托管） | ❌ | ✅ | 需自建 | 需自建 |
| 多模态交付物 | ✅ 报告/PPT/播客 | ✅ 报告 | ❌ | ✅ | ❌ | ❌ |
| 模型无关 | ✅ litellm | ❌ 仅 OpenAI | ❌ | 有限 | ✅ | ✅ |

**定位总结**：相比 OpenAI Deep Research / Perplexity 这类**闭源黑盒**，DeerFlow 的核心优势是**可自托管、可定制 LLM/工具链/知识策略**；相比 LangGraph 这类**编排积木**，DeerFlow 是**开箱即用的完整研究产品**；相比 AutoGen / CrewAI，它在 **RAG 集成与 MCP 支持**上更完整，且原生具备代码沙箱与多模态产出。

### 集成与部署说明

#### 环境 prerequisites

- Python 3.12+（部分文档写 3.10+，建议 3.12）
- Node.js 22+（前端）
- Docker（沙箱执行，必选）
- 相应 LLM / 搜索 API Key

#### 快速开始

```bash
# 1. 克隆官方仓库（原 archersama/deer-flow 已迁移至字节跳动官方组织）
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow

# 2. 安装依赖（后端）
pip install -r requirements.txt        # 或：uv pip install -e .

# 3. 配置环境变量
cp .env.example .env
#   编辑 .env，填入：OPENAI_API_KEY / ANTHROPIC_API_KEY / TAVILY_API_KEY
#   以及搜索引擎 SEARCH_API=tavily（可选 brave_search / duckduckgo / arxiv / searxng）

# 4. 启动开发服务（同时拉起前后端）
make dev          # macOS / Linux
# Windows 可用：bootstrap.bat -d   （建议在 Git Bash 中执行）
```

启动后访问 Web UI（端口以启动日志为准，常见为 `3000` 或 `2026`）。

#### 关键配置

- **`.env`**：LLM Key、搜索凭证、`SEARCH_API` 选择搜索引擎、`RAG_PROVIDER` 配置 RAGFlow 等私有知识库。
- **`conf.yaml`（或工作流 YAML）**：声明 Agent 角色、系统提示、允许使用的工具、模型后端，以及执行图（哪些 Agent 并行、哪些串行）。常见模式是把强模型（如 Claude Opus）给监督者、轻量模型（如 GPT-4o-mini）给研究员。

#### 项目结构（2.0 主线）

```
deer-flow/
├── src/            # Python 后端：LangGraph 工作流图定义
├── web/            # Next.js 前端（Web UI）
├── skills/         # 技能插件（Markdown 定义，支持 skills/custom/ 自定义）
├── docker/         # 容器配置
├── langgraph.json  # LangGraph Studio 配置（自动加载 .env）
└── conf.yaml       # 工作流 / 模型 / 工具配置
```

#### 调试与扩展

- **LangGraph Studio**：项目自带 `langgraph.json`，可执行 `langgraph dev`（需 `langgraph-cli[inmem]`）启动 Studio，实时可视化、逐步调试状态流转。
- **MCP 扩展**：在配置中接入私有 MCP Server，即可让 Researcher 访问企业内网、知识图谱等私有域。
- **自定义 Skills**：把 Markdown 工作流文件放入 `skills/custom/`，即可按需加载为新能力。

### 相关资源

- [DeerFlow GitHub（字节跳动官方）](https://github.com/bytedance/deer-flow)
- [LangGraph](/contents/ai/langgraph.md) — DeerFlow 的编排底座
- [AI Agent（智能体）](/contents/ai/ai-agent.md) — Agent 基础概念、工具、MCP、记忆前置阅读
- [LLM 基础](/contents/ai/llm-basics.md) / [提示词工程](/contents/ai/prompt-engineering.md) / [RAG](/contents/ai/rag.md) — 理解 DeerFlow 前的 AI 基础
- [LangChain 框架总览](/contents/ai/langchain.md) / [多智能体协作模式](/contents/ai/multi-agent-patterns.md) / [AI 安全与防护](/contents/ai/ai-safety.md) — DeerFlow 相关的框架与多 Agent 进阶主题
