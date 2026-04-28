---
id: "20260428200005"
type: method
tags:
  - "agent"
  - "frontend-design"
  - "evaluation"
status: draft
aliases:
  - "前端设计评分标准"
  - "前端评估四维度"
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Harness design for long-running application development.md"
confidence: 0.9
---

## 方法名称

前端设计 Grading Criteria（四维度评分法）

## 解决的问题

如何客观评估 AI 生成的前端设计质量。定义明确的评分维度，使Evaluator能给出可执行的反馈。

## 四维度框架

| 维度 | 说明 | 评估点 |
|------|------|--------|
| **Design Quality** | 颜色/字体/布局/图像是否形成统一 mood 和 identity | 视觉一致性、风格连贯性 |
| **Originality** | 是否有自定义决策，还是模板/库默认值/"AI slop"模式 | 创意决策、差异化 |
| **Craft** | 技术执行质量 | 排版层级、间距一致性、色彩和谐、对比度 |
| **Functionality** | 可用性 | 用户能理解界面、找到主要操作、完成目标 |

## 评估维度详解

### Design Quality（设计质量）

**高分标准**：
- 所有视觉元素（颜色、字体、布局、图像）形成统一 mood
- 风格一致，无突兀的元素
- 有明确的设计 identity

**低分特征**：
- 颜色/字体不统一
- 元素之间缺乏关联
- "模板感"明显

### Originality（原创性）

**高分标准**：
- 有自定义决策，不是简单调用库/模板
- 能看出设计意图和思考
- 有差异化特点

**低分特征**：
- 大量使用默认样式
- "AI slop"模式（通用但无特色）
- 无个人决策痕迹

### Craft（工艺）

**高分标准**：
- 排版层级清晰
- 间距一致、有节奏感
- 色彩和谐、对比度适当

**低分特征**：
- 排版混乱
- 间距不规则
- 对比度不足或过度

### Functionality（功能性）

**高分标准**：
- 用户能理解界面
- 能找到主要操作
- 能完成任务目标

**低分特征**：
- 界面难以理解
- 找不到关键功能
- 操作无法完成目标

## 实施方式

Evaluator 使用这四个维度对每次生成进行评分，给出 1-5 分的具体分数和文字反馈。

## 与其他方法的关联

- 用于 [[三Agent架构]] 中 Evaluator 的评分标准
- 是 [[Generator-Evaluator双组件架构]] 中反馈的具体来源

## 来源案例

博物馆网站案例中，Agent 在第 10 轮迭代时基于这些标准给出了超预期的创意（3D 空间体验）。

## 相关方法

- [[三Agent架构]]
- [[Generator-Evaluator双组件架构]]