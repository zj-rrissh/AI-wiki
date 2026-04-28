---
id: "20260428190005"
type: summary
tags:
  - "skill"
  - "agent"
  - "plugin"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Equipping agents for the real world with Agent Skills.md"
confidence: 0.9
---

## 核心结论

1. **Skill 是 Agent 的可组合能力单元**：一个包含 `SKILL.md` 文件的目录，通过 YAML Frontmatter 定义 `name` 和 `description`，运行时加载到 Agent 系统提示。

2. **渐进式披露（Progressive Disclosure）是核心设计原则**：Agent 只在需要时加载技能相关信息（通过目录结构和引用机制），使上下文信息量实际无限。

3. **技能可用于 Claude.ai、Claude Code、Claude Agent SDK、Claude Developer Platform**，并支持代码执行（预写脚本）和文档参考两种模式。

## 支撑证据

- PDF 技能示例：核心 `SKILL.md` + `forms.md` + `reference.md`，Claude 根据任务动态选择加载哪些文件
- Claude 在 10th iteration 时完全重构博物馆网站为 3D 空间体验，展示自发创意跳跃
- 安全考虑：Skill 赋予 Agent 新能力，但恶意 Skill 可能引入漏洞，需审核

## 关键方法

### SKILL.md 结构
```yaml
---
name: "pdf"
description: "Enable Claude to read, edit and fill PDF forms"
---
# Skill content with instructions, references to bundled files
```

### 技能发展循环
1. **先 eval**：观察 Agent 在哪些方面遇到困难或需要更多上下文
2. **结构设计**：当 SKILL.md 过大时拆分为多个文件并引用；互斥/少同用的内容保持路径分离
3. **站在 Claude 视角迭代**：观察 Claude 实际如何使用技能，注意意外轨迹和对特定情境的过度依赖
4. **与 Claude 迭代**：让 Claude 记录成功方法和常见错误，转化为可复用技能代码

### 技能bundling示例
- PDF Skill → `SKILL.md` + `forms.md`（表单填写说明）+ `reference.md`（其他参考）
- 表单填写说明只在填写表单时读取，核心内容保持简洁

## 质疑

- **Skill 作者需要有经验的工程师**：需要理解 Claude 如何使用技能、边界在哪、哪些是 Claude 的已知能力
- **安全审核的复杂性**：纯代码审核可能无法发现行为层面的问题，需要更深入的测试
- **技能共享生态的维护负担**：技能作者需要持续更新以跟踪 Claude 能力演进

## 跨领域迁移

- **Skill 作为组织工具**：Skill 可用于团队共享工作流程和上下文给 Claude
- **与 MCP 的互补**：MCP 提供外部工具协议，Skill 提供 Agent 行为模式编码，未来可能深度集成
- **自进化技能**：未来 Agent 可能自己创建、编辑和评估技能，将行为模式编码为可重用能力
