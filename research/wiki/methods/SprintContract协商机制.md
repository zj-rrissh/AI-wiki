---
id: "20260428200006"
type: method
tags:
  - "agent"
  - "contract"
  - "negotiation"
status: draft
aliases:
  - "迭代前对齐协议"
  - "Sprint合同"
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Harness design for long-running application development.md"
confidence: 0.9
---

## 方法名称

Sprint Contract 协商机制

## 解决的问题

Generator 和 Evaluator 对"完成"定义不一致，导致迭代效率低。通过协商形成明确合同，避免返工。

## 核心流程

```
1. Generator 提出要构建什么、如何验证成功
           ↓
2. Evaluator 审查确保构建正确的功能
           ↓
3. 两者迭代直到同意"sprint 合同"
           ↓
4. 合同是 bridging 用户 stories 和可测试实现的桥梁
```

## 合同内容

每份 Sprint Contract 应包含：

| 内容 | 说明 |
|------|------|
| **Feature** | 要构建的功能 |
| **Acceptance Criteria** | 如何验证成功（可测试） |
| **Edge Cases** | 需要处理的边界情况 |
| **Out of Scope** | 明确不包含的内容 |

## 协商原则

1. **Generator 主导**：Generator 提出方案，因为它是执行者
2. **Evaluator 审查**：Evaluator 确保方案可验证
3. **迭代对齐**：通过 1-2 轮迭代达成共识
4. **书面记录**：合同是文字协议，避免理解偏差

## 适用场景

- [[三Agent架构]] 中的迭代前对齐
- 任何需要明确"DOWN"定义的工作
- 弥补 spec 过于 high-level 的问题

## 优势

- **减少返工**：提前对齐期望
- **明确责任**：合同定义谁负责什么
- **可追溯**：争议时有据可查

## 相关方法

- [[三Agent架构]]
- [[Generator-Evaluator双组件架构]]