# AIWIKI - 个人AI知识库

> 将AI领域原始资料编译为结构化知识的学习库

> ⚠️ **知识库仍在扩展中** - 部分概念尚未完整编译，内容会持续更新

## 目录结构

```
research/
├── raw/           # 原始资料（只读不改）
│   ├── articles/  # 网页剪藏
│   ├── notes/     # 个人笔记
│   ├── reports/   # 研报与新闻
│   └── tasks/     # 待办任务
└── wiki/          # 编译产物（结构化输出）
    ├── concepts/  # 原子化概念条目
    ├── summaries/ # 逐篇精简摘要
    ├── methods/   # 方法论沉淀
    ├── projects/  # 任务产出
    └── indexes/   # 全局索引
```

## 核心概念

| 概念 | 说明 |
|------|------|
| [[Agent设计思维]] | Agent vs 传统程序的控制范式转变 |
| [[LCEL]] | LangChain表达式语言 |
| [[RAG]] | 检索增强生成 |
| [[工具调用框架]] | Function Calling设计模式 |
| [[大模型基础认知]] | Transformer架构基础 |

## 编译流程

1. **raw/** 中的原始笔记 → 编译摘要到 **wiki/summaries/**
2. 高价值概念 → 沉淀到 **wiki/concepts/**
3. 全局索引 **wiki/indexes/index.md** 自动追踪知识关联

## 学习路径

阶段0 → 阶段1 → 阶段2 → 阶段3 → 阶段4 → 阶段5 → 阶段6

覆盖：Python基础 → 大模型原理 → LangChain/LlamaIndex → Agent核心范式 → 生产级工程

## 技术栈

- **Obsidian** - 笔记管理与双向链接
- **Dataview** - 动态索引查询
- **Tasks** - 行动追踪
- **Git** - 版本控制

---
*最后更新：2026-04-23*
