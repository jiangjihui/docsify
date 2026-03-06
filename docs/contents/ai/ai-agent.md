# AI Agent

## AI Agent 基础概念

### 什么是 AI Agent？

AI Agent（AI 代理）是能够自主完成任务的智能系统，它不仅能回答问题，还能执行操作。

| 组件 | 作用 | 示例 |
|------|------|------|
| LLM (大语言模型) | "大脑"，负责思考和决策 | GPT-4、Claude、DeepSeek |
| 工具 (Tools) | "手脚"，执行具体操作 | 搜索、读写文件、执行代码 |
| 记忆 (Memory) | 存储上下文和偏好 | 用户偏好、对话历史 |
| 规划 (Planning) | 分解复杂任务 | 子任务拆分、反思 |

### 实际例子

一个完整的 Agent 平台通常包含：

```
┌─────────────────────────────────────────────┐
│              Agent 平台架构                   │
├─────────────────────────────────────────────┤
│  LLM: 可配置多个模型                        │
│  Tools: 文件操作、命令执行、Web 搜索、MCP   │
│  Memory: 用户偏好记忆、对话历史             │
│  Skills: 预置技能 (图片生成、代码审查等)   │
│  Sandbox: 安全代码执行环境                 │
└─────────────────────────────────────────────┘
```

---

## 为什么需要 Agent 而非直接用 LLM

### 直接用 LLM 的局限

| 问题 | 说明 |
|------|------|
| 上下文限制 | 对话历史太长会丢失信息 |
| 工具缺失 | 无法执行代码、访问文件、调用 API |
| 无状态 | 每次对话都是全新的 |
| 缺乏结构 | 复杂任务没有清晰的执行流程 |
| 无法持久化 | 生成的代码、文件需要手动保存 |

### 对比示例

| 操作 | 直接用 LLM | 用 Agent |
|------|-----------|---------|
| "帮我写个爬虫" | 只返回代码文字 | 直接写文件并运行 |
| "分析这个 Excel" | 无法操作 | 读取文件、分析、生成报告 |
| "查下 GitHub 最新 Issue" | 不知道 | 调用工具获取 |

### Agent 的解决方案

- **工具调用**：执行代码、访问 API、操作文件
- **记忆机制**：长期记忆 + 短期记忆
- **工作流编排**：多步骤任务规划
- **反思机制**：检查结果、修正错误

---

## Agent 框架介绍

### 主流框架对比

| 框架 | 特点 | 适用场景 |
|------|------|---------|
| **LangGraph** | 基于图的工作流，支持 checkpoint | 复杂多步骤任务 |
| **LangChain** | 简单易用，链式调用 | 快速原型 |
| **AutoGen** | 多代理协作 | 多代理系统 |
| **CrewAI** | 角色扮演，团队协作 | 团队任务 |

### LangGraph 核心概念

LangGraph 是构建 Agent 的框架，核心概念：

```
┌─────────────────────────────────────────────┐
│                    Graph                     │
│  ┌─────┐    ┌─────┐    ┌─────┐            │
│  │Node1│───▶│Node2│───▶│Node3│            │
│  └─────┘    └─────┘    └─────┘            │
│       ▲                                  │
│       │                                  │
│       └──────────────────────────────────┘
│              状态流转                       │
└─────────────────────────────────────────────┘
```

**核心概念**：

| 概念 | 说明 |
|------|------|
| **State** | 贯穿整个流程的数据 |
| **Node** | 处理逻辑的函数 |
| **Edge** | 节点之间的连接 |
| **Checkpoint** | 状态持久化，支持恢复 |

### 中间件 (Middleware)

在 LLM 调用前后执行额外处理：

| 中间件 | 功能 |
|--------|------|
| 数据预处理 | 格式化输入、注入上下文 |
| 结果处理 | 解析输出、过滤敏感信息 |
| 错误处理 | 重试、降级 |
| 日志记录 | 追踪调用历史 |

**典型中间件链**：

```
用户消息
    ↓
1. 初始化线程数据
2. 处理上传文件
3. 获取沙箱环境
4. 上下文太长？压缩它！
5. 自动生成标题
6. 任务跟踪 (计划模式)
7. 图片处理 (视觉模型)
8. 用户需要澄清？
    ↓
LLM Model Call
```

---

## 工具系统 (Tools)

### 什么是 Tool？

