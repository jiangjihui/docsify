# RAG（检索增强生成）

## 概览

**RAG（Retrieval-Augmented Generation，检索增强生成）** 是当前最主流的"让 LLM 用上私有/最新知识"的架构。它把"检索"与"生成"结合：先根据用户问题从知识库**检索相关片段**，再把片段喂给 LLM **生成**有依据的回答。

RAG 的核心价值在于：**不重训模型**，就能让通用 LLM 回答私有文档、最新信息、专业领域问题，并显著降低幻觉。

```
 离线（知识入库）                  在线（问答）
 ┌──────────────┐              ┌──────────────────────────┐
 │ 文档 → 切分   │              │ 用户问题                  │
 │ → 向量化      │              │   → 向量化                │
 │ → 存入向量库  │              │   → 相似检索 top-k        │
 └──────────────┘              │   → 拼进 prompt           │
                               │   → LLM 生成（带引用）     │
                               └──────────────────────────┘
```

> 前置阅读：[LLM 基础](/contents/ai/llm-basics.md)、[Embeddings 与向量数据库](/contents/ai/embeddings-vector-db.md)。本分类另见 [提示词工程](/contents/ai/prompt-engineering.md)。

---

### 什么是 RAG？

RAG 由 Facebook AI 在 2020 年提出，最初用于知识密集型 NLP 任务。其思想是：让生成模型在回答前，先**从外部知识源检索证据**，再把证据作为上下文参与生成。

典型问答流程：

```
问题："公司年假政策是怎样的？"
  → 检索到《员工手册》第 8 章片段
  → LLM 基于该片段作答，并可标注出处
```

相比"让模型凭记忆回答"，RAG 的答案**可追溯、可更新、可控**。

### 为什么需要 RAG？

| 维度 | 仅用 LLM（靠记忆） | RAG |
|------|-------------------|-----|
| 私有知识 | 不知道 | 可接入内部文档 |
| 知识时效性 | 受训练截止限制 | 随时更新知识库即可 |
| 幻觉 | 易编造 | 有检索依据，可引用 |
| 成本 | 需微调才有领域能力 | 无需训练，改知识库即可 |
| 可解释 | 难追溯 | 可附出处片段 |

> 一句话：当你需要的是**"基于特定资料的准确回答"**而非"通用创作"，优先 RAG 而非微调模型。

### RAG 总体架构

RAG 分**离线**与**在线**两条链路：

```
离线 Ingestion：
  原始文档 → 加载(Loader) → 清洗 → 切分(Chunk) → Embedding → 向量库

在线 Retrieval + Generation：
  问题 → Embedding → 向量检索(top-k) → [重排 Rerank] → 拼装 Prompt → LLM → 回答
```

下面分别展开两阶段的关键设计。

### 离线阶段：知识入库（Ingestion）

**1. 加载与清洗**：从 PDF / Word / 网页 / 数据库读取文本，去除噪声。

**2. 切分（Chunking）**：把长文档切成适合检索的片段，是 RAG 效果的关键。

| 切分策略 | 做法 | 适用 |
|----------|------|------|
| 固定长度 | 按 token/字符数切，带重叠 | 通用、简单 |
| 按结构 | 按标题/段落/表格切 | 结构清晰的文档 |
| 语义切分 | 按语义边界（模型辅助）切 | 长文、叙述性内容 |
| 父子切分 | 大块建索引、小块用于生成 | 兼顾召回与精度 |

**3. 向量化（Embedding）**：每个 chunk 用 Embedding 模型转成向量（见 [Embeddings](/contents/ai/embeddings-vector-db.md)）。

**4. 存储**：向量 + 原文 + 元数据（来源、页码）写入向量数据库。

### 在线阶段：检索与生成（Retrieval & Generation）

1. **问题向量化**：用同一 Embedding 模型把用户问题转向量。
2. **向量检索**：在向量库中做 ANN 搜索，取相似度最高的 top-k 个 chunk。
3. **（可选）重排**：用 Cross-Encoder 对候选精排，提升相关片段排序。
4. **拼装 Prompt**：把问题与检索片段按模板组合（含来源占位）。
5. **生成**：LLM 基于上下文作答，尽量引用来源。

### 检索增强的进阶形态

朴素 RAG（检索→拼接→生成）常遇"检索不准、拼太多、答非所问"等问题。进阶做法：

- **查询改写（Query Rewriting）**：把口语问题改写成更适合检索的语句。
- **混合检索（Hybrid）**：向量检索 + 关键词（BM25）结合，兼顾语义与精确词。
- **重排序（Rerank）**：用 Cross-Encoder 对 top-k 精排。
- **父子/窗口检索**：用小块召回、大块喂给模型。
- **元数据过滤**：先按时间/来源/权限过滤，再向量检索。
- **Self-RAG**：让模型自己决定"是否需要检索、检索是否够用"。

### 重排序（Rerank）

向量检索（Bi-Encoder）快但精度有限；**重排模型（Cross-Encoder）** 把"问题+候选"联合编码，相关性判断更准，但较慢，故通常只对 top-k（如 20→5）做精排，是"召回—精排"两段式标配。

### 简单示例

用 LangChain + Chroma 实现一个最小 RAG：

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_core.documents import Document

# 1) 准备文档并切分
docs = [Document(page_content="年假政策：入职满1年享5天，满10年享10天。")]
splitter = RecursiveCharacterTextSplitter(chunk_size=200, chunk_overlap=20)
chunks = splitter.split_documents(docs)

# 2) 向量化并入库
vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings())

# 3) 检索 + 生成
llm = ChatOpenAI(model="gpt-4o-mini")
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})
question = "入职满3年有多少天年假？"
context = "\n".join(d.page_content for d in retriever.invoke(question))
prompt = f"根据资料回答：\n{context}\n\n问题：{question}"
print(llm.invoke(prompt).content)
```

> 真实项目多用 `LangChain` 的 `RetrievalQA` / `create_retrieval_chain`，或 LlamaIndex 等框架封装上述链路。

### 评估 RAG

RAG 需同时评估**检索质量**与**生成质量**：

| 维度 | 指标 | 说明 |
|------|------|------|
| 检索 | Recall@k / MRR | 相关片段是否被召回、排得够靠前 |
| 生成忠实度 | Faithfulness | 回答是否忠于检索内容（不编造） |
| 回答相关 | Answer Relevance | 是否切题 |
| 端到端 | 人工/LLM 打分 | 综合可用性 |

常用工具：Ragas、TruLens、LangSmith 等。

### 典型使用场景

- **企业知识库 / 文档问答**：基于内部手册、Wiki 回答。
- **客服与助手套件**：结合订单/工单数据答疑。
- **法规 / 合规咨询**：用最新条文作答并附出处。
- **代码库问答**：检索私有代码与注释。
- **研究综述**：聚合多篇文献生成带引用的综述。

### 相关资源

- [Embeddings 与向量数据库](/contents/ai/embeddings-vector-db.md) — RAG 的检索底座
- [LLM 基础](/contents/ai/llm-basics.md) — 模型与上下文窗口
- [提示词工程](/contents/ai/prompt-engineering.md) — 检索结果的拼装与指令
- [AI Agent（智能体）](/contents/ai/ai-agent.md) — Agent 中的 RAG 工具
- [LangChain RAG 文档](https://python.langchain.com/docs/tutorials/rag/)
