---
name: "002-DocxXML版式与文风"
description: "提供可直接复制的 DocxXML 版式配方与直白文风规则。"
status: completed
related_noted: []
next_step:
---

# 002-DocxXML版式与文风

## 交付

- `reference/docx-xml.md`：标题、TLDR、多观点 callout、章节引导、双栏、表格、状态 callout、画板、段落与结尾的可复制片段。
- 长文层级：h1 分部、h2 分节、h3 分点；h2 可用名词短语或读者问题，不强制问句。
- 配色语义表与硬约束：一、二、三级标题默认黑色、callout 成对配色、grid 比例合计 1、表格列宽合计约 1000、callout 子块白名单。
- 反例：Markdown 语法、callout 塞表格、grid 用错属性、只写一个颜色。

## 验收

- 用配方写的示例文档通过 `lark-cli docs +script --command parse`，`assessment.status = passed`。
- 长文示例同时覆盖：h2 名词短语、h2 问句、h3 子点、多观点 callout；飞书里渲染正常。
- 文风规则覆盖：单段最多 144 字、最好 89 字以内；结论先行、数字优先、结尾给动作。

## 证据

- 长文验证文档：[长文层级与多观点验证](https://bytedance.larkoffice.com/docx/BcM5dVDX6owTtUx9FRoc2hMjnyh)，parse `passed`，char_count 315。
- 主示例：[把素材写成一篇能读的飞书文档](https://bytedance.larkoffice.com/docx/PMWUdwWBzom0wkxfs6wcLXdSn9e)，parse `passed`。
