---
name: "005-归档无用的reference-plan"
description: "dream 在维护时把 reference 中确认无用的 Plan 迁入 archived，三条判据同时成立才动。"
status: completed
related_noted:
  - 003-改造dream提示词
---

# 005-归档无用的reference-plan

## 交付

dream 提示词新增「归档」步（SKILL.md 第 4 节 + 写入边界补充）：

- 判据三条同时成立，且能在 note、spec 或领域文档里指到依据：结论已上浮 / note 全 `completed` 且 spec `completed` / 无唯一出处。
- 动作：`git mv` 迁目录 → 修指向它的相对链接 → `nano_flow_check` → commit message 写明迁移项与理由。
- 约束：单次 commit 最多归档 3 个；`active` 一律不自动归档；第 3 条真查引用，不能只看 note 状态。

## 结论

- 写入边界原本「只读 note」与「迁移要改链接」自相矛盾，补明：迁移可同步指向它的相对链接，含 note 的 `## 相关` 条目，只改路径不改正文。
- 判据 3 无法机械化，提示词只能要求真查；可靠性取决于执行者，属于已知限制。

## 验证

- 提示词落地，代码侧无改动。
- 真实归档试运行留给 [004-单测与发布验收](004-单测与发布验收.md) 的真机验收。

## 相关

- [003-改造dream提示词](003-改造dream提示词.md) — 同一份提示词承载该动作
