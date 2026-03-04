---
description: 查看 selfwork 当前执行状态。展示当前阶段、各角色 agent 状态、artifact 状态。
---

使用 `selfwork` skill 执行状态查询流程并返回 CEO  orchestration 状态摘要。

输出以下信息（使用 Read/Bash 读取 state.json 和 artifacts 目录）：

1. **Run 基本信息**：run_id, status, spec_status, input_source
2. **阶段进度**：当前处于哪个 Phase（planning/analyzing/specifying/executing/completed/blocked）
3. **Artifact 状态**：
   - analysis-report.json: ✅ 存在 / ❌ 缺失
   - plan.json: ✅ 存在 / ❌ 缺失
   - dev-report-tN.json: 逐 task 显示
   - review-report-tN.json: 逐 task 显示
4. **Task 统计**：total, completed, pending, failed, dispatched, reviewing
5. **阻塞信息**：blocked 原因（如有）
6. **Spec 门禁**：spec_status 值，是否可进入 executing