Tool 是 Agent 执行具体操作的能力，比如：
- 搜索网页
- 读取文件
- 执行代码
- 调用 API

### Tool 的实现

```python
from langchain.tools import BaseTool
from pydantic import BaseModel

class SearchInput(BaseModel):
    query: str

class WebSearchTool(BaseTool):
    name = "web_search"
    description = "搜索网页信息"
    args_schema = SearchInput

    def _run(self, query: str):
        # 实现搜索逻辑
        return search_results
```

### 工具配置示例

在配置文件中定义工具：

```yaml
tools:
  # Web 搜索
  - name: web_search
    group: web
    use: src.community.tavily.tools:web_search_tool

  # 文件操作
  - name: read_file
    group: file:read
    use: src.sandbox.tools:read_file_tool

  - name: write_file
    group: file:write
    use: src.sandbox.tools:write_file_tool

  # 命令执行
  - name: bash
    group: bash
    use: src.sandbox.tools:bash_tool
```

### 工具加载流程

```
配置文件
    │
    ▼
动态加载 Python 类
    │
    ▼
转换为 LangChain BaseTool
    │
    ▼
LLM 可以调用
```

---

## MCP (Model Context Protocol)

### 什么是 MCP？

MCP 是 AI 与外部系统交互的**标准协议**，由 Anthropic 提出。

```
┌─────────────┐      MCP       ┌─────────────┐
│   LLM/Agent │◀──────────────▶│  MCP Server │
│             │   JSON-RPC     │  (GitHub等) │
└─────────────┘                └─────────────┘
```

### MCP 架构

```
┌─────────────────────────────────────────────┐
│                   Client                      │
│  ┌───────────────────────────────────────┐  │
│  │  LLM/Agent                            │  │
│  └───────────────────────────────────────┘  │
│                    │                         │
│                    ▼                         │
│  ┌───────────────────────────────────────┐  │
│  │  MCP Client (SDK)                    │  │
│  └───────────────────────────────────────┘  │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│                MCP Server                    │
│  ┌───────────────────────────────────────┐  │
│  │  工具列表 (tools/list)                │  │
│  │  工具调用 (tools/call)               │  │
│  │  资源 (resources/*)                   │  │
│  │  提示 (prompts/*)                     │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### MCP Server 类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **stdio** | 子进程方式 | GitHub, Filesystem |
| **SSE** | Server-Sent Events | 远程服务 |
| **HTTP** | REST API | 自定义服务 |

### MCP 配置示例

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "$GITHUB_TOKEN"
      }
    },
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    }
  }
}
```

### 常用 MCP 服务

| 服务 | 功能 |
|------|------|
| GitHub | 仓库操作、Issue、PR |
| Filesystem | 本地文件访问 |
| PostgreSQL | 数据库操作 |
| Brave Search | 网页搜索 |
| Puppeteer | 浏览器自动化 |

---

## Tool vs MCP

### 对比

| 维度 | Tool | MCP |
|------|------|-----|
| 实现方式 | 自己写代码 | 使用现成服务 |
| 灵活性 | 高 | 低 |
| 维护 | 自己维护 | 第三方维护 |
| 协议 | 自定义 | MCP 标准 |

### 使用场景

| 场景 | 推荐 |
|------|------|
| 本地文件操作 | Tool |
| Web 搜索 | Tool |
| 连接外部服务 | MCP |
| 数据库操作 | MCP |

---

## 记忆系统 (Memory)

### 记忆类型

| 类型 | 说明 | 例子 |
|------|------|------|
| **短期记忆** | 当前对话上下文 | 对话历史 |
| **长期记忆** | 用户偏好、历史信息 | 个人资料、偏好设置 |

### 记忆实现方式

```
用户输入 ──▶ 提取关键信息 ──▶ 存储
                              │
                              ▼
                    ┌─────────────────┐
                    │  Vector Store  │
                    │  (向量数据库)  │
                    └─────────────────┘
                              │
                              ▼
              查询 ──▶ 相似度匹配 ──▶ 注入上下文
```

### 常用向量数据库

| 数据库 | 特点 |
|--------|------|
| Chroma | 轻量、易用 |
| Pinecone | 云服务、可扩展 |
| Weaviate | 开源、多模态 |
| Qdrant | 高性能 |

---

## 技能系统 (Skills)

### 什么是 Skill？

Skill 是预定义的提示词模板，让 Agent 具备特定能力。

