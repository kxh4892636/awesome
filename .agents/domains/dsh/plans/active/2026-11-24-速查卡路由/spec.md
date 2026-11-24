---
name: "速查卡路由"
description: "给 learning 加 cheatsheet 子路由：按高频×难记取材、按知识形态分区，Markdown 经 marked 与 mermaid 渲染成固定 5 栏 A4 速查卡。"
status: completed
---

# 速查卡路由

给 `learning` 加一条速查卡产线：把书、工具文档或 SOP 压成可打印的 A4 速查卡，Markdown 写、PNG 出。

## 目标

- 内容规则：按「高频 × 难记」取材，按知识形态分区，源是书时按 5%~8% 篇幅；
- 版面规则：A4 横向、固定 5 栏、300dpi；
- 渲染链路：Markdown → HTML/CSS → Chrome headless 截图。

## 交付

- `repos/dsh-toolkit/src/nano-flow/skills/learning/CHEATSHEET.md` — 7 节规则（上卡 / 提炼 / 分区 / 篇幅 / 版式 / 渲染 / 验收）；
- `.../scripts/render-sheet.mjs` — 解析、分页、截图、指标；
- `.../assets/sheet.html` — 版面模板；
- `.../assets/vendor/` — marked 与 mermaid；
- `SKILL.md` 指针与 description；
- 领域文档：ADR 018、CONTEXT 一句。

## 关键决策

- 渲染走浏览器而不是 LaTeX/Typst：本机无 TeX 工具链，CSS 直接管折行、分栏与避头尾（[ADR 018](../../../adr/018-cheatsheet用html分栏经chrome出png.md)）；
- 富文本交给 marked；mermaid 随包、按需注入；
- 固定 5 栏、栏距 10px、栏底留 18px；页数与字号是仅有的旋钮；
- 分页按单元填充，栏底只剩不足一行；列表跨栏续排，有序列表续编号；
- 文风不单列：2026-11-24 用户删掉「条目」「文风」两节后，卡片沿用 [SKILL.md](../../../../../../repos/dsh-toolkit/src/nano-flow/skills/learning/SKILL.md) 第 3 节，只保留渲染与版面规则。

## 验收

- 首个真实用例：计算机网络（529 页书 + 本地笔记）出 855 条、30,097 字，5pt 排 4 页，压缩比 4.95%，拼成 3369×9528 单张 PNG；
- 19,684 字样例：6pt 出 5 页、5.5pt 出 4 页；页 1 末行距纸边 53px，等于版心留白；
- 富文本样例覆盖粗斜体、删除线、链接、引用、嵌套列表、表格、代码块、flowchart 与 sequenceDiagram；
- `pnpm build`、`pnpm check`、`nano_flow_check` 通过。

## 事实核查

- 书 529 页；抽样 10 页平均 1,149 字/页，估 60.8 万字；
- Chrome `--force-device-scale-factor=3` 截 1123×794 得 3369×2382，约 288dpi；
- 5 栏 5pt 实测容量约 8,700 字/页；mermaid 11.4 随包 2.6MB，只在含 mermaid 的卡上注入；
- 0.5.92 已装进 web profile。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
