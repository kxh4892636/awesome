---
name: 014-Vite打包原理笔记
description: "在 030-构建与打包 下新写一篇 Vite 打包原理知识笔记，替换已删的 002，覆盖开发态与生产态两条链路。"
status: completed
---

# Vite打包原理笔记

## 交付单元

用户删掉 `030-构建与打包/002-构建工具对比.md`（Gulp/Grunt），要求在 `001-Webpack打包原理` 的写法基础上补一篇 Vite 打包原理。

- 新文件：`repos/wiki/docs/020-计算机/030-工程化/030-构建与打包/002-Vite打包原理.md`
- 结构照 [NOTE.md](file:///home/kongxiaohan.xiaoyu/.dsh/profiles/web/node_modules/@kxh4892636/dsh-toolkit/dist/nano-flow/skills/learning/NOTE.md)：H1 问句 → 常见问题 → 概念索引 → 分节正文；
- 八股题单里没有 Vite 题，常见问题写实际疑问；
- 正文 `标签: 解释;` 半角标点，代码/表格/mermaid 优先，不设参考章节；
- 篇幅 2–6 KB，条目长度与每节条数落在 learning skill 的分布内。

## 落位

- README `030-构建与打包/README.md` 里已删的 `002-构建工具对比` 链接要换成新文件；
- Anki：删掉旧 002 的卡（uuid `bd79a5fe-68de-4da7-a93b-0a65945420de`），给新笔记建一张。

## 验收

1. `note-metrics.py` 跑该目录不报越界；
2. H1 与「常见问题」条目对得上，概念索引覆盖正文所有小节；
3. UUID 全站唯一，README 相对链接可达；
4. Anki 牌组 `kxh::020-计算机::030-工程化::030-构建与打包` 张数与目录笔记数一致。
