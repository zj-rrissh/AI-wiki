---
id: "20260428190001"
type: summary
tags:
  - "agent"
  - "long-running"
  - "harness"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Effective harnesses for long-running agents.md"
confidence: 0.9
---

## 核心结论

1. **Initializer Agent + Coding Agent 双组件架构**是解决长程 Agent 断续问题的关键：_initializer_ 在首 session 设置环境（feature list、progress file、init.sh），_coding agent_ 增量推进并留下结构化交接文档。

2. **Feature List 是防止 Agent 过早终止的核心机制**：以 JSON 格式维护完整功能清单，每项标记 `passes: false`，要求 Agent 逐一验证而非宣告胜利。

3. **增量提交 + 基础验证循环**使跨上下文连续工作成为可能：session 开始时读取 progress/git log 并运行基础测试，session 结束时 commit 并更新 progress。

## 支撑证据

- Claude Opus 4.5 在 "build a clone of claude.ai" 任务中，两种失败模式：① 试图 one-shot 全功能导致上下文溢出；② 早期宣告完成
- 使用 feature list 后，Agent 必须逐项验证而非一次性交付
- init.sh + Puppeteer MCP 端到端测试确保进入新 feature 前环境处于可用状态

## 关键方法

### Feature List 示例
```json
{
  "category": "functional",
  "description": "New chat button creates a fresh conversation",
  "steps": ["Navigate to main interface", "Click 'New Chat'", "Verify new conversation"],
  "passes": false
}
```

### Session 起始协议
1. `pwd` 确认目录
2. 读取 `claude-progress.txt` 和 git log
3. 启动 dev server 并用 Puppeteer 做基础功能测试
4. 从 feature list 选最高优先级未完成项

### 失败模式对照表
| 问题 | Initializer 方案 | Coding Agent 方案 |
|------|-----------------|------------------|
| 过早宣告胜利 | 建立完整 feature list | 按 list 逐项验证 |
| 环境有 bug/无文档 | 初始化 git + progress notes | session 结束时 commit + 更新 progress |
| 功能标记完成但未测试 | 建立 feature list | 自我验证全部功能后才改 passes |

## 质疑

- JSON feature list 维护成本较高，真实项目可能有 200+ 条目，更新负担重
- "clean state for merging" 的主观性：不同工程师对"可合并"标准不同
- 测试依赖 Puppeteer MCP，浏览器自动化盲区（如 native alert modal）会导致某些 bug 无法被发现

## 跨领域迁移

- **多 Agent 架构**：未来可能演化为 specialized agent（testing/QA/code cleanup），各司其职跨上下文协作
- **非 Web 开发泛化**：科学计算、金融建模等领域可借鉴相同的"环境初始化 → 增量推进 → 结构化交接"模式
- **压缩替代方案**：若压缩足够好，可能不需要 initializer/coding agent 的双组件划分
