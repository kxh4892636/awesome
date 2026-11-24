---
name: "001-固定提示词补note状态推进规则"
description: "dsh 与 pi 的 system.md 补「开工即 in_progress」两行，按 to-skill 收敛措辞并加断言。"
status: blocked
reason: "实现与测试已通过，待用户验收"
related_noted: []
next_step: "用户验收后置 completed；003/004 已交付"
---

# 001-固定提示词补note状态推进规则

## 交付

- dsh `repos/dsh-toolkit/src/nano-flow/session/prompts/system.md`：`### note` 末尾（第 90 行后）加两行。
- pi `repos/pi-toolkit/src/nano-flow/prompts/system.md`：同一位置加同样两行；本地未 clone，先 clone 到 `repos/pi-toolkit`。
- dsh `assembly/prompt-rules.test.ts`：加断言「开工即」与「第一次改正文」。

## 文案

```md
- 开工即 `in_progress`：轮首批量写先把状态置上；中途才定的，在第一次改正文前补一次只改 YAML 头的写入（不占批量写额度）。
- `pending` 只表示还没开始；从第一次改正文起就是 `in_progress`，直到完成或阻塞。
```

## 验收

- 两包 system.md 的 note 段逐字一致。
- `pnpm test src/nano-flow/assembly/prompt-rules.test.ts` 通过。
- 文案符合 to-skill：正向陈述、leading word「开工即 in_progress」、无重复定义。

## 结果

- dsh `src/nano-flow/session/prompts/system.md:91-92`：`### note` 末尾逐字加两行（「开工即 `in_progress`…」与「`pending` 只表示还没开始…」）。
- dsh `src/nano-flow/assembly/prompt-rules.test.ts:27-30`：新增断言 `开工即` 与 `第一次改正文`。
- pi `src/nano-flow/prompts/system.md:96-97`：同一位置、同两行；与 dsh 该两行逐字一致。
- pi 断言：pi 无提示词测试文件（只有 `register.ts:59` 读 system.md），按口径未新增，仅由 dsh 侧断言锁字面。
- 测试：dsh `pnpm exec vp test --run src/nano-flow` → 174 passed；`pnpm exec vp check` 全过。pi nano-flow → 60 passed（全量 299）；`pnpm exec vp check` 全过。
- 偏差：pi 仓是本次 clone 的全新 `repos/pi-toolkit`（https、main、未提交）；两包 system.md 仍有历史差异（CONTEXT-MAP 说明、ADR status 段、更新优先级措辞），本轮只加两行，未顺手拉平。
- 风险：提示词只锁字面，是否真照做要看 003 真机验收。
