---
name: 012-各级目录补齐README索引
description: 020-计算机 各级目录补 README.md 索引（只列链接），并更新 docs/README.md 的栏目索引，排除被 gitignore 的目录。
status: completed
related_noted: []
---

# 012-各级目录补齐README索引

## 决策

- 每个目录一个 `README.md`，形态照 `090-归档`：H1 是目录名，空行，然后是 `- [条目](./目标)` 列表，**不写描述、不写 frontmatter、不写「相关」小节**；
- 叶子目录的链接文字 = 文件名去掉 `NNN-` 编号；父目录只列子目录，链接指向子目录的 `README.md`；
- `docs/README.md` 只列会发布的栏目：`020-八股文梳理`、`020-计算机`、`090-归档`、`perf.md`；`000-中转`、`010-记录`、`070-工作`、`code` 被 `.gitignore` 排除，不进索引。

## 为什么给 020-八股文梳理 也补一个

它原先没有落地页（侧栏里只是折叠分组）。不补的话 `docs/README.md` 无法链接它——构建是 `onBrokenLinks: throw`，指向目录的链接会直接失败。补上后它有了目录页，域文档里「没有 README」这条要同步改。

## 交付

`020-计算机` 下 21 个目录各一个 `README.md`（`docs/020-计算机/README.md` + 5 个一级 + 16 个二级），加 `020-八股文梳理/README.md` 与 `docs/README.md`，共 23 个文件、约 180 条相对链接。

## 验收

- 25 个 README 的每条 `./` 链接都能解析到磁盘上的真实文件；
- 每个目录下的子目录与 `.md` 都被对应 README 列出（自查脚本逐目录比对）；
- `docs/README.md` 不含被忽略的四个目录；
- `pnpm build` 通过。
