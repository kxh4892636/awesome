---
name: "001-新增backup-plan手工技能"
description: "随包 skill：按整体内容重命名当前 Plan 并归档，只能人工调用。"
status: blocked
reason: "0.5.75 已装进 web profile，等用户在斜杠菜单确认 /backup-plan 出现。"
related_noted: []
next_step: "重启宿主后在斜杠菜单确认 backup-plan 出现且不参与模型自动挑选"
---

# 001-新增backup-plan手工技能

## 交付

- `src/nano-flow/skills/backup-plan/SKILL.md`：正文固定为按整体内容重命名当前 Plan 后归档，带 `disable-model-invocation: true`。
- 随包技能目录打包时自动带走（`scripts/copy-nano-flow-assets.mjs` 复制整个 `skills/`），无需登记清单。

## 实现要点

- 重命名只改目录名：`<原日期>-<内容概括>`，note 与 spec 正文不动；日期取原目录名前缀。
- 归档 = 移到 `plans/archived/`（与 active 同级，指向领域文档的相对链接深度不变，只有直接写旧目录名的链接要改）。
- 动完跑 `nano_flow_check` 再 `nano_flow_bind` 到新路径（`lifecycle: archived`）：绑定存的是 `planPath`，不改就会指向不存在的目录。

## 验收

1. 斜杠菜单里出现 `/backup-plan`，模型不会自动挑中它。
2. 人工调用后：目录名按内容变化、位置在 `plans/archived/`、绑定指向新路径、`nano_flow_check` 通过。
3. `vp check` 与 `vp test src` 全绿。

## 证据

- `vp check` 通过；`vp test src` 36 个文件 288 例全绿（技能只是资产，无新代码路径）。
- 打包 `0.5.75` 已 `remove` 后 `add` 进 web profile；`dist/nano-flow/skills/` 下已有 `backup-plan`，SKILL.md 头部带 `disable-model-invocation: true`。
- 待填：重启后斜杠菜单里 `/backup-plan` 的确认。
