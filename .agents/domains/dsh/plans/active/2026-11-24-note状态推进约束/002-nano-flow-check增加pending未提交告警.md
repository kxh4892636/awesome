---
name: "002-nano-flow-check增加pending未提交告警"
description: "check 对 active+pending+未提交+有正文的 note 报警；无 git 静默跳过，两包同改。"
status: blocked
reason: "实现与测试已通过，待用户验收"
related_noted: []
next_step: "用户验收后置 completed；003 继续发布"
---

# 002-nano-flow-check增加pending未提交告警

## 交付

- dsh `documents/lib/pacing.mjs`：`collectDirtyPaths(rootDir)` 用 `git status --porcelain -z` 取未提交路径；失败返回 null。`pacingWarnings(ctx, plan, dirty)` 做纯判定。
- dsh `documents/check-domain.mjs`：`scanPlans` 把 lifecycle 传给 `scanPlan`；只对 `active` 调 `pacingWarnings`。
- pi 侧同步 `scripts/lib/pacing.mjs` 与 `scripts/check-domain.mjs`。

## 判定

- 条件：active Plan、note 状态 `pending`、正文 > 120 字节、description 不以「占位」开头、note 已跟踪且 `git status` 为 M/R。`??` 新建与 A 新增不报，避免把刚建的 note 当成已开工。
- warning rule 用 `pacing`，文案：`note 仍是 pending，但工作区有未提交改动；第一次改正文前应置 in_progress`。
- 无 git / 非仓库 / 命令失败 / 超时 2s → 返回 null，不报。

## 验收

- 临时 git 仓库：pending note 有未提交正文改动 → 1 条 warning，`ok` 仍 true，`fixed` 为 0。
- 提交后、`in_progress`/`completed`、reference Plan、占位 note、无 git → 0 条。
- 连跑两次：第二次 `fixed`/`updated` 为 0，warning 仍在。
- dsh 与 pi 的 `lib/` 差异符合 common 004；两包测试通过。

## 结果

- dsh 新增 `src/nano-flow/documents/lib/pacing.mjs`：`isModification`(:19，`??`/`A` 跳过)、`collectDirtyPaths`(:34，先 `git rev-parse --show-toplevel` 再 `git status --porcelain -z`，2s 超时、rename 第二段跳过、异常返回 `null`；仓库根路径换算成工作区相对路径)、`pacingWarnings`(:64，active + pending + description 不以占位开头 + 正文 >120 字节 + 命中脏路径 → rule `pacing`)。
- dsh 新增 `src/nano-flow/documents/lib/pacing.d.mts`：测试直接 import 用的类型声明；构建的 `copy-nano-flow-assets` 会过滤 `.d.mts`。
- dsh `documents/check-domain.mjs`：:27 引入；:189 `scanPlan` 收 `lifecycle`；:292 读完全部 note、spec 派生之前调 `pacingWarnings`；:335 返回值带 `lifecycle`；:393-400 `scanPlans` 传 `lifecycleEntry.name`；:513-514 `checkWorkspace` 只收集一次 `ctx.dirty`。
- dsh 测试 `documents/check-domain.test.ts`：:670 纯判定 6 例（命中/in_progress/completed/reference+archived/占位/短正文/空与 null dirty）、:726 `collectDirtyPaths` 4 例（`??`/`A` 跳过、rename 收新路径、子目录工作区路径换算、无 git 返回 null）、:779 集成（pending 改正文 → 1 条 warning 且 `ok=true`、连跑 `fixed`/`updated` 为 0、提交后 0）、:812 无 git 时静默跳过。
- pi：`src/nano-flow/scripts/lib/pacing.mjs` 与 dsh 逐字一致（`diff` 无输出）；`scripts/check-domain.mjs` 同样接线（:23 / :185 / :288 / :331 / :392-395 / :509-510）。
- pi 测试 `src/nano-flow/check-domain.test.ts:304`：同口径集成用例 1 条（`describe.skipIf` 无 git 时跳过）。
- 测试：dsh `pnpm exec vp test --run src/nano-flow` → 174 passed，`pnpm exec vp check` 全过；pi nano-flow → 60 passed（全量 299），`pnpm exec vp check` 全过。
- 偏差/风险：
  - `git status` 的路径相对仓库根；已用 `git rev-parse --show-toplevel` 换算到工作区相对路径，并补了子目录回归用例。
  - 未提交的全新 Plan 目录整体是 `??`，不报警（符合「新建 note 不算开工」）；首轮靠固定提示词，不靠告警。
  - 告警每轮最多一次 `git status`，2s 超时即静默跳过；大仓/网络盘上可能漏报。
  - 未改版本号、未 build/pack、未提交。
