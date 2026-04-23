---
id: "20260423180003"
type: concept
tags: [RAG, 检索增强生成, 向量数据库, PDF问答]
status: draft
aliases: [检索增强生成, Retrieval-Augmented Generation]
created: 2026-04-23
updated: 2026-04-23
source: raw/notes/Agent_Loading/阶段三.md
confidence: 0.85
---

# RAG

检索增强生成（Retrieval-Augmented Generation），一种结合检索与生成的架构模式。

## 定义

不依赖模型内部知识，而是通过检索外部文档增强生成质量。

## 两阶段架构

### 索引阶段

```
文档 → 清洗 → 分块 → 向量化 → 向量数据库
```

关键组件：
- **文本清洗**：`clean_rag_text()` 处理 PDF 排版问题
- **分块策略**：`chunk_size=800, overlap=200`，多级分隔符（段落→句子→标点）
- **向量化**：`SentenceTransformer` 或 `GoogleGenerativeAIEmbeddings`
- **存储**：Chroma（持久化）vs InMemoryVectorStore（临时）

### 检索阶段

```
用户查询 → 改写/扩充 → 向量化 → 相似度搜索 → 文档片段 → LLM 生成
```

关键组件：
- **查询改写**：提高检索覆盖度
- **相似度度量**：余弦相似度、欧几里得距离
- **Top-K 检索**：`k=3` 或 `k=5`

## 项目对比

| 维度 | 简单版 | 生产级 |
|------|--------|--------|
| 向量存储 | InMemoryVectorStore | Chroma持久化 |
| 分块大小 | 200 | 800 |
| 记忆能力 | 无 | MemoryManager |
| 元数据 | 无 | 页码、来源、chunk_index |

## 关键数据点

- **分块策略**：800字符 + 200重叠是生产级基线
- **Top-K**：通常 k=3~5
- **向量维度**：依赖嵌入模型（如 paraphrase-multilingual-MiniLM-L12-v2 输出 384 维）

## 前提与局限性

- **检索质量依赖分块**：chunk_size 过大导致上下文碎片化，过小丢失关联
- **嵌入模型依赖**：英文用 OpenAI Embeddings，中文需多语言模型
- **无法处理复杂推理**：多跳问题需要 GraphRAG

## 关联概念

- [[LCEL]]：RAG 链用 LCEL 实现
- [[MemoryManager]]：会话记忆组件
- [[向量数据库]]：Chroma 是生产级选择

---

**来源**：`阶段三.md`