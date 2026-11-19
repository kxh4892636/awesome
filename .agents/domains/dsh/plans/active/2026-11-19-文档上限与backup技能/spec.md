---
name: "文档上限与backup技能"
description: "新增人工调用的 backup-plan 技能；在 dream 提示词里要求每域 question/ADR/workflow 不超过 89 条。"
status: in_progress
---

# 文档上限与backup技能

## 背景

两件独立的文档治理工作，2026-11-19 由用户提出：

1. 领域里的 workflow 与 ADR 越攒越多，Plan 里的 note 也会变多，缺少数量上限与到顶后的处理规则。
2. 需要一个手工技能，把当前 Plan 按整体内容重命名后归档，避免大量 Plan 名字与内容脱节。

## 目标

- 技能：`backup-plan` 随包分发，`disable-model-invocation`，只能用户手动调用；内容是按整体内容重命名当前 Plan，`reference` 还是 `archived` 由技能给建议、用户拍板，再移动并重新绑定。
- dream：原 `dream/prompt.md` 整体搬成 `skills/dream/SKILL.md`（同样只能人工或定时任务点名调用），宿主定时任务只投一句「调用 dream 技能」。
- 上限：每域 `question` / `ADR` / `workflow` 各不超过 89 条；超限由 dream 在维护时整理。**只写进 dream 提示词，不改代码**：不新增设置项、不给 check 加条数校验、不碰 Plan 与 note 的数量。

## 决策（2026-11-19）

- 「check 超限自动重构」与「check 只修唯一确定的问题」冲突，且 0.5.3 曾因同一理由删掉 ADR 数量上限；最终改成 dream 侧的人工式整理：数字 89 写死在 `dream/prompt.md`，由 dream 每轮自查并合并/剪枝/拆分。
- Plan 拆分成 A/B、note 数量上限一并取消（用户明确「plan 的拆分就不用管了」）。
- 已把代码侧改动（设置项、面板字段、check 条数校验、注入块的计数行）全部回退，0.5.76 只含 dream 提示词的这段改动。

## 相关

- [001-新增backup-plan手工技能](001-新增backup-plan手工技能.md) — 本 Plan 的第一项交付

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
| [001-新增backup-plan手工技能](001-新增backup-plan手工技能.md) | 随包 skill：按整体内容重命名当前 Plan，reference/archived 给建议由用户定。 | blocked | 0.5.76 已装进 web profile，等重启后在斜杠菜单确认 backup-plan 与 dream 两个技能出现。 | 重启宿主，确认 /backup-plan 与 /dream 出现在菜单且不被模型自动挑中 | — |
