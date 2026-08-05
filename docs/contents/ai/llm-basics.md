# LLM 基础

## 概览

大语言模型（Large Language Model, LLM）是当前 AI 应用的"发动机"。理解它的工作机制，是后续学习提示词工程、RAG、Agent、LangGraph 等内容的前提。

本文用最少的数学，把 LLM 的核心概念讲清楚：它本质上是一个**基于 Transformer 的"下一个 token 预测器"**，通过海量文本训练出语言与世界的统计规律，再以可调的采样参数生成文本。

```
        输入提示 (prompt)
              │
              ▼
   ┌──────────────────────┐
   │   Transformer 模型    │
   │  (自注意力 + 前馈)     │
   └──────────────────────┘
              │
              ▼
   逐 token 预测下一个词
   "今天" → "的" → "天气" → …
              │
              ▼
         生成结果 (completion)
```

> 本文是后续 Agent / LangGraph / DeerFlow 等文章的基础。配套阅读：[AI Agent（智能体）](/contents/ai/ai-agent.md)。

---

### 什么是大语言模型（LLM）？

大语言模型是一类基于深度学习（主要是 **Transformer** 架构）训练得到的语言模型，参数量通常在十亿（B）到万亿级别。它的训练目标是**自回归（autoregressive）地预测下一个 token**：给定前面已有的文本，最大化下一个 token 出现的概率。

```
P("天气" | "今天的")  →  最大
P("苹果" | "今天的")  →  较小
```

由于模型在预训练阶段"读"过极其庞大的文本语料，它事实上"记住"了大量语言结构、常识与领域知识，从而能在推理时通过"续写"完成问答、翻译、摘要、代码生成等任务。

### 核心架构：Transformer 与自注意力

现代 LLM 几乎都建立在 **Transformer**（2017, "Attention Is All You Need"）之上。其核心机制是 **自注意力（Self-Attention）**：模型在处理每个 token 时，会"看一眼"序列中其他所有 token，并按相关性加权聚合信息。

注意力计算可抽象为：

```
Attention(Q, K, V) = softmax(Q·Kᵀ / √d) · V
```

- **Q（Query）**：当前 token 想"问"什么
- **K（Key）**：每个 token 提供的"索引"
- **V（Value）**：每个 token 携带的"内容"
- **√d**：缩放因子，防止点积过大导致梯度消失

模型通常并行使用多组注意力（**多头注意力，Multi-Head Attention**），从不同子空间捕捉语法、指代、语义等关系。

按结构划分，LLM 主要有两类：

- **仅解码器（Decoder-only）**：如 GPT 系列、Llama、Qwen、DeepSeek。当前主流，擅长生成。
- **编码器-解码器（Encoder-Decoder）**：如 T5、BART。多用于翻译、摘要等序列到序列任务。

> 注：Embedding 模型多为**仅编码器（Encoder-only）**，如 BGE、BERT 系，详见 [Embeddings 与向量数据库](/contents/ai/embeddings-vector-db.md)。

### Token 与分词（Tokenization）

模型并不直接处理"字"或"词"，而是先把文本切成 **token**（模型词表中最基本的单位），再转为数字 id。主流分词算法是 **BPE（Byte-Pair Encoding）**。

- 英文中 1 个 token ≈ 0.75 个单词；中文通常 1 个汉字 ≈ 1~2 个 token。
- 计费与上下文长度都以 **token 数** 计，而非字符数。
- 同样的句子，不同模型分词结果不同（词表不同）。

```
"我爱北京"  →  ["我", "爱", "北京"]          (3 tokens，示意)
"machine learning" → ["machine", " learning"]  (2 tokens，示意)
```

理解 token 很重要：它直接影响**成本、延迟、上下文上限**，也解释了为什么长上下文会"吃掉"预算。

### 上下文窗口（Context Window）

**上下文窗口**指模型单次推理能"看到"的最大 token 数，包含**输入（prompt）+ 已生成的输出**之和。

- 早期模型仅 2K~4K；如今常见 32K、128K，甚至 200K+。
- 窗口内早期部分会被 **KV Cache** 缓存复用，提升推理效率；超出窗口的内容模型"看不到"。
- 长文档处理、超长对话都需要考虑窗口限制，这也是 RAG（见 [RAG](/contents/ai/rag.md)）存在的核心动机之一。

