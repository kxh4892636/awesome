---
name: "002-DocxXML版式与文风"
description: "提供可直接复制的 DocxXML 版式配方与直白文风规则。"
status: completed
related_noted: []
next_step:
---

# 002-DocxXML版式与文风

## 交付

- `reference/docx-xml.md`：标题、TLDR、章节引导、双栏、表格、状态 callout、画板、段落与结尾的可复制片段。
- 配色语义表与硬约束：一级标题默认黑色、callout 成对配色、grid 比例合计 1、表格列宽合计约 1000、callout 子块白名单。
- 反例：Markdown 语法、callout 塞表格、grid 用错属性、只写一个颜色。

## 验收

- 用配方写的示例文档通过 `lark-cli docs +script --command parse`，`assessment.status = passed`。
- TLDR、双栏 callout、表格在飞书里渲染正常（示例文档截图）。
- 文风规则覆盖：单段最多 144 字、最好 89 字以内；结论先行、数字优先、结尾给动作。

## 证据

- 示例：`lark-cli docs +create` 创建的两篇验证文档（doc2 与最终 demo）均无 warning。
- 解析：demo 的 `parse` 输出 `passed`，`char_count` 808。