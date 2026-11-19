---
name: "文档上限与backup技能"
description: "新增人工调用的 backup-plan 技能；给 workflow、ADR 与 Plan 的 note 加数量上限（默认 89）。"
status: in_progress
---

# 文档上限与backup技能

## 背景

两件独立的文档治理工作，2026-11-19 由用户提出：

1. 领域里的 workflow 与 ADR 越攒越多，Plan 里的 note 也会变多，缺少数量上限与到顶后的处理规则。
2. 需要一个手工技能，把当前 Plan 按整体内容重命名后归档，避免大量 Plan 名字与内容脱节。

## 目标

- 技能：`backup-plan` 随包分发，`disable-model-invocation`，只能用户手动调用；内容是按整体内容重命名当前 Plan、移到 `plans/archived/` 并重新绑定。
- 上限：新增三个数量上限（每域 workflow 数、每域 ADR 数、每 Plan note 数），默认值都是 89；超限时 check 要给出明确处理动作——workflow/ADR 提示重构，note 给出「重构 note」与「把 Plan 拆成 A/B，旧 note 归 A、新 note 归 B，并 bind 到 B」两个选择。

## 未定问题

- 「workflow 与 ADR 自动重构」与现有「check 只就地修由既有事实唯一确定的问题」冲突：合并哪些、怎么合不是唯一解。需要定成「check 只报清单，模型执行」，还是「check 生成确定的合并方案」。
- 拆分 Plan 的实现边界：是否由 check 直接搬文件、还是只输出指令由模型执行；`log.jsonl` 与绑定如何跟着走。

## 相关

- [001-新增backup-plan手工技能](001-新增backup-plan手工技能.md) — 本 Plan 的第一项交付

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
| [001-新增backup-plan手工技能](001-新增backup-plan手工技能.md) | 随包 skill：按整体内容重命名当前 Plan 并归档，只能人工调用。 | blocked | 0.5.75 已装进 web profile，等用户在斜杠菜单确认 /backup-plan 出现。 | 重启宿主后在斜杠菜单确认 backup-plan 出现且不参与模型自动挑选 | — |