```
┌─────────────────────────────────────────────┐
│              Skill 结构                      │
├─────────────────────────────────────────────┤
│  name: 技能名称                              │
│  description: 功能描述                        │
│  allowed-tools: 可用工具                      │
│  ─────────────────────────────────────────  │
│  # 技能指令                                 │
│  你是一个...                                 │
│  请按照以下步骤...                           │
└─────────────────────────────────────────────┘
```

### 常见技能

| 技能 | 功能 |
|------|------|
| 代码审查 | 分析代码问题 |
| 图片生成 | 调用 AI 生成图片 |
| 视频生成 | 调用 AI 生成视频 |
| 数据分析 | 处理和分析数据 |

---

## 沙箱系统 (Sandbox)

### 为什么需要沙箱？

Agent 执行代码可能存在风险：
- 恶意代码
- 资源耗尽
- 环境污染

### 沙箱方案

| 方案 | 特点 |
|------|------|
| Docker | 容器隔离 |
| WebAssembly | 轻量虚拟机 |
| E2B | 云沙箱服务 |
| 本地执行 | 开发环境 |

### 沙箱能力

| 能力 | 说明 |
|------|------|
| 命令执行 | 运行 shell 命令 |
| 文件操作 | 读写文件 |
| 网络访问 | HTTP 请求 |

---

## 子代理 (Subagent)

### 什么是 Subagent？

将复杂任务委托给子代理处理：

```
主 Agent
  │
  ├──▶ 子代理 A (代码任务)
  │
  ├──▶ 子代理 B (搜索任务)
  │
  └──▶ 子代理 C (分析任务)
```

### 适用场景

- 并行处理多个独立任务
- 专业领域任务
- 长流程任务

---

## 常见架构模式

### ReAct (Reason + Act)

```
思考 ──▶ 行动 ──▶ 观察 ──▶ 思考 ──▶ ...
```

示例：
```
用户：今天天气如何？

思考：我需要搜索天气信息
行动：调用 web_search("今天天气")
观察：获取到结果 "北京 25°C，晴"
思考：可以回答用户了
回答：今天北京天气晴朗，25°C
```

### Reflection

```
执行 ──▶ 反思 ──▶ 改进 ──▶ 重试
```

示例：
```
代码审查：
  执行：分析代码
  反思：发现内存泄漏
  改进：建议使用 context manager
  重试：重新审查
```

### Plan-and-Execute

```
规划 ──▶ 执行 ──▶ 评估
```

示例：
```
用户：帮我写个爬虫

规划：
  1. 分析目标网站结构
  2. 编写爬虫代码
  3. 运行测试
  4. 优化性能

执行：按步骤执行
评估：检查结果是否正确
```

---

## 一个完整的 Agent 平台示例：Deer Flow

前面介绍了各种概念，如何把它们组合成一个完整的 Agent 平台？

**Deer Flow** 是一个开源的 AI Agent 平台，完整实现了本笔记提到的所有概念：

```
Deer Flow = LangGraph + Tools + MCP + Memory + Skills + Sandbox + Subagents
```

### 核心特性

| 特性 | 实现方式 |
|------|---------|
| Agent 框架 | LangGraph |
| 工具系统 | config.yaml 配置 |
| 外部集成 | MCP (GitHub、Filesystem 等) |
| 记忆系统 | memory.json 持久化 |
| 技能扩展 | SKILL.md 插件 |
| 代码执行 | Docker 沙箱 |
| 子代理 | 并行任务执行 |

### 快速体验

```bash
# 克隆项目
git clone https://github.com/archersama/deer-flow.git

# 启动服务（window环境 使用：Git Bash 控制台执行下面的命令，否则会报错）
make dev

# 访问 http://localhost:2026
```

### 项目结构

```
deer-flow/
├── backend/          # Python 后端 (LangGraph)
├── frontend/         # Next.js 前端
├── skills/          # 技能插件 (15+ 种)
└── docker/          # 容器配置
```

通过研究 Deer Flow 的源码，可以深入理解本笔记中的各个概念是如何在实际项目中落地的。

---

## 从零构建 Agent：12 个进阶机制

