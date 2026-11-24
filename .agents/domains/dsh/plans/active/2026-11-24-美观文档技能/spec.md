---
name: "美观文档技能"
description: "新增手调 skill pretty-doc：按飞书文档版式组织 DocxXML，画板由 diagram-design 产出 SVG 后嵌入，经 lark-cli 创建并回读验收。"
status: completed
---

# 美观文档技能

## 目标

给 dsh 随包 skill 加一个只可手调的 `pretty-doc`：用户给素材，产出可直接分享的飞书文档。版式参考 lark-pretty-doc 的 style-cookbook，语言参考《AI+全栈：实践、培养及评估指南》。

## 交付

- `repos/dsh-toolkit/src/nano-flow/skills/pretty-doc/`
  - `SKILL.md`：循环（定读者 → 搭骨架 → 写正文 → 配画板 → 创建回读）。
  - `reference/docx-xml.md`：DocxXML 骨架、callout / grid / table 配方、配色与反例。
  - `reference/diagram.md`：diagram-design 出图、Lark 皮肤、SVG 转换、画板回读。
  - `scripts/larkify_svg.py`：把 CSS 类驱动的 SVG 内联成画板可读的属性，并删不支持特性。
  - `assets/lark-palette.json`：diagram-design 默认色 → Lark 色的映射。
  - `agents/openai.yaml`：`allow_implicit_invocation: false`。
- dsh 领域文档：CONTEXT 的随包 skill 清单、README 的 skill 表、ADR 017、question 020。

## 关键决策

- 产出飞书文档，不产出 Markdown / HTML；用户已确认。
- 画板用 SVG，不用 Mermaid；SVG 由 diagram-design 生成，再经 `larkify_svg.py` 转成画板安全格式。
- 配色默认用 style-cookbook 的 Lark 色板（蓝 / 紫 / 绿主色，橙 / 黄 / 红点缀），不沿用 diagram-design 的 paper/ink/coral。
- 只可手调：frontmatter `disable-model-invocation: true`；DSH 只认这个字段，`agents/openai.yaml` 同步保留给其他宿主。

## 验收

- skill 目录被动态发现，且 `disable-model-invocation` 生效。
- 用技能流程创建一篇示例文档，包含 TLDR、双栏 callout、表格、SVG 画板；回读结构与画板预览均正常。
- dsh 单测与 build 通过；领域文档 `nano_flow_check` 无新增问题。

## 事实核查

- 飞书画板不执行 SVG 的 `<style>` / class：diagram-design 的 loop 示例原样嵌入后是黑块、文字消失；内联属性后正常（2026-11-24 实测）。
- 画板保留 marker 箭头；pattern 可渲染，但按 lark-doc-whiteboard 的约束默认删掉，避免降级差异。
- `docs +create` 与 `block_replace` 均能写入 SVG 画板，回读可拿到 board token 并导出预览。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
