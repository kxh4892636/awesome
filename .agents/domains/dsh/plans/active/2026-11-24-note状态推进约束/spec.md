---
name: "2026-11-24-note状态推进约束"
description: "给 nano-flow 补 note 状态推进提示词与 pending 未提交告警，发布 0.5.82 并验收。"
status: in_progress
---

# note状态推进约束

## 目标

note 从第一次改正文起进入 `in_progress`，不再长期停在 `pending` 直到 `completed`。用户已选 ① 固定提示词 + ③ check 告警；② spec 模板骨架不做。

## 证据

- 现场 077/081/082/083 各 11/4/10/3 次提交、0 次 `in_progress`；同一天 078 在 14:27 置过 `in_progress` 并保持 10 次提交——是「默认不用、偶尔用」，不是完全不会。
- 全 Plan 19/84 条 note（22.6%）在 `pending` 下改过正文；54/170 次提交落在 `pending`。
- 根因：reference Plan 009 取消「状态先行」后，只保留了 blocked/completed 口径；`system.md` 的 note 段没有进入 `in_progress` 的规则，spec 状态又由 note 派生，跳过不报错。
- ③ 的原始信号「最近有提交改过它」在 check 时点不成立：流程是写 → check → commit；改用 `git status` 未提交改动。PTC 日志里 100% 工具调用是 `run_code`，拿不到结构化文件路径。

## 交付

- [001-固定提示词补note状态推进规则](001-固定提示词补note状态推进规则.md)：dsh + pi 的 `system.md` 各加 2 行，按 to-skill 风格；补断言。
- [002-nano-flow-check增加pending未提交告警](002-nano-flow-check增加pending未提交告警.md)：active + pending + 未提交 + 有正文 → warning；无 git 静默跳过；两包同改。
- [003-发布0.5.82并真机验收](003-发布0.5.82并真机验收.md)：patch、构建、pack、安装、push、验收。
- [004-领域文档收口](004-领域文档收口.md)：CONTEXT、ADR 016、common 004；GUIDE 留给 dream。

## 口径

- 提示词是主杠杆；check 告警只提醒，不自动改状态、不计入 `ok`。
- 两包同改；本机 pi 仓未 clone，实施前先 clone 到 `repos/pi-toolkit`。
- 提交：dsh-toolkit、pi-toolkit 各一个代码 commit；awesome 的 Plan 与领域文档一个 docs commit；然后 push。
- 安装放最后：`dsh plugin --profile web remove` + `add` 新 tgz，重启宿主后生效。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
| [001-固定提示词补note状态推进规则](001-固定提示词补note状态推进规则.md) | dsh 与 pi 的 system.md 补「开工即 in_progress」两行，按 to-skill 收敛措辞并加断言。 | blocked | 实现与测试已通过，待用户验收 | 用户验收后置 completed；003 继续发布 | — |
| [002-nano-flow-check增加pending未提交告警](002-nano-flow-check增加pending未提交告警.md) | check 对 active+pending+未提交+有正文的 note 报警；无 git 静默跳过，两包同改。 | blocked | 实现与测试已通过，待用户验收 | 用户验收后置 completed；003 继续发布 | — |
| [003-发布0.5.82并真机验收](003-发布0.5.82并真机验收.md) | 涨 0.5.82，跑全量检查、构建、pack、安装 web profile、push 并验收。 | blocked | 已发布 0.5.82 并安装，待用户重启宿主/新会话验收 | 用户重启宿主或开新会话验收后置 completed | — |
| [004-领域文档收口](004-领域文档收口.md) | 更新 dsh CONTEXT、ADR 016 与 common 004，GUIDE 留给 dream。 | blocked | 领域文档已改完、check 通过，待用户验收 | 用户验收后置 completed；GUIDE 的两处口径留给 dream | — |
