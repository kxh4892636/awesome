---
name: "001-新增backup-plan手工技能"
description: "随包 skill：按整体内容重命名当前 Plan，reference/archived 给建议由用户定。"
status: blocked
reason: "0.5.76 已装进 web profile，等重启后在斜杠菜单确认 backup-plan 与 dream 两个技能出现。"
related_noted: []
next_step: "重启宿主，确认 /backup-plan 与 /dream 出现在菜单且不被模型自动挑中"
---

# 001-新增backup-plan手工技能

## 交付

- `src/nano-flow/skills/backup-plan/SKILL.md`：按整体内容重命名当前 Plan，`reference` 还是 `archived` 先给建议与理由、由用户选，然后移目录、改链、重绑。带 `disable-model-invocation: true`。
- `src/nano-flow/skills/dream/SKILL.md`：原 `dream/prompt.md` 整体搬成技能（同样人工/定时任务点名调用），宿主定时任务只投一行 `/dream` 手势加一句要求。
- 随包技能目录打包时自动带走，无需登记清单。

## 实现要点

- 重命名只改目录名：`<原日期>-<内容概括>`，note 与 spec 正文不动。
- 归档目标不写死：note 全 completed 推荐 `reference`，计划废弃、结论被取代推荐 `archived`；用户选哪个就移到 `plans/<lifecycle>/` 并 `nano_flow_bind` 到同一 lifecycle。
- 绑定存的是 `planPath`，改名搬运后不重绑就会指向不存在的目录。
- dream 搬家用 `git mv` 保留历史；同时删掉 `assetsDreamPromptPath`、copy 脚本映射与 README 里的旧路径。

## 验收

1. 斜杠菜单里出现 `/backup-plan` 与 `/dream`，模型不会自动挑中它们。
2. 人工调用 backup-plan 后：目录名按内容变化、位置是用户选的 lifecycle、绑定指向新路径、`nano_flow_check` 通过。
3. 定时任务到点后能加载 dream 技能并跑完 scan。
4. `vp check` 与 `vp test src` 全绿。

## 证据

- `vp check` 通过；`vp test src` 36 个文件 288 例全绿。
- 打包 `0.5.76` 已装进 web profile；`dist/nano-flow/skills/` 有 `backup-plan` 与 `dream`，`dist/nano-flow/prompts/dream.md` 已消失。
- 定时任务 `schedule-d4311690…` 的 prompt 已换成 `/dream …`（斜杠手势），下次触发 22:00。不能用「调用 skill 工具」的写法：`skill` 工具对 `disable-model-invocation` 的技能直接抛错，只有 `/name` 手势走 userInvocable 注入。
- 提交：`b99c312`（技能改动）、`ac410c5`（0.5.76）。
- 待填：重启后的菜单确认。
