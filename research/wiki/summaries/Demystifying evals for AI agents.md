---
id: "20260428190002"
type: summary
tags:
  - "eval"
  - "agent"
  - "benchmark"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Demystifying evals for AI agents.md"
confidence: 0.9
---

## 核心结论

1. **Agent 评测是端到端系统测试**，包含 Agent、Harness、环境三者，任何组件的配置差异都会影响分数。Terminal-Bench 2.0 上资源配置差异（1x→uncapped）产生 6 个百分点的差距（p<0.01）。

2. **Grader 类型需合理组合**：代码级（快速/客观/可复现）、模型级（灵活/可扩展/处理开放式输出）、人类级（黄金标准/校准模型 grader）。评测设计核心是选对 grader 类型。

3. **Eval 饱和问题**：当 Agent 通过所有可解任务，分数趋近 100%，失去改进信号。SWE-bench Verified 从 30% 提升到 >80%，前沿模型已接近饱和。

## 支撑证据

- Terminal-Bench 2.0 上 6 种资源配置实验：1x 严格 enforce → uncapped，成功率单调递增，主要由 infra error 率下降驱动（5.8% → 0.5%）
- 3x 以下额外资源只修复基础设施可靠性问题；3x 以上开始真正帮助 Agent 解决之前无法解决的问题
- Opus 4.5 在 CORE-Bench 初始 42%，修正 grading bug 和 scaffold 后跳到 95%

## 关键方法

### 评测组件术语
- **Task/Problem**：单次测试，有定义输入和成功标准
- **Trial**：一次尝试
- **Grader**：评分逻辑，每个 task 可有多个 grader
- **Transcript/Trace**：完整记录（API 调用+返回+推理）
- **Harness**：运行基础设施；Agent Harness 是让模型像 Agent 一样行动的系统

### pass@k vs pass^k
- **pass@k**：k 次尝试中至少一次成功。k↑ 时 pass@k↑
- **pass^k**：k 次尝试全部成功。k↑ 时 pass^k↓
- 适用场景：工具类任务关心"至少一次成功"→ pass@k；用户Facing任务关心"每次都可靠"→ pass^k

### 评测构建路线图
1. **Step 0**：20-50 个简单任务即可起步，不要等"几百个任务"
2. **Step 1**：将手动检查/用户报告失败转化为测试用例
3. **Step 2**：任务要无歧义，两位领域专家独立判断应得相同结论
4. **Step 3**：平衡问题集——既测"应该发生"也测"不应该发生"
5. **Step 4**：隔离 trial——每次从干净环境开始，避免状态污染

## 质疑

- **资源配置作为混淆变量**：终端用户在公开 leaderboard 上看到的分数差异可能反映硬件差异而非模型能力差异
- **评分的主观性**：即使"客观"的代码测试也可能因 grading bug 产生 40%+ 的分数差异（Opus 4.5 on CORE-Bench 案例）
- **Eval 过度设计陷阱**：团队可能在构建 eval 上花费数周，比实际构建功能还多

## 跨领域迁移

- **静态 vs Agentic 评测**：静态评测（代码输出）不涉及运行时环境；Agentic 评测必须将资源、时限作为实验变量控制
- **从其他领域借鉴**：OS/网络基准测试早就知道资源规格会影响结果，这一认识在 Agent 评测领域才刚开始
- **多方法互补**：自动化评测 + 生产监控 + A/B 测试 + 用户反馈 + 人工转审阅形成完整的 Agent 质量视图
