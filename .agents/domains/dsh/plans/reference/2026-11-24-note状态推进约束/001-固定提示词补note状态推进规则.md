---
name: "001-固定提示词补note状态推进规则"
description: "dsh 与 pi 的 system.md 补「开工即 in_progress」一行，按 to-skill 收敛措辞并加断言。"
status: completed
reason:
related_noted: []
next_step: "已交付并验收；规则见 ADR 016"
---

# 001-固定提示词补note状态推进规则

## 交付

- dsh `src/nano-flow/session/prompts/system.md`：`### note` 末尾加一行。
- pi `src/nano-flow/prompts/system.md`：同一位置加同样一行。
- dsh `assembly/prompt-rules.test.ts`：断言「开工即 `in_progress`」。

## 文案

```md
- 开工即 `in_progress`;
```

## 验收

- 两包 system.md 的 note 段逐字一致。
- `pnpm test src/nano-flow/assembly/prompt-rules.test.ts` 通过。
- 文案符合 to-skill：正向陈述、leading word「开工即 in_progress」、无解释段。

## 结果

- dsh `src/nano-flow/session/prompts/system.md:91` 一行；pi `src/nano-flow/prompts/system.md:96` 同一行，两包逐字一致。
- dsh `src/nano-flow/assembly/prompt-rules.test.ts:27-30` 断言「开工即 `in_progress`」；pi 无提示词测试文件，字面由 dsh 侧锁住。
- 测试：dsh nano-flow 174 passed、pi nano-flow 60 passed；两边 `vp check` 全过。
- 风险：提示词只锁字面，是否照做要看 003 真机验收。
