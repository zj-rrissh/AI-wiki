---
id: "20260428200002"
type: method
tags:
  - "agent"
  - "multi-agent"
  - "architecture"
status: draft
aliases:
  - "Planner-Generator-Evaluator架构"
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Harness design for long-running application development.md"
confidence: 0.9
---

## 方法名称

三 Agent 架构（Planner + Generator + Evaluator）

## 解决的问题

单 Agent 无法同时完成：理解用户意图、生成高质量输出、验证输出正确性。三 Agent 各司其职，实现全栈应用生成。

## 架构组件

```
┌─────────────┐
│   Planner   │  将一句话 prompt 扩展为完整规格（16-feature spec）
└──────┬──────┘
       ↓
┌─────────────┐
│  Generator  │  按 sprint 工作方式构建功能
└──────┬──────┘
       ↓
┌─────────────┐
│  Evaluator  │  端到端验证功能正确性
└─────────────┘
```

### Planner 职责
- 将 high-level 需求扩展为具体规格
- 防止 generator under-scope（生成内容不足）
- 维护 16-feature spec 作为合同

### Generator 职责
- 按 sprint 工作，每次构建 1-2 个 feature
- 与 Evaluator 协商"Sprint Contract"
- 在迭代中学习 Evaluator 的标准

### Evaluator 职责
- Playwright MCP 实时交互验证
- 评分并给出改进建议
- 维护 grading criteria（设计质量、原创性、工艺、功能性）

## 关键方法：Sprint Contract

```
Generator 提出要构建什么、如何验证成功
        ↓
Evaluator 审查确保构建正确的功能
        ↓
两者迭代直到同意"sprint 合同"
        ↓
合同是 bridging 用户 stories 和可测试实现的桥梁
```

## 版本演进

| 版本 | 架构 | 特点 |
|------|------|------|
| V1 | Full 三组件 + Sprint 结构 | 功能完整但成本高（$200/次，6小时） |
| V2 | Planner + Generator + 单次 Evaluator | 简化后成本 $9，20分钟 |

## V2 简化原则

- 保留 Planner（仍需防止 under-scope）
- 保留 Evaluator（仍提供超出模型可靠 solo 能力的提升）
- 移除 Sprint 结构（Opus 4.6 可原生处理增量工作）
- 单次 Pass at End 而非每 sprint grading

## 适用场景

- 全栈应用生成
- 复杂前端项目
- 需要多轮迭代的创意任务

## 相关方法

- [[Generator-Evaluator双组件架构]] - 基础版本
- [[前端设计GradingCriteria]] - Evaluator 的评分标准