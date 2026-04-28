---
id: "20260428190008"
type: summary
tags:
  - "agent"
  - "architecture"
  - "managed-agents"
status: evergreen
aliases: []
created: 2026-04-28
updated: 2026-04-28
source: "research/raw/articles/Scaling Managed Agents Decoupling the brain from the hands.md"
confidence: 0.9
---

## 核心结论

1. **解耦"大脑"(Claude + harness)与"手"(sandbox + tools)及"会话"(event log)是核心架构创新**：会话作为独立接口存储在 harness 外，使故障恢复和弹性扩展成为可能。

2. **解耦后性能大幅提升**：p50 TTFT 下降约 60%，p95 下降超 90%，因为容器按需 provisioning，而非每个 session 开始时等待容器初始化。

3. **虚拟化接口思想与操作系统同源**："程序即尚未想到的程序"问题，OS 通过虚拟化硬件（进程、文件）解决，Managed Agents 虚拟化 Agent 组件（会话、harness、沙箱）。

## 支撑证据

- 耦合设计的问题：容器成为"pet"，不可替代；调试只能通过 WebSocket event stream，无法定位问题来源
- 解耦后容器变为"cattle"：容器死亡时 harness 捕获为 tool-call error，Claude 决定重试，新容器可通过标准 recipe 重新初始化
- Git 凭证 bundled with resource 而非在沙箱内可达，防止提示注入攻击

## 关键方法

### 核心接口抽象
```python
execute(name, input) → string    # 沙箱执行工具
provision({resources})          # 容器初始化
wake(sessionId)                # Harness 重启
getSession(id)                  # 获取事件日志
emitEvent(id, event)           # 写入会话记录
getEvents()                    # 选择性读取事件片段
```

### 安全边界设计
- **Auth bundled with resource**：Git 访问 token 在沙箱初始化时克隆 repo 并 wire 到 local git remote，Agent 不处理 token 本身
- **MCP OAuth tokens 存于 vault**：Claude 通过 dedicated proxy 调用 MCP，proxy 从 vault 获取对应凭证，沙箱外的 harness 永远不接触凭证

### 上下文管理分离
- **会话**负责持久化上下文存储（可恢复）
- **Harness**负责任意上下文工程（压缩、裁剪、组织）
- 分离使两方可独立演进，各自优化

## 质疑

- **接口复杂性**：为支持灵活解耦，需要设计足够通用的接口，这增加了系统复杂度
- **调试难度转移**：当问题发生在接口层面（如事件流中的 packet drop）而非实现层面时，定位可能更难
- **模型假设**：架构假设 Claude 能够自主决定何时需要容器、何时不需要，这一假设可能随模型能力变化

## 跨领域迁移

- **微服务架构的类比**：每个组件可独立部署和扩展，通过接口通信
- **元框架设计**：系统对特定 harness 实现无观点，只对接口有观点，允许未来任意 harness 实现
- **Many brains, many hands**：允许一个 Agent 同时控制多个执行环境（多沙箱），或多个 Agent 共享同一个沙箱
