---
id: "20260428190010"
type: summary
tags:
  - "indexing"
  - "llamaindex"
  - "RAG"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Indexing.md"
confidence: 0.8
---

## 核心结论

1. **索引是将 Document 对象列表转换为可查询数据结构的核心步骤**，LlamaIndex 中 `VectorStoreIndex` 通过将文本转为向量嵌入实现语义搜索。

2. **嵌入（Embedding）是文本的数值表示**，语义相似的文本具有数学上相似的嵌入，这使得"按含义搜索"而非"按关键词匹配"成为可能。

3. **Top-K 检索是最简单的向量索引查询方式**：将查询转为嵌入，计算与所有嵌入的语义相似度，返回 top-K 最相似的 chunk。

## 支撑证据

- LlamaIndex 默认使用 `text-embedding-ada-002`（OpenAI 默认嵌入），不同 LLM 可能需要不同嵌入模型
- 生成嵌入需要多次 API 调用，大文档集可能耗时较长且成本较高
- Summary Index 简单返回所有文档，适用于需要生成摘要而非定位特定信息的场景

## 关键方法

### VectorStoreIndex 用法
```python
from llama_index.core import VectorStoreIndex
index = VectorStoreIndex.from_documents(documents)
# 或基于 nodes
index = VectorStoreIndex(nodes)
```

### 嵌入生成流程
1. Document → Nodes（分块）
2. 对每个 node 调用嵌入 API 生成向量
3. 存储向量在向量数据库
4. 查询时将 query 转为向量，计算相似度，返回 top-K

### Top-K 参数
- `k`：返回的嵌入数量
- `top_k`：控制返回多少最相似嵌入

## 质疑

- **嵌入成本**：大文档集嵌入耗时且贵，需要考虑缓存策略
- **块大小影响**：块大小、边界和重叠选择影响检索性能，需实验
- **Embedding 模型选择**：不同模型效果、效率和成本差异显著

## 跨领域迁移

- **其他向量数据库**：Pinecone、Weaviate、ChromaDB 等都基于类似向量嵌入原理
- **混合检索**：结合 BM25（稀疏）和嵌入（稠密）的混合检索策略正成为标准
- **重排序层**：在 top-K 检索后加重排序模型（如 Cohere reranker）可进一步提升精度
