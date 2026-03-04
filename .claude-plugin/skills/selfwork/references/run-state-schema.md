# Run State Schema

## 权威定义

JSON Schema 文件：`references/schemas/run-state.schema.json`

本文档为人类可读的补充说明。冲突时以 JSON Schema 为准。

## 状态值

### Run Status

| 值 | 说明 | 前置条件 |
|----|------|---------|
| `planning` | 初始化，收集需求 | 新 run 创建 |
| `analyzing` | Analyst agent 探索代码库 | planning 完成 |
| `specifying` | Architect agent 产出规格 | 分析报告就绪 |
| `executing` | Developer + Reviewer 循环 | spec_status=approved |
| `completed` | 全部任务完成 | 所有 task completed |
| `blocked` | 不可恢复阻塞 | 超过重试/架构问题 |

### spec_status

| 值 | 说明 |
|----|------|
| `draft` | Architect 产出中/待审 |
| `approved` | 用户确认，可执行 |
| `obsolete` | 需重新规格化 |

### Task Status

| 值 | 说明 |
|----|------|
| `pending` | 等待依赖或分派 |
| `dispatched` | 已分派给 agent |
| `agent_done` | Agent 完成，待 review |
| `reviewing` | Reviewer 审查中 |
| `completed` | 审查通过 |
| `failed` | 失败（超过重试或 blocked） |

## Artifacts 目录

```
runs/<run-id>/
├── state.json
└── artifacts/
    ├── analysis-report.json   # Analyst 输出
    ├── plan.json              # Architect 输出
    ├── dev-report-<tid>.json  # Developer 输出
    └── review-report-<tid>.json # Reviewer 输出
```

每个 artifact 有对应的 JSON Schema：`references/schemas/<name>.schema.json`

## 状态流转门禁

| 流转 | 前置条件 |
|------|---------|
| planning → analyzing | run 已创建 |
| analyzing → specifying | analysis-report.json 存在且合规 |
| specifying → executing | plan.json 存在 + spec_status=approved |
| executing → completed | 所有 task status=completed |
| any → blocked | 不可恢复错误 |

## 兼容性

- 旧 run（无 analyzing/specifying 状态）仍受支持
- 缺失字段使用 schema 默认值填充
- `agent` 和 `agent_type` 均识别
