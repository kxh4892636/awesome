---
name: "003-速查卡规则上浮到to-skill"
description: "把压缩比例与多级标题两条通用规则写进 to-skill：Headings 最多 4 层，Budget 按源材料 5%~8%。"
status: completed
related_noted:
  - "001-速查卡路由与版面规则"
next_step: 无
---

# 003 速查卡规则上浮到 to-skill

速查卡里两条规则其实是通用写作规则，换成任何交付物都成立，所以上浮到 [to-skill](../../../../../../repos/dsh-toolkit/src/nano-flow/skills/to-skill/SKILL.md)，速查卡只留自己的配额。

## 交付

- to-skill 的「信息层级」加 **Headings**：标题最多 4 层，skill 名 → 分区（`##`）→ 子分区（`###`）→ 条目；两级说得清就不开三级；
- to-skill 的「Pruning」加 **Budget**：篇幅跟着源材料走，书或长文档取 5%~8%，上限不是配额；到上限还装不下就下沉成 reference，分区只留指针；
- 两条都用 leading word 起头，与现有 **Sprawl** / **Co-location** / **Relevance** / **No-ops** 同形。

## 为什么放这里

- 归位：`##` / `###` 的分层与 5%~8% 的篇幅预算管的是「写多少、怎么分层」，不是速查卡专有；
- 不重复：速查卡留自己的换算（5 栏、约 8,700 字/页、一页一个查找场景），to-skill 只给通用口径；
- 副作用可控：to-skill 只多两行，没有新开 section，仍守它自己的 Sprawl 与 No-ops 要求。

## 验收

- to-skill 正文 7 行新增生效，`pnpm build`、`pnpm check`、`nano_flow_check` 通过；
- 0.5.94 已装进 web profile，安装副本里两条规则可见。
