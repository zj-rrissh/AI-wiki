---
id: "20260423180002"
type: concept
tags: [LangChain, LCEL, 链式调用, 结构化输出]
status: draft
aliases: [LangChain表达式语言, LangChain Expression Language]
created: 2026-04-23
updated: 2026-04-23
source: raw/notes/LangChain学习笔记.md, raw/notes/Agent_Loading/阶段三.md
confidence: 0.85
---

# LCEL

LangChain Expression Language，LangChain 框架中的声明式链式调用语法。

## 定义

通过 `|` 操作符将组件串联，上一个组件的输出自动作为下一个组件的输入。

```python
chain = prompt | model | parser
```

## 核心特性

### Runnable 协议

所有组件遵循统一接口：

| 方法 | 用途 |
|------|------|
| `invoke()` | 单输入处理 |
| `batch()` | 并行多输入 |
| `stream()` | 流式响应 |
| `ainvoke/abatch/astream` | 异步版本 |

### 结构化输出

通过 PydanticOutputParser 实现类型安全输出：

```python
class IntentEntity(BaseModel):
    intent: str
    entities: dict

parser = PydanticOutputParser(pydantic_object=IntentEntity)
chain = prompt | model | parser
result = chain.invoke({"input": "..."})
```

## 应用场景

- **结构化输出链**：从用户输入提取意图+实体
- **RAG 检索链**：检索 → 增强 → 生成
- **Agent 工具链**：推理 → 工具选择 → 执行

## 关键数据点

- **最小块**：`prompt | model | parser` 三件套
- **批量处理**：`.batch()` 支持并行
- **流式**：`|` 可配合 `.stream()` 实现流式输出

## 前提与局限性

- **依赖 LangChain**：跨框架迁移成本高
- **调试复杂**：长链路的错误定位需要 LangSmith
- **版本注意**：旧版 `create_chain` vs 新版 `create_agent`

## 关联概念

- [[RAG]]：RAG 检索链基于 LCEL 实现
- [[工具调用框架]]：Agent 工具选择链
- [[结构化输出]]：LCEL 的典型应用

---

**来源**：`LangChain学习笔记.md`、`阶段三.md`