### 生成与采样参数

模型对下一个 token 输出的是**概率分布**。采样参数决定如何从分布中"挑一个词"，从而调节输出的随机性与质量。

| 参数 | 作用 | 调高 / 调低的经验 |
|------|------|------------------|
| **temperature** | 对概率分布做再缩放，控制随机性 | 高 → 更有创意/发散；低（→0）→ 更确定/保守、易重复 |
| **top_p**（核采样） | 只从累计概率达 `p` 的最小词集中采样 | 与 temperature 类似但更平滑可控；常取 0.9 |
| **top_k** | 只从概率最高的 `k` 个词中采样 | `k` 小→更聚焦；`k` 大→更多样；常与 top_p 二选一 |
| **max_tokens** | 单次生成的最大 token 数 | 控制长度与成本；设太小会被截断 |
| **frequency / presence penalty** | 抑制已出现词的重复 | 调高 → 更少重复用词、更多样 |

实践建议：**temperature 与 top_p 不要同时拉满**，二者都控制随机性，选其一精调即可；创意写作调高 temperature，代码/事实问答调低。

### 训练范式：预训练 / 微调 / 对齐

一个可用的 LLM 通常经历三个阶段：

| 阶段 | 目标 | 数据 | 典型方法 |
|------|------|------|----------|
| **预训练 Pre-training** | 学习通用语言规律与世界知识 | 海量无标注文本/代码 | 自监督（next-token） |
| **监督微调 SFT** | 学会按指令、按格式回答 | 高质量（指令, 回答）对 | 有监督微调 |
| **对齐 Alignment** | 让回答更安全、有用、符合人类偏好 | 人类偏好对比数据 | RLHF / DPO |

- **预训练**成本极高（千卡级、数周），一般只有大厂/机构能做。
- **SFT** 让模型"听得懂人话"，是大多数垂直模型的第一步。
- **RLHF/DPO** 通过人类反馈把模型行为对齐到期望价值（有用、诚实、无害）。

> 当你需要在"已有知识"和"私有/最新知识"之间做选择时，通常优先考虑 RAG 而非重训模型，见 [RAG](/contents/ai/rag.md) 中的对比。

### 主流模型家族（速览）

| 家族 | 代表模型 | 特点 |
|------|----------|------|
| OpenAI GPT | GPT-4o / o1 / o3 | 强通用能力，生态成熟，闭源商用 |
| Anthropic Claude | Claude 3.5 / 3.7 Sonnet | 长上下文、强推理与代码 |
| Google Gemini | Gemini 1.5 / 2.0 | 原生多模态、超长上下文 |
| Meta Llama | Llama 3.x | 开源可自部署，社区庞大 |
| 阿里 Qwen | Qwen2.5 系列 | 中英双语强，开源生态活跃 |
| 深度求索 DeepSeek | DeepSeek-V3 / R1 | 强推理（R1）、高性价比开源 |

选型时关注：**上下文长度、是否支持工具调用（Function Calling）、是否开源可私有化、多语言/中文能力、成本**。

### 局限与注意点

LLM 很强大，但有明确边界，工程落地时必须正视：

- **幻觉（Hallucination）**：模型可能"自信地编造"看似合理的错误事实。关键场景需 RAG + 引用 + 校验。
- **知识截止（Knowledge Cutoff）**：训练数据有时间边界，不了解之后的事件。
- **上下文 ≠ 记忆**：窗口内信息不等于真正"理解"，长文本易被"中间遗忘"（lost-in-the-middle）。
- **推理不稳定**：同一 prompt 不同温度可能得到不同结论；数学/逻辑需校验或借助工具。
- **偏见与安全**：训练数据含偏见，需对齐与护栏（Guardrails）。

这些局限正是 Agent、RAG、工具调用、人工介入等工程方案要解决的问题，也是本分类后续文章的出发点。

### 相关资源

- [AI Agent（智能体）](/contents/ai/ai-agent.md) — LLM 之上如何构建自主 Agent
- [提示词工程](/contents/ai/prompt-engineering.md) — 如何更好地"指挥"模型
- [RAG（检索增强生成）](/contents/ai/rag.md) — 用外部知识抑制幻觉
- [Embeddings 与向量数据库](/contents/ai/embeddings-vector-db.md) — RAG 的向量检索底座
- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) — 直观理解自注意力
