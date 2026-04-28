---
id: "20260428200001"
type: method
tags:
  - "agent"
  - "architecture"
  - "generator-evaluator"
status: draft
aliases:
  - "生成器-评估器架构"
  - "双组件迭代架构"
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Harness design for long-running application development.md"
confidence: 0.9
---

## 方法名称

Generator-Evaluator 双组件迭代架构

## 解决的问题

单次生成无法保证输出质量，特别是对于复杂的前端应用或需要多轮迭代的任务。通过生成-评估的迭代循环，可以显著提升输出质量。

## 核心机制

```
Generator → 输出 → Evaluator → 评分 + 反馈 → Generator（下一轮）
   ↑                                              |
   └──────────────────────────────────────────────┘
                  5-15 轮迭代
```

- **Generator**：负责生成输出（代码、设计、解决方案等）
- **Evaluator**：通过实时交互评分并给出具体反馈
- **迭代次数**：通常 5-15 轮后输出质量趋于稳定

## 关键参数

| 参数 | 说明 | 典型值 |
|------|------|--------|
| max_iterations | 最大迭代次数 | 15 |
| quality_threshold | 通过阈值（可选） | - |
| early_stop_streak | 连续通过次数阈值 | 3 |

## 适用场景

- 前端应用生成
- 代码审查与优化
- 创意生成任务
- 复杂任务的迭代改进

## 实施要点

1. **Evaluator 必须可执行**：使用 Playwright MCP 等工具进行真实交互验证
2. **反馈要具体**：不能只说"不好"，要指出哪里不好、如何改
3. **成本控制**：每轮都有成本，需权衡质量提升和成本增加

## 来源依据

博物馆网站案例中，Generator 在第 10 轮自发重构为 3D 空间体验，展示了迭代过程中的创意跳跃。

## 相关方法

- [[三Agent架构]] - 扩展版本，增加 Planner 组件
- [[SprintContract协商机制]] - 迭代前的对齐协议