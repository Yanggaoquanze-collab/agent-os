# Agent OS — AI 协作协议

大多数 Multi-Agent 系统的失败不是因为 Agent 不够聪明，而是因为没有**单一事实源**：谁在改什么、哪些任务完成了、架构决策为什么这样做、当前项目是否健康。

Agent OS 定义了一个**协议 + 状态 + 调度**的三层架构，让多 Agent 协作变成可恢复、可追踪、可并发、可演化的软件工程系统。

```
协议层 (SKILL.md)    — 决策树、执行细则、验收标准、升级路径
状态层 (state.yaml)  — 11 模块 Schema，所有 Agent 的共享世界模型
调度层 (Orchestrator) — 自动分级、DAG 推导、并发控制、文件锁
```

---

## 快速开始

1. 将 `agent-state.yaml` 复制到项目的 `.claude/agent-state.yaml`
2. 将 `SKILL.md` 放入 `.claude/skills/agent-os/SKILL.md`
3. 重启 Claude Code 会话，skill 自动注册

之后任何任务进入，Orchestrator 会自动按协议分级执行。

---

## 任务分级

| 级别 | 触发条件 | 流程 | 预估耗时 |
|------|---------|------|----------|
| L0 直行 | 单文件、不改逻辑 | 直接改，不写 state | < 10s |
| L1 轻量 | 单文件、改逻辑 | task + artifact | < 2min |
| L2 标准 | 多文件、中等风险 | 完整流程 + 并行 review | 5-10min |
| L3 严格 | 跨模块、高风险 | 完整流程 + 串行 review + 全量 build | 10-30min |

---

## 状态 Schema（11 模块）

```
session → project → constraints → tasks → locks → health
    → decisions → artifacts → task_classification → risk_policy → risks
```

每字段标注：必填/可选/自动生成 + 读写权限（Orchestrator / Executor / Validator / Architect / Human）。

---

## 文件结构

```
.claude/
├── agent-state.yaml          ← 状态协议 (v1)
└── skills/
    └── agent-os/
        └── SKILL.md           ← 执行规则
```

---

## 许可

MIT
