---
name: "003-速查卡规则上浮到to-skill"
description: "结论：压缩比例与多级标题两条规则留在 learning 的 CHEATSHEET.md；写进 to-skill 的两行已回退。"
status: completed
related_noted:
  - "001-速查卡路由与版面规则"
next_step: 无
---

# 003 规则归属：留在 learning

2026-11-24 曾把「压缩比例」与「多级标题」作为通用规则写进 [to-skill](../../../../../../repos/dsh-toolkit/src/nano-flow/skills/to-skill/SKILL.md)，用户澄清后全部回退。

## 结论

- 两条规则是速查卡自己的写法，权威位置在 [CHEATSHEET.md](../../../../../../repos/dsh-toolkit/src/nano-flow/skills/learning/CHEATSHEET.md)：§4 篇幅（书 3%）、§5 版式（最多 4 层）；
- to-skill 保持原样，只讲循环、信息层级、leading words 与 pruning，不掺交付物的配额。

## 过程

- `bb929be` 加了 **Headings**（最多 4 层）与 **Budget**（5%~8%）两行；
- 用户指出这是速查卡规则，不该动 to-skill；
- `6e9f746` 把 to-skill 恢复到 `c4a1e47` 的内容，文件与改前逐字一致。

## 验收

- `git diff c4a1e47 HEAD -- src/nano-flow/skills/to-skill/SKILL.md` 为空；
- CHEATSHEET §4 用 3%，§5 用 4 层，两处口径不交叉。
