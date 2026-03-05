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

## 相关资源

- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [MCP 规范](https://modelcontextprotocol.io)
- [LangChain 文档](https://python.langchain.com)
- [AutoGen 文档](https://microsoft.github.io/autogen)

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

# 启动服务
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
