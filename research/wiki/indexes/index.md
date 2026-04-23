---
id: "20260423180012"
type: index
title: AI知识库全局索引
tags: [index, 知识库]
status: evergreen
created: 2026-04-23
updated: 2026-04-23
---

# AI知识库全局索引

## 📊 概念卡片

```dataview
TABLE file.ctime as 创建时间, status, tags
FROM "research/wiki/concepts"
WHERE status = "evergreen" or status = "draft"
LIMIT 20
SORT file.ctime DESC
```

## 📝 最近编译

```dataview
TABLE source, confidence
FROM "research/wiki/summaries"
LIMIT 10
SORT file.ctime DESC
```

## 🎯 学习路径（来自 导航.md）

| 阶段 | 周期 | 核心内容 |
|------|------|----------|
| 阶段0 | 1周 | 环境与思维预热 |
| 阶段1 | 2周 | Python快速掌握 |
| 阶段2 | 2周 | 大模型原理与调用 |
| 阶段3 | 4周 | LangChain/LlamaIndex精通 |
| 阶段4 | 5周 | Agent核心范式与LangGraph |
| 阶段5 | 6周 | 生产级Agent工程 |
| 阶段6 | 4周 | 高级架构与创新 |

## 📚 核心概念

- [[LCEL]] - LangChain表达式语言
- [[RAG]] - 检索增强生成
- [[工具调用框架]] - Function Calling设计模式
- [[Agent设计思维]] - Agent vs 传统程序
- [[大模型基础认知]] - Transformer架构基础

---

*最后更新：2026-04-23*