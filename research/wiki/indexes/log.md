# 操作日志

## 2026-04-23

### 编译任务

**源目录**：`raw/notes/`

**处理文件**：
- `Agent_Loading/导航.md` → `summaries/导航_编译.md`
- `Agent_Loading/阶段三.md` → `summaries/阶段三_编译.md` + `concepts/RAG.md`
- `Agent_Loading/阶段二.md` → `summaries/阶段二_编译.md` + `concepts/工具调用框架.md`
- `Agent_Loading/大模型基础认知.md` → `summaries/大模型基础认知_编译.md` + `concepts/大模型基础认知.md`
- `Agent_Loading/我理解的agentVS传统app.md` → `summaries/agentVS传统app_编译.md` + `concepts/Agent设计思维.md`
- `LangChain学习笔记.md` → `summaries/LangChain学习笔记_编译.md` + `concepts/LCEL.md`

**跳过**：
- `LlamaIndex框架学习.md`（空文件）

### 编译结果

| 类型 | 数量 |
|------|------|
| summaries | 6 |
| concepts | 5 |
| 总产出 | 11 |

### 概念条目

1. [[LCEL]] - LangChain表达式语言
2. [[RAG]] - 检索增强生成
3. [[工具调用框架]] - Function Calling设计模式
4. [[Agent设计思维]] - Agent vs 传统程序对比
5. [[大模型基础认知]] - Transformer架构基础

### 下一步

- 初始化 wiki/methods/ 目录（如有方法论内容）
- 触发 git 提交（待用户确认）

---

## 2026-04-24

### 日记创建

- 创建 `2026-04-24_待办日记.md`，总结学习进展并规划明日任务

### 学习进度

- 当前阶段：阶段三完成 → 阶段四（Agent核心范式与LangGraph）
- 核心概念：5个（LCEL、RAG、工具调用框架、Agent设计思维、大模型基础认知）

### 明日计划

- 知识学习：阶段四 - LangGraph、Agent循环设计、多Agent协作
- 产出目标：编译阶段四笔记，创建 LangGraph 概念条目

---

## 2026-04-28

### 编译任务

**源目录**：`raw/articles/`

**处理文件**（12篇，均为 L1 快速摘要）：

| 文章 | 来源 | 核心概念 |
|------|------|----------|
| Effective harnesses for long-running agents | Anthropic | Initializer+Coding Agent 双组件、Feature List |
| Demystifying evals for AI agents | Anthropic | Grader 类型、pass@k vs pass^k、Eval 饱和 |
| Effective context engineering for AI agents | Anthropic | 上下文衰减、三大压缩技术对比 |
| Contextual Retrieval in AI Systems | Anthropic | 上下文嵌入+BM25、重排序、49%/67% 提升 |
| Equipping agents for the real world with Agent Skills | Anthropic | SKILL.md 格式、渐进式披露 |
| Making Claude Code more secure and autonomous with sandboxing | Anthropic | 文件系统+网络双重隔离、84% 提示减少 |
| Quantifying infrastructure noise in agentic coding evals | Anthropic | 资源配置混淆变量、6个百分点差异 |
| Scaling Managed Agents | Anthropic | Brain-Hand-Session 解耦、cattle vs pet |
| The think tool | Anthropic | τ-Bench pass^k、54% 相对提升 |
| Indexing | LlamaIndex | VectorStoreIndex、Top-K 检索 |
| Designing AI resistant technical evaluations | Anthropic | v1→v3 版本迭代、Zachtronics 设计 |
| Harness design for long-running application development | Anthropic | Generator-Evaluator 架构、Sprint Contract |

### 编译结果

| 类型 | 数量 |
|------|------|
| summaries | 12 |
| concepts | 0（均采用 L1，未创建新概念） |
| 总产出 | 12 |

### Git 提交

- `89fd919` - comp: add 12 summaries from raw articles