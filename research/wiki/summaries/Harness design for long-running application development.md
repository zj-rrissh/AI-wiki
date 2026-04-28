---
id: "20260428190012"
type: summary
tags:
  - "agent"
  - "multi-agent"
  - "generator-evaluator"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Harness design for long-running application development.md"
confidence: 0.9
---

## 核心结论

1. **Generator-Evaluator（类似 GAN 的双组件）是显著提升 Agent 输出质量的关键**：Generator 生成，Evaluator 通过 Playwright MCP 实时交互评分并给反馈，5-15 轮迭代后输出质量明显优于单次生成。

2. **三 Agent 架构（Planner + Generator + Evaluator）实现全栈应用生成**：Planner 将一句话 prompt 扩展为 16-feature spec，Generator 按 sprint 工作，Evaluator 端到端验证，Solo harness 20分钟 $9 vs Full harness 6小时 $200。

3. **Harness 设计假设会随模型能力演进而过时**：Opus 4.5 的"context anxiety"在 Opus 4.6 中消失，说明需要持续审视和简化 harness，移除不再需要的组件。

## 支撑证据

- 博物馆网站案例：10th iteration 时 Agent 完全重构为 3D 空间体验（CSS perspective、游离挂画、门口导航），展示自发创意跳跃
- 复古游戏制作器：Solo run 核心功能（游戏运行）broken，Full harness 功能完整可用
- DAW 测试：QA agent 发现"display-only"问题（clips 不能拖动、无 instrument UI、无图形效果编辑器）

## 关键方法

### 前端设计 Grading Criteria
- **Design Quality**：颜色/字体/布局/图像是否形成统一 mood 和 identity
- **Originality**：是否有自定义决策，还是模板/库默认值/"AI slop"模式
- **Craft**：技术执行（排版层级、间距一致性、色彩和谐、对比度）
- **Functionality**：可用性（用户能理解界面、找到主要操作、完成）

### Sprint Contract 协商机制
- Generator 提出要构建什么、如何验证成功
- Evaluator 审查确保构建正确的功能
- 两者迭代直到同意"sprint 合同"
- 合同是bridging 用户 stories 和可测试实现的桥梁

### V2 简化后的架构
- 保留 Planner（防止 generator under-scope）
- 保留 Evaluator（仍给超出模型可靠solo能力的任务提供真实提升）
- 移除 Sprint 结构（Opus 4.6 可原生处理增量工作）
- 单次 Pass at End 而非每 sprint grading

## 质疑

- **成本权衡**：Full harness 是 Solo 的 22x 成本（$200 vs $9），需要任务复杂度决定是否值得
- **Evaluator 调优难度**：初始时 Evaluator 倾向于宽容，需要多轮迭代才能对齐判断
- **主观性边界**：即使有 grading criteria，"设计质量"的判断仍存在主观性

## 跨领域迁移

- **GAN 思想的应用**：Generator-Discriminator 架构可迁移到代码生成（代码生成 vs 代码审查）、音乐生成（旋律生成 vs 乐理评估）等领域
- **Sprint Contract 模式**：可用于任何需要明确"DOWN"定义的工作，弥补 spec 过于 high-level 的问题
- **Harness 持续简化原则**：每次新模型发布时重新审视 harness，移除不再 load-bearing 的组件，添加新组件以扩大能力边界
