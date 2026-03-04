---
name: selfwork
description: CEO 编排模式：自主多步骤开发执行。Main agent 只做调度决策，专项 agent 负责分析、架构、实施、质量审查。
user_invocable: false
---

# selfwork — CEO 编排技能

## 触发映射

（保留原有命令触发映射：/selfwork, /selfwork:status, /selfwork:queue, /selfwork:clean）

## 核心原则

- **Main = CEO**：读状态 → 做决策 → 分派 → 验收 → 交付
- **Agent = 员工**：各有明确角色、输入合约、输出合约
- **JSON = 通信协议**：agent 间唯一可靠的结构化通信方式
- **Hook = 执法者**：校验状态合规性，阻断违规流转
- **CEO 绝不动手**：不写代码、不写 spec、不跑测试

## 角色定义

| 角色 | Agent | 职责 | 输出 artifact |
|------|-------|------|---------------|
| Analyst | Agent(subagent_type=Explore) | 探索代码库、分析可行性 | analysis-report.json |
| Architect | Agent(subagent_type=Plan) | 产出规格文档和实施计划 | spec 文件 + plan.json |
| Developer | Agent(subagent_type=haiku-dev/sonnet-dev) | 按规格写代码 | 代码变更 + dev-report.json |
| Reviewer | Agent(subagent_type=code-reviewer) | 代码审查、测试、质量报告 | review-report.json |

## 文件与目录

- 调度根目录：`.claude/dispatch/`
- Active run 指针：`.claude/dispatch/active`
- Run 目录：`.claude/dispatch/runs/<run-id>/`
  - `state.json` — 主状态文件（schema: `references/schemas/run-state.schema.json`）
  - `artifacts/` — 各 agent 输出的 JSON 合约
    - `analysis-report.json`
    - `plan.json`
    - `dev-report-<task-id>.json`
    - `review-report-<task-id>.json`
- Task specs：`.claude/task-specs/<run-id>/subtasks/tN.md`
- 规格文档：`devDocs/spec/selfwork/<topic>.md`

## 状态模型

### Run 状态流转

```
planning → analyzing → specifying → executing → completed
                                                    ↓
                                                 blocked
```

- `planning`: 初始化 run，识别 input_source
- `analyzing`: Analyst agent 工作中
- `specifying`: Architect agent 工作中，spec_status 管控
- `executing`: Developer + Reviewer 循环
- `completed`: 全部任务完成
- `blocked`: 不可恢复的阻塞

### spec_status 门禁

- `draft`: Architect 产出中
- `approved`: 用户确认，可进入 executing
- `obsolete`: 需重新规格化

### Task 状态

`pending → dispatched → agent_done → reviewing → completed | failed`

## CEO 编排流程

### Phase 0: Bootstrap

1. 检查 `.claude/dispatch/active` 是否存在
2. 存在 → 读取 state.json，从断点恢复
3. 不存在 → 进入 Phase 1

### Phase 1: 接收需求（planning）

1. 识别 input_source：
   - `interactive`: 用户口述需求，CEO 记录
   - `external_plan`: 用户提供已有计划文件
   - `mixed`: 部分已有、部分需澄清
2. 创建 run，初始化 state.json（status=planning）
3. 设置 `.claude/dispatch/active` 指向当前 run

### Phase 2: 派遣 Analyst（analyzing）

1. 更新 state.json: status=analyzing
2. 派遣 Agent(subagent_type=Explore)，提示词包含：
   - 需求描述
   - 代码库范围
   - 要求输出 analysis-report.json 到 artifacts/
3. 等待 agent 完成
4. 读取 analysis-report.json
5. 评估 requirement_confidence：
   - `high`: 继续
   - `medium`: 向用户确认关键假设后继续
   - `low`: 与用户澄清后重新派遣 Analyst

### Phase 3: 派遣 Architect（specifying）

1. 更新 state.json: status=specifying, spec_status=draft
2. 派遣 Agent(subagent_type=Plan)，提示词包含：
   - 分析报告内容
   - 需求描述
   - 要求输出 spec 文件 + plan.json
3. 等待 agent 完成
4. 读取 plan.json，向用户展示规格摘要
5. 用户确认 → spec_status=approved
6. 用户要求修改 → 重新派遣 Architect
7. **门禁**：spec_status 必须为 approved 才能进入 executing

### Phase 4: 分派 Developer（executing）

1. 更新 state.json: status=executing
2. 从 plan.json 读取 execution_order
3. 按顺序（或可并行的任务并行）分派：
   - 为每个 task 生成 subtask spec（.claude/task-specs/）
   - 派遣 Agent(subagent_type=haiku-dev 或 sonnet-dev)
   - Agent 完成后读取 dev-report.json
4. 每个 dev 完成后，立即派遣 Reviewer

### Phase 5: 派遣 Reviewer

1. 对每个完成的 task 派遣 Agent(subagent_type=code-reviewer)
2. Reviewer 输出 review-report.json
3. CEO 读取 verdict：
   - `approved` → 标记 task completed
   - `changes_requested` → 重新派遣 Developer（检查 retry_count < max_retries）
   - `blocked` → 标记 task failed，报告用户

### Phase 6: 完成

1. 所有 task 完成 → status=completed
2. 清除 `.claude/dispatch/active`
3. 向用户汇总交付：变更文件列表、测试状态、质量报告

## Agent 分派模板

### Analyst 分派

```
使用 Agent tool:
- subagent_type: Explore (或 general-purpose)
- prompt: 包含需求描述 + 输出路径 + schema 引用
- 关键：要求将 analysis-report.json 写入 artifacts/
```

### Architect 分派

```
使用 Agent tool:
- subagent_type: Plan (或 general-purpose)
- prompt: 包含分析报告 + 需求 + 输出路径
- 关键：要求同时输出 spec 文件和 plan.json
```

### Developer 分派

```
使用 Agent tool:
- subagent_type: haiku-dev (small/medium) 或 sonnet-dev (hard)
- prompt: 包含 subtask spec 内容 + dev-report 输出路径
- 关键：要求完成后写入 dev-report.json
```

### Reviewer 分派

```
使用 Agent tool:
- subagent_type: code-reviewer
- prompt: 包含 dev-report 内容 + spec 引用 + review-report 输出路径
- 关键：要求运行质量门禁并写入 review-report.json
```

## 决策规则

### 需求置信度判定

| 信号 | confidence |
|------|-----------|
| 用户给出明确功能描述 + 验收标准 | high |
| 用户描述模糊但分析报告可补全 | medium |
| 分析报告也无法确定范围 | low |

### Agent 选择

| 任务复杂度 | Agent |
|-----------|-------|
| small | haiku-dev |
| medium | sonnet-dev |
| hard | sonnet-dev |

### 重试策略

- 最大重试次数：max_retries（默认 2）
- 重试时附加 review issues 作为 context
- 超过重试次数 → status=blocked，报告用户

## 安全约束

1. run-id 必须匹配 `^[A-Za-z0-9._-]+$`
2. 路径解析必须锚定 repo root
3. state.json 写入使用原子操作（临时文件 + 重命名）
4. Hook 校验每次状态流转
