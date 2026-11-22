---
name: '002-归档Plan的模板缺项不再报错'
description: '下线前的 nnf 文档校验只查未归档 Plan 的模板缺项；现行 pi / dsh check 扫全部生命周期，不再有模板缺项这类规则。'
---

# 002-归档Plan的模板缺项不再报错

## 适用条件与现象

Plan 迁入 `reference` 或 `archived` 后，`node <nano-flow-skill-root>/scripts/check-domain.mjs .` 不再报告其缺少「执行检查点」「决策与证据」或 story.md。

> 纠正（2026-11-22，dream 复核）：带这套模板规则的流程随 nnf 的 `workspace` 命令组与文档流程 skill 于 2026-11-09 下线；现行校验是 pi / dsh 随包的 `check-domain.mjs`，扫各域 `plans/` 下全部生命周期，只查 YAML 头、三位编号、篇幅与 status/reason/next_step 口径，没有「模板缺项」这类检查。原来指向的 `domain/PLAN.md` 已不存在，模板在 `repos/dsh-toolkit/src/nano-flow/documents/templates`。下文是下线前的历史行为。

## 原因

`reference` 与 `archived` 是冻结内容，模板随 skill 演进；只校验仍在推进的 `planning` 与 `implementing`，避免历史缺项淹没本次新增问题。

## 解决方法

Plan 在迁入冻结生命周期前完成模板校验；归档后不补写历史文档，恢复工作时先迁回 `planning` 或 `implementing`。

## 验证

校验输出只包含 `planning` 与 `implementing` 下的 Plan。

## 来源

[dsh 侧 check-domain.mjs](../../../../repos/dsh-toolkit/src/nano-flow/documents/check-domain.mjs)（现行实现）；旧 `domain/PLAN.md` 路径已随流程 skill 下线删除。
