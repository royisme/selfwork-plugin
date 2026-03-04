---
description: 查看 selfwork 可调度任务队列。按角色分组展示待分派工作。
---

使用 `selfwork` skill 执行队列计算流程并返回可分派任务列表。

输出以下信息：

1. **当前阶段应分派的角色**：
   - analyzing → 待分派 Analyst
   - specifying → 待分派 Architect
   - executing → 按 task 分组
2. **可分派 Tasks**（executing 阶段）：
   - 依赖已满足的 pending tasks
   - 每个 task 显示：id, title, complexity, suggested agent_type, task_type, criticality
3. **待 Review Tasks**：
   - agent_done 状态的 tasks
   - 每个显示：id, title, 对应 dev-report 是否存在
4. **被阻塞 Tasks**：依赖未满足的 pending tasks，显示阻塞原因
