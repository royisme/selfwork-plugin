# Operational Workflow — CEO 编排模式

## 概览

Main agent（CEO）只做调度决策，不动手实施。工作通过结构化 JSON 合约在 agent 间流转。

## Phase A: Bootstrap / Resume

1. 检查 `.claude/dispatch/active`
2. 存在 → 读取 `state.json`
   - 根据 `status` 字段恢复到对应 Phase
   - `analyzing` → 检查 analysis-report.json 是否已完成
   - `specifying` → 检查 plan.json 和 spec_status
   - `executing` → 检查各 task 状态
3. 不存在 → Phase B

## Phase B: Planning（status=planning）

1. 接收用户需求
2. 识别 `input_source`:
   - `interactive`: 对话式需求收集
   - `external_plan`: 用户提供计划文件 → 记录 `input_refs`
   - `mixed`: 部分已有、部分需澄清
3. 创建 run 目录和 state.json
4. 设置 active 指针
5. 流转到 Phase C

## Phase C: Analysis（status=analyzing）

1. CEO 派遣 Analyst agent（只读探索）
2. Analyst 输出 `artifacts/analysis-report.json`
3. CEO 审阅报告：
   - 评估 `requirement_confidence`
   - `low` → 与用户澄清，可重新派遣
   - `medium/high` → 继续
4. 流转到 Phase D

## Phase D: Specification（status=specifying）

1. CEO 派遣 Architect agent
2. Architect 输出：
   - 规格文档 → `devDocs/spec/selfwork/<topic>.md`
   - 实施计划 → `artifacts/plan.json`
3. CEO 向用户展示规格摘要，请求确认
4. 用户确认 → `spec_status=approved`
5. **门禁**：必须 `spec_status=approved` 才能流转
6. 将 plan.json 中的 tasks 同步到 state.json
7. 流转到 Phase E

## Phase E: Execution Loop（status=executing）

对 `execution_order` 中的每个 task：

### E1: Dispatch Developer

1. 为 task 生成 subtask spec（参考 subtask-template.md）
2. 按 complexity 选择 agent:
   - `small` → haiku-dev
   - `medium/hard` → sonnet-dev
3. 派遣 agent，prompt 包含 subtask spec 全文
4. 更新 task status=dispatched

### E2: Developer 完成

1. Agent 返回 → 读取 `artifacts/dev-report-<tid>.json`
2. 更新 task status=agent_done

### E3: Dispatch Reviewer

1. 派遣 code-reviewer agent
2. Reviewer 输出 `artifacts/review-report-<tid>.json`
3. 更新 task status=reviewing

### E4: CEO 审阅 Review

1. 读取 review-report verdict:
   - `approved` → task status=completed
   - `changes_requested` → 检查 retry_count
     - 未超限 → retry_count++, 重新 dispatch（附加 issues context）
     - 超限 → task status=failed
   - `blocked` → task status=failed, 考虑回退
2. 检查是否有新的 dispatchable tasks（依赖已满足）
3. 继续循环

## Phase F: Completion（status=completed）

1. 所有 task completed
2. 清除 active 指针
3. CEO 向用户汇总：
   - 变更文件总览
   - 测试状态
   - 质量报告
   - Spec 文档位置

## Phase G: Blocked Handling（status=blocked）

1. 存在 failed task 且无可 dispatch 的任务
2. 向用户报告：
   - 失败原因
   - Review issues
   - 建议（拆分需求/手动修复/重新规格化）

## 安全约束

1. **run-id 校验**：`^[A-Za-z0-9._-]+$`
2. **路径锚定**：所有路径从 repo root 解析
3. **原子写入**：state.json 通过 temp + rename 写入
4. **Lock 协调**：写 state.json 前检查 lock 文件
5. **旧 run 阻断**：发现旧 active run → 提示用户处理

## Stop Hook 集成

Hook 在每次 agent 尝试停止时校验：
- state.json schema 合规性
- artifact 存在性（按状态流转要求）
- TDD 门禁（critical + tdd 必须有 test_command）
- 阻断不合规的停止请求
