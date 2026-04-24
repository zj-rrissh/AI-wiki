---
id: "20260423180007"
type: summary
tags: [LangChain, Agent, 记忆机制, LCEL]
status: evergreen
aliases: []
created: 2026-04-23
updated: 2026-04-23
source: raw/notes/LangChain学习笔记.md
confidence: 0.85
---

# LangChain学习笔记 编译摘要

## 核心结论

1. **create_agent 是 LangChain 1.0 的核心抽象**
   - 统一了模型、工具、提示词、输出格式的组合方式
   - 支持 context_schema 实现运行时注入

2. **LCEL 是 LangChain 的声明式编程范式**
   - 通过 `|` 操作符串联组件
   - 所有组件遵循 Runnable 协议（invoke/batch/stream）

3. **记忆机制通过 checkpointer 实现持久化**
   - InMemorySaver 用于开发环境
   - 生产环境需要数据库持久化（PostgreSQL 等）

---

## 支撑证据

- **create_agent 示例**：model + system_prompt + tools + response_format + checkpointer
- **LCEL 三件套**：prompt | model | parser
- **checkpointer 配置**：thread_id 会话隔离

---

## 质疑

1. **版本迁移**：`langchain-classic` 的使用场景需要明确
2. **记忆持久化**：InMemorySaver 仅适用于开发，生产需要外置存储
3. **结构化输出**：Pydantic vs dataclass 的选择建议缺失

---

## 关联概念

- [[LCEL]]
- [[Agent设计思维]]
- [[工具调用框架]]