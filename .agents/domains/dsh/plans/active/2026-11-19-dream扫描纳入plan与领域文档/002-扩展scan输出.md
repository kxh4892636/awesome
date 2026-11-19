---
name: "002-扩展scan输出"
description: "scan.mjs 收敛为 active+reference，新增存量 Plan 与领域文档盘点两段输出，无增量也不早退。"
status: completed
related_noted:
  - 001-定案scan输入模型
---

# 002-扩展scan输出

## 交付

改 [scan.mjs](../../../../../repos/dsh-toolkit/src/nano-flow/dream/scan.mjs)：

1. 遍历范围收敛为 `active` + `reference`，`archived` 不再参与（游标里的旧条目原样保留）。
2. 新增存量 Plan 采集（按目录枚举，不以 `log.jsonl` 为门槛）：每域全部 active+reference Plan 的 lifecycle、`spec.md` status、note 完成数/总数、目录内最新 mtime、`log.jsonl` 字节数。
3. 新增领域文档盘点：CONTEXT 字节与 mtime、三目录条目数与最大编号、三份 index 是否存在、GUIDE mtime。
4. 新增落差标记：最新 Plan 的日期晚于 CONTEXT。
5. `formatReport` 增加 `## 存量 Plan（active + reference，全部）` 与 `## 领域文档盘点` 两段，新证据段改名 `## 新证据：<domain>`；`report` 结构与 `scan.d.mts` 同步扩展。

## 结论

- 增量语义零变化：新 turn 仍要求 `log.jsonl` 存在，只有存量盘点改成按目录枚举。原定「Plan 发现统一看 `log.jsonl`」不成立——29 个 active+reference Plan 里只有 8 个有轨迹，dsh 域 13 个里只有 1 个；按原判据存量段只剩 1 条，与验收（与 `ls` 一致）冲突。
- 落差按天比较，不按时间戳：同日几分钟的改动会把域永久刷成「可能落后」。
- 删掉未再渲染的 `idle` 字段（旧「无新增（只读）」段已被存量段覆盖），对应单测改为断言 `inventory`。
- 无新证据时的首行从「没有新增 turn。」改成「新证据：无（本轮按存量 Plan 与领域文档盘点维护）」，避免读成收工信号。

## 验证

- `pnpm test src`：36 文件 / 291 例全绿（`scan.test.ts` 11 例，含新增「无新 turn 仍输出存量与盘点」「archived 不参与」「落差标记是/否」）。
- `pnpm check`：格式、lint、类型均通过。
- 真实工作区冒烟：dsh 存量 13 条 = `ls plans/{active,reference}` 的 13；全域 7 域 32 个 Plan。

## 相关

- [001-定案scan输入模型](001-定案scan输入模型.md) — 输出形态与口径来源
- [004-单测与发布验收](004-单测与发布验收.md) — 发布与真机验收
