# Agent OS — AI 协作协议

> 多 Agent 协作运行时：任务自动分级 → 派发专职 Agent → 多维审查验收 → 整改或通过。
> 不是"调几个 Agent 干活"，而是**可恢复、可追踪、可并发、可演化的 AI 软件工程操作系统**。

## 触发条件

任何任务进入时自动执行分级，无需用户感知。

## 决策树

```
接到任务
  │
  ▼
单文件？不改逻辑？ ──是──→ L0 直行：直接改，不写 state
  │
  ▼否
单文件？内部逻辑？ ──是──→ L1 轻量：写 task + artifact，跳过 lock/build
  │
  ▼否
>5文件？跨模块？改接口？ ──是──→ L3 严格：完整流程，串行 review，全量 build
  │
  ▼否
L2 标准：并行 review，定向 build
```

## L0-L3 执行细则

### L0 — 直行

```
不改 state → 直接 Edit/Write → 口头告知用户
```

不写 task，不加 lock，不 review。前提是修改必须原子化、可一眼验证。

### L1 — 轻量

```
1. 写 task 记录到 agent-state.yaml（status: running）
2. 执行修改
3. 写 artifact 记录产出文件
4. 更新 task status → done，更新 session.updated
```

跳过 lock（单文件无冲突风险），跳过 build（内部逻辑不影响其他模块），单一 quality reviewer。

### L2 — 标准

```
1. 读取 agent-state.yaml 检查 locks 冲突
2. 写 task + 获取 lock（file 或 logical）
3. 评估依赖：有 depends_on 则等待
4. 派发子 Agent 并行执行（无依赖的各自并行）
5. 并行 review：quality + architecture
6. 不通过 → 退回整改（最多 2 次）→ 再审
7. 通过 → 写 artifact、释放 lock、记录 health
8. 更新 task status → done
```

### L3 — 严格

```
L2 全部流程 +
  - 串行 review（quality → architecture → security）
  - 全量 build 验证
  - 每次 review 不通过 → 1 次整改机会
  - 高风险失败 → 自动升级 human_review
  - 超 2 次失败 → 升级 architect_review
  - 完成后写 decisions 日志
```

## 并行策略

- 无依赖的调研/搜索类 → 所有 Agent 同时发出
- 有依赖链的 → 按 DAG 拓扑序串行
- 同文件锁冲突 → 后发任务自动排队

## 验收标准（四维）

| 维度 | 检查项 |
|------|--------|
| 完整性 | 所有要求文件已产出，无遗漏 |
| 正确性 | 逻辑无错误，API 调用正确，类型匹配 |
| 一致性 | 命名/风格/架构模式与现有代码一致 |
| 可用性 | 能 build/run，无需人工修补 |

## 升级路径

```
L2/L3 任务失败:
  第 1 次 → 退回原 Agent 整改
  第 2 次 → 升级 architect_review
  高风险失败 → 直接 human_review
  信心 < 0.5 → human_review
```

## 状态文件

所有状态读写操作的目标文件：`.claude/agent-state.yaml`

- 子 Agent 启动前传入 state context（只读）
- 子 Agent 完成后 orchestrator 写入更新
- 所有时间戳使用 ISO 8601

## 禁止事项

- 子 Agent 禁止直接写 state 文件
- L0 任务禁止走流程（大炮打蚊子）
- 禁止跳过 review 标为 done
- 禁止 lock 未释放就结束会话
