---
name: "016-note状态推进以提示词为主"
description: "开工即 in_progress 靠固定提示词落地；check 只对 active+pending 且有未提交改动的 note 报警，不改状态、不派生。"
status: accepted
---

# 016-note 状态推进以提示词为主

## 背景

note 的四个状态里只有两端有口径：交付完成未验收一律 `blocked`、用户验收后才 `completed`（[006](006-note阻塞态以reason表达.md)）。中间那段在 2026-11-09 取消「状态先行」后没人管——那轮取消的理由是「每次状态变更一次写入」太贵，不是状态不该推进（[009-调研Plan写入现状与惰性化取舍](../plans/reference/2026-11-09-nano-flow优化迭代/009-调研Plan写入现状与惰性化取舍.md)）。

现场因此是「默认不用、偶尔用」：19/84 条 note 在 `pending` 下改过正文，54/170 次提交落在 `pending`；077/081/082/083 各 11/4/10/3 次提交、0 次 `in_progress`，而同一天 078 置过一次并保持 10 次提交（口径与来源见 [spec](../plans/active/2026-11-24-note状态推进约束/spec.md) 的「证据」）。面板上的 `in_progress` 近乎空档，看不出哪条正在做。

check 也不报：spec 状态由 note 派生，note 停在 `pending` 不违反任何规则。

## 决策

主杠杆放固定提示词——dsh 与 pi 的 `system.md` note 段各两行、逐字一致：开工即 `in_progress`（轮首批量写先把状态置上；中途才定的，在第一次改正文前补一次只改 YAML 头的写入，不占批量写额度），`pending` 只表示还没开始。

`nano_flow_check` 只做旁证：`active` Plan 里 `pending` 的 note 若正文超过 120 字节、description 不以「占位」开头、且已跟踪但有未提交改动（`git status` 的 M/R），报一条 `pacing` 警告。不自动改状态、不从告警派生 spec 状态、不新增 spec 模板骨架。

## 后果

- 警告随未提交改动常驻：正常节奏「写 → check → commit」里第一次 check 就会提示，提交后消失。
- 没有 git、不在仓库、命令失败或超时（2s）时跳过该告警——退化成「只有提示词」，不是错误。
- `??` / `A` 的新建 note 不报：刚建的 Plan 整目录未跟踪，首轮只靠提示词。
- 告警只降低漏置的概率，不保证照做；是否做完仍要在 `blocked` → `completed` 时由用户验收判定（[010](010-note按交付单元划分.md)）。

## 相关

- [001-固定提示词补note状态推进规则](../plans/active/2026-11-24-note状态推进约束/001-固定提示词补note状态推进规则.md) — 两包 `system.md` 的两行与断言
- [002-nano-flow-check增加pending未提交告警](../plans/active/2026-11-24-note状态推进约束/002-nano-flow-check增加pending未提交告警.md) — `pacing.mjs` 的判定与两包接线
- [004-两包校验规则是否同源](../../common/questions/004-两包校验规则是否同源.md) — 同源范围已纳入 `pacing.mjs`
- [006-note阻塞态以reason表达](006-note阻塞态以reason表达.md) — 状态与 `reason` 口径
