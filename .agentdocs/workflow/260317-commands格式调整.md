# .agentdocs/workflow/260317-commands格式调整.md

## 任务概述
将 .opencode/commands 中的命令提示词结构对齐到 oh-my-openagent 的块式写法（identity/mission/constraints/workflow/output）。

## 当前分析
- 命令模板属于“执行入口文本”，不要求像 agent 一样复杂，但可以统一为块式结构。
- 需要调整的主要是：dispatch / kickoff / sync（三个有实际内容的命令）。
- 纯 `$ARGUMENTS` 的 wrapper 命令无需改动。

## 方案设计
保留语义不变，仅重排为标签块：
<identity> → <mission> → <hard_constraints> → <workflow>/<output_contract> → <context>。

## 实施计划
### Phase 1: 文件修改
- [x] T-01：改写 dispatch / kickoff / sync 的命令模板（Markdown）
- [x] T-02：同步更新 opencode.json 内对应 command.template

### Phase 2: 校验
- [x] T-03：结构校验（front matter + 标签配对）并记录结果

## 备注
- 仅调整格式，不改变原有语义与约束。
