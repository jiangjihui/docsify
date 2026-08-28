# AI（人工智能）

> 本分类系统梳理大语言模型（LLM）与智能体（Agent）的实战知识：从基础概念、提示工程、RAG，到 Function Calling、LangChain、多智能体编排与安全防护，再到 LangGraph、DeerFlow 等可直接落地的框架与平台。目标是形成一条可循序渐进、边读边用的学习路径。

## 学习路径

建议按「基础 → 工程化 → 平台与实战」的顺序阅读，每一层都为下一层打地基：

### 基础（打地基）

- [LLM 基础](contents/ai/llm-basics.md) — Transformer、Token、上下文窗口、采样参数与训练范式
- [提示词工程](contents/ai/prompt-engineering.md) — Zero/Few-shot、CoT、ReAct 与结构化输出
- [RAG](contents/ai/rag.md) — 检索增强生成管线、分块、重排与评估
- [Embeddings 与向量数据库](contents/ai/embeddings-vector-db.md) — 向量嵌入、相似度度量与向量库选型

### 工程化（搭管道）

- [Function Calling / Tool Use](contents/ai/function-calling.md) — 让模型学会调用外部工具
- [LangChain 框架总览](contents/ai/langchain.md) — LCEL、检索、记忆与 Agent 编排
- [多智能体协作模式](contents/ai/multi-agent-patterns.md) — Supervisor / Orchestrator-Workers / Swarm 等
- [AI 安全与防护](contents/ai/ai-safety.md) — 提示注入、护栏、沙箱与人工介入

### 平台与实战（直接用）

- [Agent](contents/ai/ai-agent.md) — 智能体核心概念与 12 个进阶机制
- [LangGraph](contents/ai/langgraph.md) — 有状态图工作流编排
- [DeerFlow](contents/ai/deerflow.md) — 字节开源的深度研究框架

### AI 辅助编程（工具化）

- [AI 辅助编程总览](contents/ai/ai-assisted-coding.md) — Codex / Claude Code / Skills 横向对比与选型
- [Codex](contents/ai/codex.md) — OpenAI 的云端异步 + 终端 CLI 编码智能体
- [Claude Code](contents/ai/claude-code.md) — Anthropic 的终端同步编码智能体，7 种定制方式
- [Agent Skills（superpowers 等）](contents/ai/agent-skills.md) — 以 obra/superpowers 为例的技能复用体系

## 如何阅读

- **初次接触**：先读完「基础」4 篇，建立对 LLM / 提示 / 检索的共同语言。
- **准备做应用**：重点看「工程化」的 Function Calling 与 LangChain，再按需补 AI 安全。
- **要落地多 Agent**：直接进「平台与实战」，用 LangGraph 搭有状态工作流，或基于 DeerFlow 快速起一个深度研究原型。
- **想直接上手编码智能体**：看「AI 辅助编程（工具化）」分组，先横向对比 Codex / Claude Code / Skills，再按场景深入对应专页。

## 相关分类

- [实践 / AI Agent 实践](实践/AI智能体/基于LLM超级代理框架的领域增强定制实践.md) — 一个真实的「LLM 超级代理框架」领域落地案例，可与本分类互为印证。
