---
id: "20260428190007"
type: summary
tags:
  - "eval"
  - "infrastructure"
  - "benchmark"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Quantifying infrastructure noise in agentic coding evals.md"
confidence: 0.85
---

## 核心结论

1. **Agentic 评测的资源配置是隐藏混淆变量**：Terminal-Bench 2.0 上，1x→uncapped 资源配置产生 6 个百分点的成功差异（p<0.01），这种差异可超过 leaderboard 上的真实模型能力差距。

2. **3x 资源以下是基础设施可靠性修复，3x 以上开始影响模型真实能力**：低于 3x 时分数波动在噪声范围内（p=0.40），高于 3x 后分数开始明显上升。

3. **SWE-bench 上资源影响较小（约 1.54 个百分点）**：因为其任务资源密集度低于 Terminal-Bench，说明资源配置的量化影响与任务类型强相关。

## 支撑证据

- Terminal-Bench 2.0 Kubernetes 实现：per-task resource spec 被同时用作 floor 和 ceiling，零缓冲导致瞬态内存波动可 OOM kill 容器
- 6 种资源配置实验：严格 enforce→uncapped，infra error 率从 5.8% 单调下降到 0.5%（1x→uncapped）
- `bn-fit-modify` 任务：部分模型首步安装完整 Python 数据科学栈，在严格限制下 OOM，在宽松限制下成功

## 关键方法

### 资源配置最佳实践
- **指定 guaranteed allocation + hard kill threshold 两个参数**，而非单一 pinned value
- 两者之间留 3x 缓冲：infra error 率降低 2/3（5.8%→2.1%，p<0.001），同时分数变化在噪声范围内
- 精确乘数因基准测试和任务分布而异，需经验校准但原则通用

### 时间限定下的行为差异
- **严格限制奖励高效策略**（lean, efficient code）
- **宽松限制奖励资源密集策略**（brute-force with heavyweight tools）
- 两者都是合法测试维度，但不说明资源配置的差异会导致结果难以解释

## 质疑

- **报告的 leaderboard 精度存疑**：3 个百分点以内的差异应持怀疑态度，尤其在没有记录资源配置时
- **时间维度混淆**：API 延迟随流量变化，pass rate 可能在一天内波动，这种噪音难以控制
- **外部评测者的困境**：无法知道评测时用的硬件是否与报告的完全一致

## 跨领域迁移

- **传统系统基准测试早已知晓资源规格影响结果**，这一认识在 Agent 评测领域才被正式量化
- **未来建议**：公开基准测试应在多个时间段和日期运行以平均化噪音；结果应附带资源配置标准差
- **决策者警示**：当模型选择决策基于 leaderboard 差异 <3% 时，这些差异可能只反映硬件差异
