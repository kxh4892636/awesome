---
name: "笔记与导图制作流程"
description: "知识笔记制作流程与交付记录：React / Go / TCP 笔记、Anki 卡片、Mermaid 支持与 learning skill 规则。"
status: in_progress
---

# 笔记与导图制作流程

## 目标

当前以**知识笔记**为主：从八股题、归档资料与联网检索整理可复习的 Markdown 笔记，配 Anki 卡片；笔记要能被人工阅读，也能被 agent 按同一规则继续生产。

导图分支已下线：wiki 移除 mindmap 插件，learning skill 删除 MINDMAP.md；本 Plan 后续只维护知识笔记与配套工具。

## 现状（2026-11-20）

| 项 | 事实 |
| --- | --- |
| wiki 仓库 | `repos/wiki`，main，最新 `a0fcaae`，工作区干净 |
| dsh-toolkit | `repos/dsh-toolkit`，main，最新 `142acf9`（0.5.78），工作区干净 |
| React 笔记 | `docs/020-计算机/050-客户端/010-react`：7 篇知识笔记 + 1 个分类占位页 |
| Go 笔记 | `docs/020-计算机/020-编程语言/040-go`：7 篇 |
| TCP 笔记 | `docs/020-计算机/010-理论基础/010-计算机网络`：6 篇，覆盖八股 TCP 组 21 题 |
| Anki | React 牌组 7 张、Go 牌组 7 张、TCP 牌组 6 张；均单一根牌组 |
| Mermaid | Docusaurus 升至 3.10.2，启用 `@docusaurus/theme-mermaid`；`theme-common` 只剩 1 个实例 |
| learning skill | 仅知识笔记；NOTE 规定「常见问题 → 概念索引 → 正文」；SKILL 允许 mermaid 状态/时序/流程图 |
| dev | `pnpm start --port 3000 --host 127.0.0.1`，根页与笔记页返回 200 |

## 决策

- **知识笔记优先**：一个领域一个目录，知识笔记平铺；不再生产导图笔记。
- **常见问题取原文**：第一个 H2 固定为「常见问题」，直接列八股原文，不改写；没有八股来源才列实际疑问。
- **富文本有边界**：能代码演示的优先代码；一对一对应用列表；表格只放多维对比或演算；状态流转、时序、流程用 mermaid。
- **不设参考章节**：来源在正文内联链接，避免文末堆链接。
- **Anki 一 note 一卡**：正面 `主题-章节-问题`，背面 wiki UUID 短链；一个主题一个根牌组，不建子牌组。
- **Mermaid 依赖单实例**：所有 `@docusaurus/*` 同版本，防止 theme-common 被 peer 拆成多个上下文。

## 交付

1. **[001-知识笔记交付记录](001-知识笔记交付记录.md)**：本轮 React / Go / TCP 笔记、Anki 卡片、Mermaid 与 skill 规则的交付清单和验收证据。
2. **[002-清理导图遗留文档](002-清理导图遗留文档.md)**：把已下线的导图 workflow、ADR 与 CONTEXT 词条改写或归档，避免新会话误读。

## 非范围

- 不再新增或维护 `<Mindmap>` 导图、mindmap 插件与 MINDMAP.md。
- 不批量改造既有 1495 篇归档笔记的标题层级或补 frontmatter id。
- 不做在线编辑器、自动抓取整站或自动生成全文。
- 不覆盖 `020-八股文梳理` 的快照生成脚本。

## 风险与假设

- **Anki 依赖隧道**：AnkiConnect 在用户本机，远端要能访问 `127.0.0.1:8765` 才能建卡；隧道断开时只能准备参数，不能写入。
- **Mermaid 客户端渲染**：语法错误不会让 build 失败，页面才报错；新增图要用浏览器或解析器验证。
- **导图遗留文档**：`wiki workflow 001`、ADR 001–006 仍描述导图，已不适用于新流程，后续由 dream 或单独清理。
- **规则双份**：learning skill 的 NOTE/SKILL 与 wiki 实际笔记可能漂移，改规则时以 skill 为源，再回写示例。

## 恢复入口

1. 先读 [001-知识笔记交付记录](001-知识笔记交付记录.md) 了解本轮产出与证据。
2. 规则源：`repos/dsh-toolkit/src/nano-flow/skills/learning/{SKILL,NOTE,ANKI}.md`。
3. 笔记目录：React / Go / TCP 三处，见「现状」表。
4. Anki 连接：先测 `curl -X POST http://127.0.0.1:8765 -d '{"action":"version","version":6}'`，失败先修隧道，不要猜成功。

## 进度

- 2026-11-16：克隆 `repos/wiki`，建立 Plan；删除预设 note，改为按用户输入创建。
- 2026-11-17：删除 learning skill 的 RULE.md；统一 NOTE/MINDMAP 规则；补充 note-metrics。
- 2026-11-18：新增 React 7 篇 + Go 7 篇知识笔记；创建 React/Go Anki 卡片；删除已迁移的八股题。
- 2026-11-19：TCP 6 篇笔记覆盖 21 题；删除 MINDMAP.md 与导图支持；wiki 移除 mindmap 插件；接入 Mermaid；Docusaurus 统一 3.10.2，修复 ColorModeProvider 错误；TCP 笔记移至 `010-理论基础` 并同步 Anki 牌组。
- 2026-11-20：更新本 spec 与交付记录，准备轮转。
- 2026-11-20：轮转前复检；spec 与两条 note 已就位，`nano_flow_check` 通过；Plan 文档尚未提交。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
| [002-清理导图遗留文档](002-清理导图遗留文档.md) | wiki workflow 001 与 ADR 001–006 仍描述已下线的导图，需改写或归档，避免新会话误读。 | pending | — | 确认 workflow 001 与 ADR 001–006 是改写还是归档，再更新索引 | — |
