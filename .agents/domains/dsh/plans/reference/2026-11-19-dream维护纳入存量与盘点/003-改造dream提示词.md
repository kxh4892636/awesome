---
name: "003-改造dream提示词"
description: "dream skill 按一个维护循环重写为 5 步，细则下沉到 reference，无增量轮次也出结论。"
status: completed
related_noted:
  - 001-定案scan输入模型
---

# 003-改造dream提示词

## 交付

重写 [skills/dream/SKILL.md](../../../../../repos/dsh-toolkit/src/nano-flow/skills/dream/SKILL.md)：

1. 结构按一个循环组织：取输入 → 定范围 → 维护 → 归档 → 收尾，五步平铺，120 行降到 57 行。
2. 输入说明写清三类输入（新证据 / 存量 Plan / 领域文档盘点）与各自用法。
3. 定范围写明：有新证据的域、带「域文档可能落后」标记的域；没有标记的轮次抽查离现在最近的 Plan 所属域，照样出结论。
4. 收尾不再以「没有新增 turn」为结束条件，改为报告核查范围、改动或未改原因、归档项、未处理项。
5. 细则按需下沉：`reference/轨迹.md`（`log.jsonl` 格式与 `python3` 切片）、`reference/领域文档.md`（知识归属、五个 pass、数量整理）、`reference/并行.md`（多域 `subagent` 编排）。

## 结论

- 遵循 `to-skill`：一个 skill 一个循环、leading word 锚定（新证据 / 存量 Plan / 域文档可能落后 / 上浮 / 复核 / 归档）、负向句改写成正向要求（「不允许没核查就结束」→「没有标记的轮次抽查最近的域」）。
- 人工调用两件套补齐：`disable-model-invocation: true` + 新增 `agents/openai.yaml`（`policy.allow_implicit_invocation: false`）。
- 删掉「单次运行超过预算就停在该 pass 边界」——全文没定义「预算」，答不出它改变哪种默认行为。
- `runtime.test.ts` 断言 SKILL.md 里必须出现 `python3` 与 `subagent`，下沉后这两个词留在指针句里，测试未改。

## 验证

- `src/nano-flow/assembly/runtime.test.ts` 8 例通过；全量 291 例全绿。
- `scripts/copy-nano-flow-assets.mjs` 对 `skills` 整树复制，`reference/` 与 `agents/` 随包发布。

## 相关

- [001-定案scan输入模型](001-定案scan输入模型.md) — 行为口径
- [005-归档无用的reference-plan](005-归档无用的reference-plan.md) — 同一份提示词的归档动作
