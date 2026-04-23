---
id: "20260423180010"
type: summary
tags: [Agent, 设计思维, 传统程序对比]
status: evergreen
aliases: []
created: 2026-04-23
updated: 2026-04-23
source: raw/notes/Agent_Loading/我理解的agentVS传统app.md
confidence: 0.75
---

# 我理解的agentVS传统app 编译摘要

## 核心结论

1. **Agent 与传统程序的核心差异是决策方式**
   - 传统程序：确定性规则，完全可控
   - Agent：概率性推理，黑盒决策

2. **Prompt Engineering 的本质是「引导而非控制」**
   - 约束代替精确指令
   - 提示代替流程规划

3. **Agent 开发的核心考验是提示工程，而非架构设计**

---

## 关键洞察

> "就像在教育一个有着庞大知识储量的新手。虽然他知识储备大，但是不知道怎么开始。"

---

## 质疑

1. **过度简化**：实际 Agent 系统需要结合规则引擎，不能完全依赖概率推理
2. **安全边界**：约束如何防止 Agent 产生有害行为？

---

## 关联概念

- [[Agent设计思维]]
- [[大模型基础认知]]
- [[工具调用框架]]