---
id: "20260428190004"
type: summary
tags:
  - "retrieval"
  - "RAG"
  - "embedding"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Contextual Retrieval in AI Systems.md"
confidence: 0.9
---

## 核心结论

1. **上下文检索将前 20 块检索失败率降低 49%**（5.7%→2.9%），结合重排序后降低 67%（5.7%→1.9%）。核心创新：在嵌入前为每个 chunk 添加特定于该 chunk 的解释性上下文。

2. **BM25 + 语义嵌入组合优于单独使用嵌入**：BM25 处理精确匹配（如唯一标识符、技术术语），嵌入处理语义相似性，两者互补。

3. **提示缓存使上下文检索成本极低**：生成 800 token chunk + 8000 token 文档上下文的一次性成本为 **$1.02/百万文档 token**。

## 支撑证据

- ACME SEC 文件示例：原句"The company's revenue grew by 3%"缺乏上下文；加上下文后变为"This chunk is from an SEC filing on ACME corp's performance in Q2 2023; the previous quarter's revenue was $314 million..."
- Gemini Text 004 和 Voyage 嵌入模型在该方法下表现最佳
- 结合重排序后 top-20 块检索失败率从 5.7% 降至 1.9%

## 关键方法

### 上下文检索流程
1. 文档分成 chunk（通常几百 token）
2. 用 Claude 3 Haiku 为每个 chunk 生成上下文（50-100 token）
3. 将上下文 + chunk 一起嵌入 + 创建 BM25 索引
4. 检索时使用融合排序合并 BM25 和嵌入结果
5. 取 top-K（如 20）送入模型

### 重排序流程
1. 初始检索取 top-150
2. 重排模型对 top-150 + 用户 query 评分
3. 取 top-20 送入模型生成最终答案

### 实施注意事项
- **块边界**：影响检索性能，需实验
- **嵌入模型**：Gemini/Voyage 表现最佳
- **K 值**：top-20 优于 top-10/top-5
- **提示**：领域定制提示可能优于通用提示

## 质疑

- **Claude 3 Haiku 生成上下文的成本 vs 质量**：便宜但可能不如更大模型生成的上下文精确
- **数据集规模阈值**：知识库 <20 万 token 时直接放入上下文更简单（提示缓存后延迟降低 2x，成本降低 90%）
- **评估指标的局限性**：仅用 recall@20 可能无法捕获其他质量维度（如答案完整性）

## 跨领域迁移

- **其他 RAG 变体**：HyDE（假设文档嵌入）、基于摘要的索引之前已有人尝试，效果不如上下文检索
- **结构化数据检索**：SEC 文件、医疗记录等高度结构化的文档集特别适合上下文检索
- **多跳推理**：上下文检索可作为多跳推理系统的第一跳，为后续推理提供更好上下文
