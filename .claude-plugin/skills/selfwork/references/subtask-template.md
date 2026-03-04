# Subtask Specification Template

路径：`.claude/task-specs/<run-id>/subtasks/tN.md`

## 格式

文件头部用 JSON code block 存放结构化合约字段，描述性内容用 markdown。

```
\`\`\`json
{
  "task_id": "t1",
  "task_type": "tdd",
  "criticality": "critical",
  "test_command": "bun run test:run src/xxx.test.ts",
  "spec_source": "devDocs/spec/selfwork/xxx.md#section",
  "output_artifact": ".claude/dispatch/runs/<run-id>/artifacts/dev-report-t1.json"
}
\`\`\`

# tN: <task title>

## 目标
一句话说明本任务的核心目标。

## 目标文件
- `path/to/file-a.ts`
- `path/to/file-b.ts`

## 依赖
- tX（原因）
- tY（原因）

## 实现要点
- 列出 3-6 条关键实现点
- 每条应具体到函数/模块级别

## 验收标准
1. 结果行为可验证
2. 指定 test_command 通过
3. 无额外范围漂移

## 完成后

Developer 必须写入 dev-report JSON 到 `output_artifact` 指定路径：
\`\`\`json
{
  "run_id": "<run-id>",
  "task_id": "tN",
  "files_changed": ["path/to/file.ts"],
  "tests_written": ["path/to/file.test.ts"],
  "test_result": "pass",
  "notes": "实施备注"
}
\`\`\`