本节参考 [learn-claude-code](https://github.com/anthropics/learn-claude-code) 项目，展示构建一个完整 Agent 系统的12个进阶机制。从最基础的循环开始，逐步叠加功能，最终形成一个完整的多智能体协作系统。

### s01: Agent Loop（智能体循环）

> *"一个工具 + 一个循环 = 一个智能体"*

Agent Loop 是整个系统的基础，将 LLM 与工具连接起来形成闭环：

```
+--------+      +-------+      +---------+
|  User  | ---> |  LLM  | ---> |  Tool   |
| prompt |      |       |      | execute |
+--------+      +---+---+      +----+----+
                    ^                |
                    |   tool_result  |
                    +----------------+
                    (loop until stop_reason != "tool_use")
```

**核心代码（约30行）：**

```python
def agent_loop(query):
    messages = [{"role": "user", "content": query}]
    while True:
        response = client.messages.create(
            model=MODEL, system=SYSTEM, messages=messages,
            tools=TOOLS, max_tokens=8000,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason != "tool_use":
            return  # 结束

        # 执行工具调用
        results = []
        for block in response.content:
            if block.type == "tool_use":
                output = run_tool(block.name, block.input)
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        messages.append({"role": "user", "content": results})
```

---

### s02: Tool Use（工具扩展）

> *"加一个工具，只加一个 handler"*

通过 dispatch map 模式，加工具不需要修改循环逻辑：

```
+--------+      +-------+      +------------------+
|  User  | ---> |  LLM  | ---> | Tool Dispatch    |
| prompt |      |       |      | {                |
+--------+      +---+---+      |   bash: run_bash |
                    ^           |   read: run_read |
                    |           |   write: run_wr  |
                    +-----------+   edit: run_edit |
                    tool_result | }                |
                                +------------------+
```

**路径沙箱：**

```python
def safe_path(p: str) -> Path:
    path = (WORKDIR / p).resolve()
    if not path.is_relative_to(WORKDIR):
        raise ValueError(f"Path escapes workspace: {p}")
    return path
```

---

### s03: TodoWrite（任务规划）

> *"先列步骤再动手，完成率翻倍"*

多步任务需要规划机制，防止丢失进度：

```
              +-----------+-----------+
              | TodoManager state     |
              | [ ] task A            |
              | [>] task B  <- doing  |
              | [x] task C            |
              +-----------------------+
                          |
              if rounds_since_todo >= 3:
                inject <reminder> into tool_result
```

**nag reminder：** 模型连续3轮以上不调用 `todo` 时注入提醒。

---

### s04: Subagents（子智能体）

> *"大任务拆小，每个小任务干净的上下文"*

子智能体用独立 messages[]，不污染主对话：

```
Parent agent                     Subagent
+------------------+             +------------------+
| messages=[...]   |             | messages=[]      | <-- fresh
|                  |  dispatch   |                  |
| tool: task       | ----------> | while tool_use:  |
|   prompt="..."   |             |   call tools     |
|                  |  summary    |   append results |
|   result = "..." | <---------- | return last text |
+------------------+             +------------------+
```

---

### s05: Skills（技能加载）

> *"用到什么知识，临时加载什么知识"*

两层注入机制，按需加载：

```
System prompt (Layer 1 -- always present):
+--------------------------------------+
| Skills available:                    |
|   - git: Git workflow helpers        |  ~100 tokens/skill
|   - test: Testing best practices     |
+--------------------------------------+

When model calls load_skill("git"):
+--------------------------------------+
| tool_result (Layer 2 -- on demand):  |
| <skill name="git">                   |
|   Full git workflow instructions...  |  ~2000 tokens
| </skill>                             |
+--------------------------------------+
```

---

### s06: Context Compact（上下文压缩）

> *"上下文总会满，要有办法腾地方"*

三层压缩策略：

```
[Layer 1: micro_compact]        (silent, every turn)
  Replace tool_result > 3 turns old
  with "[Previous: used {tool_name}]"

[Check: tokens > 50000?]
   no              yes
   |               |
   v        [Layer 2: auto_compact]
   |         Save transcript to .transcripts/
   |         LLM summarizes conversation.
   |               |
   |               v
   |        [Layer 3: compact tool]
   |          Model calls compact explicitly.
   +-------> Same summarization as auto_compact
```

---

### s07: Task System（任务系统）

> *"大目标要拆成小任务，排好序，记在磁盘上"*

持久化任务图（DAG），支持依赖管理：

```
.tasks/
  task_1.json  {"id":1, "status":"completed"}
  task_2.json  {"id":2, "blockedBy":[1], "status":"pending"}
  task_3.json  {"id":3, "blockedBy":[1], "status":"pending"}
  task_4.json  {"id":4, "blockedBy":[2,3], "status":"pending"}

任务图 (DAG):
                 +----------+
            +--> | task 2   | --+
            |    | pending  |   |
+----------+     +----------+    +--> +----------+
| task 1   |                          | task 4   |
| completed| --> +----------+    +--> | blocked  |
+----------+     | task 3   | --+     +----------+
                 | pending  |
                 +----------+

状态: pending -> in_progress -> completed
```

---

### s08: Background Tasks（后台任务）

> *"慢操作丢后台，agent 继续想下一步"*

守护线程执行慢命令，完成后注入通知：

```
Main thread                Background thread
+-----------------+        +-----------------+
| agent loop      |        | subprocess runs |
| ...             |        | ...             |
| [LLM call] <---+------- | enqueue(result) |
|  ^drain queue   |        +-----------------+
+-----------------+

Timeline:
Agent --[spawn A]--[spawn B]--[other work]----
             |          |
             v          v
          [A runs]   [B runs]      (parallel)
             |          |
             +-- results injected before next LLM call --+
```

---

### s09: Agent Teams（智能体团队）

> *"任务太大一个人干不完，要能分给队友"*

持久化队友 + JSONL 邮箱通信：

```
Teammate lifecycle:
  spawn -> WORKING -> IDLE -> WORKING -> ... -> SHUTDOWN

Communication:
  .team/
    config.json           <- team roster + statuses
    inbox/
      alice.jsonl         <- append-only, drain-on-read
      bob.jsonl

              +--------+    send("alice","bob","...")    +--------+
              | alice  | -----------------------------> |  bob   |
              | loop   |    bob.jsonl << {json_line}    |  loop  |
              +--------+                                +--------+
                   ^
                   |        BUS.read_inbox("alice")
                   +---- alice.jsonl -> read + drain ---------+
```

---

### s10: Team Protocols（团队协议）

> *"队友之间要有统一的沟通规矩"*

request-response 模式驱动所有协商：

```
Shutdown Protocol            Plan Approval Protocol
==================           ======================

Lead             Teammate    Teammate           Lead
  |                 |           |                 |
  |--shutdown_req-->|           |--plan_req------>|
  | {req_id:"abc"}  |           | {req_id:"xyz"}  |
  |                 |           |                 |
  |<--shutdown_resp-|           |<--plan_resp-----|
  | {req_id:"abc",  |           | {req_id:"xyz",  |
  |  approve:true}  |           |  approve:true}  |

FSM:
  [pending] --approve--> [approved]
  [pending] --reject---> [rejected]
```

---

### s11: Autonomous Agents（自治智能体）

> *"队友自己看看板，有活就认领"*

自组织机制，队友自己扫描任务看板：

```
Teammate lifecycle with idle cycle:

+-------+   tool_use     +-------+
| WORK  | <------------- |  LLM  |
+-------+                +-------+
    |
    | stop_reason != tool_use (or idle tool called)
    v
+--------+
|  IDLE  |  poll every 5s for up to 60s
+---+----+
    |
    +---> check inbox --> message? ----------> WORK
    |
    +---> scan .tasks/ --> unclaimed? -------> claim -> WORK
    |
    +---> 60s timeout ----------------------> SHUTDOWN

Identity re-injection after compression:
  if len(messages) <= 3:
    messages.insert(0, identity_block)
```

---

### s12: Worktree 任务隔离

> *"各干各的目录，互不干扰"*

git worktree 隔离，每个任务独立目录：

```
Control plane (.tasks/)             Execution plane (.worktrees/)
+------------------+                +------------------------+
| task_1.json      |                | auth-refactor/         |
|   status: in_progress  <------>   branch: wt/auth-refactor
|   worktree: "auth-refactor"   |   task_id: 1             |
+------------------+                +------------------------+
| task_2.json      |                | ui-login/              |
|   status: pending    <------>     branch: wt/ui-login
|   worktree: "ui-login"       |   task_id: 2             |
+------------------+                +------------------------+

State machines:
  Task:     pending -> in_progress -> completed
  Worktree: absent  -> active      -> removed | kept
```

**事件流：** 每个生命周期步骤写入 `.worktrees/events.jsonl`

---

## 相关资源

- [learn-claude-code 项目](https://github.com/anthropics/learn-claude-code) - 12个 Agent 进阶机制的完整实现
- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [MCP 规范](https://modelcontextprotocol.io)

