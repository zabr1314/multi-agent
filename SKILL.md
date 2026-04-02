---
name: multi-agent
description: "Design and orchestrate multi-agent systems for complex tasks. Use when (1) designing agent dispatch strategies (spawn/fork/direct), (2) building agent communication protocols, (3) implementing agent lifecycle management (create/execute/complete/cleanup), (4) orchestrating verification and adversarial agent patterns, (5) building team-based agent systems. Triggers on phrases like multi-agent, agent orchestration, spawn agent, fork agent, agent communication, team agents, agent lifecycle, verification agent."
---

# Multi-Agent

不是"一个 agent 做所有事"，而是"多个 agent 各司其职，主 agent 编排"。

## Core Architecture

```
主 Agent
  ├─ spawn（独立上下文，从零开始）
  ├─ fork（共享 prompt cache，后台执行）
  └─ direct（自己做，不派发）

子 Agent ←→ 通信协议 ←→ 子 Agent
```

## Step 1: Choose a Dispatch Strategy

| 策略 | 方式 | 特点 | 何时用 |
|:---|:---|:---|:---|
| spawn | 指定 agent 类型 | 独立上下文，不共享 cache | 需要完全独立的判断 |
| fork | 省略类型 | 共享父级 cache，便宜 | 研究类任务，噪音大的工作 |
| direct | 不派发 | 自己做 | 简单任务，不需要隔离 |

**Fork 的关键规则：**
- Don't peek — 不要在 fork 运行时读中间输出
- Don't race — 不要伪造 fork 的结果
- 研究用 fork，实现用 spawn

详见 [references/dispatch-strategies.md](references/dispatch-strategies.md)

## Step 2: Design the Agent Ecosystem

不同 agent 有不同角色定位：

| Agent | 能力 | 特点 |
|:---|:---|:---|
| 通用 Agent | 所有工具 | 通用执行 |
| 搜索 Agent | 只读搜索 | 快速，禁写入 |
| 规划 Agent | 只读规划 | 输出关键文件列表 |
| 验证 Agent | 对抗验证 | PASS/FAIL/PARTIAL |
| 文档 Agent | 文档查询 | 查官方文档 |

Agent 生命周期：创建 → 初始化 → 执行 → 清理（finally 块清理所有资源）

详见 [references/agent-ecosystem.md](references/agent-ecosystem.md)

## Step 3: Orchestrate Patterns

6 种可复用的编排模式：

| 模式 | 描述 | 适用场景 |
|:---|:---|:---|
| Worker Pool | 并行派发多个同类 agent | 批量处理 |
| Pipeline | A 的输出喂给 B | 多步骤流程 |
| Supervisor-Worker | 主 agent 分配+回收 | 复杂任务分解 |
| Adversarial | 实现 + 独立验证 | 质量保证 |
| Fork-Join | fork 多个并行 → 汇总结果 | 并行研究 |
| Resume Chain | 跨轮恢复 agent 上下文 | 长时间任务 |

验证 Agent 的对抗性编排：预判模型的自我欺骗（验证逃避、80% 魅惑），用结构化输出强制证据。

详见 [references/orchestration-patterns.md](references/orchestration-patterns.md)

## Key Rules

- **spawn 给判断，fork 给噪音** — 需要独立判断用 spawn，噪音大的工作用 fork
- **Don't peek, don't race** — fork 的两条铁律
- **Completion 是 push-based** — 不要轮询，等通知
- **前台 vs 后台** — 需要结果再继续用前台，独立工作用后台
- **Agent 间通信用专用工具** — 文本输出对其他 agent 不可见
- **验证必须独立** — 自我验证不可靠，必须另起 agent
