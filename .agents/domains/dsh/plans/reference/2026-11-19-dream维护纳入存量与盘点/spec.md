---
name: "dream维护纳入存量与盘点"
description: "scan 同时给增量 turn 与全部 active+reference 的 Plan 和领域文档盘点；无增量也维护领域文档；dream 可把无用的 reference Plan 迁入 archived。"
status: completed
---

# dream扫描纳入plan与领域文档

## 背景

dsh 的 `nano_flow_dream` scan 只干一件事：按「Plan × 会话」游标捞出游标之后的新 turn。Plan 靠 `log.jsonl` 存在才被发现，`active` / `reference` / `archived` 三态全扫；没有新 turn 的 Plan 只在「无新增（只读）」段里留一个计数，连路径都不给。结果是：**没有新 turn 的轮次，dream 拿不到任何可处理对象，只能空转。**

领域文档的维护完全依赖提示词自觉（`skills/dream/SKILL.md` 第 2 步），scan 不提供任何库存事实，也没有「最近若干 Plan」的范围概念。

## 目标

1. scan 同时输出两类输入：**新证据**（游标之后的新 turn）与**存量对象**（全部 `active` + `reference` 的 Plan、每域领域文档盘点）。
2. turn 没有增量时，dream 仍要维护领域文档：对着存量对象与文档盘点做复核、提炼，不空转。
3. dream 增加归档动作：`reference` 中确认无用的 Plan 迁入 `archived`，同步相对链接。

## 口径

- 范围：`.agents/domains/*` 全部域，只扫 `active` 与 `reference`；**`archived` 不参与扫描**。
- Plan 数量不设上限（弃用 pi 的「每域最新 21 个」；候选就是全部 active + reference）。
- 游标语义不变：仍按「Plan × 会话」记 seq 上界，只前进，仍只在 `nano_flow_check` 通过且 commit 成功后才 mark。
- 无落差就不制造改动：无增量轮次允许「核查后无改动」，但不允许「没核查就结束」。

## 非目标

- 不改 pi 侧（pi 无游标、取最新 21 的模型保持原样）。
- 不改 `nano_flow_check` 的校验规则与生成物。
- 不引入新的 CLI 入口。

## 验收

- 一个没有新 turn 的工作区里跑 scan，输出仍带每域全部 active+reference 的 Plan 清单与文档盘点，且不报错。
- dream 在无增量轮次能说出「核查了哪些域、改了什么/为什么没改」。
- 归档动作留下可回溯证据（迁移的 Plan 路径 + 理由），链接与 `nano_flow_check` 通过。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
