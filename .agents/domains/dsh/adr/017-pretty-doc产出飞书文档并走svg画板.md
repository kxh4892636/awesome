---
name: "017-pretty-doc产出飞书文档并走svg画板"
description: "手调 pretty-doc 只产出飞书文档：DocxXML 承载版式，diagram-design 的 SVG 经 larkify 内联后进画板；配色改用 Lark 色板。"
status: accepted
---

# 017-pretty-doc 产出飞书文档并走 SVG 画板

## 背景

要给 dsh 加一个手调 skill：用户给素材，产出排版美观、语言直白的文档。可选形态有三类：Markdown + SVG、独立 HTML、飞书文档。前两类便于本地保存，但 callout / 分栏 / 表格等版式要么没有，要么需要另做一套渲染；用户已明确选择飞书文档。

画板也有三条路：Mermaid、diagram-design 的 HTML/SVG、截图。Mermaid 在飞书画板里可用，但 diagram-design 的编辑式布局和几何细节用不上；截图不可编辑。选 diagram-design 的 SVG，但实测飞书画板不执行 SVG 的 `<style>` / class：类样式驱动的图会变成黑块、文字消失（[question 020](questions/020-飞书画板忽略svg类样式.md)）。

## 决策

- 产出飞书文档，正文用 DocxXML（`lark-cli docs +create/+update`），不产出 Markdown / HTML。
- 画板链路固定为：diagram-design 出 HTML → `export_svg.py` 导出 → `scripts/larkify_svg.py` 内联 CSS、映射色板、删不支持特性 → `<whiteboard type="svg">`。
- 配色默认用 style-cookbook 的 Lark 色板：蓝 `#245bdb` 作焦点，白 / 浅灰作底，紫 / 绿 / 橙 / 红只按语义点缀；不沿用 diagram-design 的 paper / ink / coral。
- 只可手调：frontmatter `disable-model-invocation: true`，`agents/openai.yaml` 同步保留给其他宿主。

## 后果

- 运行依赖 lark-cli 的登录与目标空间权限；未登录时停下，不退化成 Markdown。
- 画板会统一成 Noto Sans SC，并删掉 pattern / clipPath / mask / 非阴影 filter；转换脚本是链路的一部分，不是可选清理。
- 交付物是一个飞书链接，不是仓库里的文件；本地只留 `.temp/pretty-doc-<slug>/` 下的草稿与 SVG。
- 回读是验收信号：`docs +fetch` 看结构，白板 `+export` 看文字 / 箭头 / 颜色；预览是方形画布，文档内的实际高度按 SVG 的 viewBox 走。

## 相关

- [001-技能骨架与手调配置](../plans/active/2026-11-24-美观文档技能/001-技能骨架与手调配置.md) — 循环与手调配置
- [003-画板svg链路](../plans/active/2026-11-24-美观文档技能/003-画板svg链路.md) — 转换器与实测
- [020-飞书画板忽略svg类样式](../questions/020-飞书画板忽略svg类样式.md) — 黑块现象与判别