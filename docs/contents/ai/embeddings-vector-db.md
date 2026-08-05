# Embeddings 与向量数据库

## 概览

**Embedding（嵌入）** 是把文本、图片等内容映射成**高维浮点向量**的技术，语义相近的内容在向量空间中距离更近。它是 RAG、语义搜索、推荐、去重等能力的数学底座。

**向量数据库**则专门存储这些向量，并提供**近似最近邻（ANN）检索**，让我们在毫秒级从百万级向量中找到"最相似"的几个。

```
  "猫" ──┐
         ├─▶ [0.21, -0.83, 0.05, …]  (向量)
  "狗" ──┤      │ 语义相近 → 距离小
         │      ▼
  "汽车" ─┘  向量数据库 ANN 检索 → 返回 top-k 相似
```

> 前置阅读：[LLM 基础](/contents/ai/llm-basics.md)、[RAG（检索增强生成）](/contents/ai/rag.md)。

---

### 什么是 Embedding（嵌入）？

Embedding 是用神经网络把离散对象（词、句、文档、图）编码为**稠密向量（dense vector）** 的表示。核心性质：

- **语义压缩**：向量的方向/位置编码了语义信息。
- **可计算相似度**：用距离衡量"语义接近程度"。
- **同模型同空间**：同一模型产出的向量才可直接比较。

例如：向量(`"国王"`) − 向量(`"男人"`) + 向量(`"女人"`) ≈ 向量(`"女王"`)，这是词向量的经典类比性质。

### 相似度度量

| 度量 | 公式直觉 | 适用 |
|------|----------|------|
| **余弦相似度** | 两向量夹角余弦，范围 [-1,1] | 最常用，忽略向量长度 |
| **点积（Dot）** | 各维相乘求和 | 已归一化时等价于余弦；可含大小信息 |
| **欧氏距离** | 空间直线距离 | 需考虑绝对位置/幅度时 |

实践上 Embedding 通常**先归一化再算余弦/点积**，`score = 1` 表示最相似，`score = -1` 最不相似。

### 主流 Embedding 模型

| 模型 | 维度 | 特点 |
|------|------|------|
| OpenAI `text-embedding-3-small/large` | 1536 / 3072 | 易用、强泛化，闭源 |
| BGE（`bge-large-zh` 等） | 1024 | 中文强，开源可自部署 |
| E5 / multilingual-e5 | 多尺寸 | 多语言，开源 |
| Cohere Embed | 多档 | 多语言、企业级 API |
| Jina / Nomic | 多尺寸 | 长文本、开源 |

选型关注：**维度（影响存储与检索速度）、多语言/中文能力、是否开源可私有化、最大输入长度、领域适配性**。

> 注意：检索与入库**必须使用同一 Embedding 模型**，否则向量不在同一空间，相似度无意义。

### 什么是向量数据库？

传统数据库擅长"精确匹配"（WHERE id=1），但**不擅长"找最相似的"**。向量数据库为此而生：

- 存储 **向量 + 原文 + 元数据**。
- 提供 **ANN（Approximate Nearest Neighbor）检索**：牺牲极少精度换取数量级加速。
- 支持 **元数据过滤**（先按来源/时间过滤，再算相似度）。

### 近似最近邻索引（ANN）

| 索引 | 思路 | 特点 |
|------|------|------|
| **Flat（暴力）** | 逐一比对 | 100% 准确，但慢，仅适合小库 |
| **HNSW** | 多层可导航小世界图 | 快、内存占用大，最常用 |
| **IVF-PQ** | 聚类 + 乘积量化压缩 | 省内存，适合超大规模 |
| **Annoy** | 随机投影森林 | 只读、构建快 |

多数生产库默认 **HNSW**：在召回率与延迟间取得好平衡。

### 主流向量数据库对比

| 数据库 | 形态 | 特点 | 适用 |
|--------|------|------|------|
| **Chroma** | 轻量库/服务 | 上手快、Python 友好 | 原型、中小规模 |
| **Qdrant** | 服务 | Rust 编写、过滤强、易部署 | 生产、需强过滤 |
| **Milvus** | 分布式 | 十亿级、云原生 | 超大规模 |
| **Pinecone** | 全托管 SaaS | 免运维 | 不想自建 |
| **pgvector** | Postgres 扩展 | 复用现有 PG | 已有 PG、中等规模 |
| **Weaviate** | 服务 | 模块化、内置向量化 | 企业级 |

### 基本使用流程（代码）

以 Chroma 为例，演示"建索引 → 入库 → 查询"：

```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

embeddings = OpenAIEmbeddings()

# 1) 入库（文本自动向量化）
texts = ["猫喜欢睡觉", "狗喜欢跑步", "汽车靠汽油驱动"]
store = Chroma.from_texts(texts, embeddings, collection_name="demo")

# 2) 相似查询
hits = store.similarity_search("宠物犬", k=1)
print(hits[0].page_content)   # → "狗喜欢跑步"
```

### 选型建议

- **原型 / 学习**：Chroma（零配置）。
- **生产 + 已有 Postgres**：pgvector（少运维）。
- **生产 + 强元数据过滤**：Qdrant。
- **十亿级 / 分布式**：Milvus。
- **不想运维**：Pinecone / 云厂商托管版。

### 相关资源

- [RAG（检索增强生成）](/contents/ai/rag.md) — Embedding 在 RAG 中的直接应用
- [LLM 基础](/contents/ai/llm-basics.md) — 模型与上下文背景
- [提示词工程](/contents/ai/prompt-engineering.md) — 检索结果的拼装
- [Chroma 文档](https://docs.trychroma.com/) / [Qdrant 文档](https://qdrant.tech/documentation/)
