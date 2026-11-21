---
name: "007-导图退出站点改用mermaid主题"
description: "2026-11-18 站点撤下自研 mindmap 插件、删掉两篇 MINDMAP.md；次日启用官方 mermaid 主题。插件仓保留 0.1.0，ADR 001–006 从此只描述插件、不描述站点。"
status: accepted
---

# 007-导图退出站点改用mermaid主题

## 背景

2026-11-13 到 11-17，站点用自研 `@kxh4892636/docusaurus-plugin-mindmap` 把正文里 `<Mindmap>` 标签内的标题渲染成图（形态见 [001](001-导图是文档内渲染块而非独立路由.md)、[005](005-导图自绘SVG零新依赖.md)）。

2026-11-18 20:58（`repos/wiki` `6b2d31b`「移除 mindmap 插件与 React/Go 导图」）一次提交把插件从站点摘掉：`package.json` 去掉依赖、`docusaurus.config.ts` 去掉 remark 入口、删掉 `src/theme/MDXComponents.tsx` 里 `MindmapBlock` 的注册，并删掉两篇导图文档 `docs/020-计算机/020-编程语言/040-go/MINDMAP.md` 与 `docs/020-计算机/050-客户端/010-react/MINDMAP.md`。

次日 21:29（`042ebb2`）启用官方 `@docusaurus/theme-mermaid`，Docusaurus 3.10.1 升到 3.10.2。

## 决策

站点不再挂 mindmap 插件，图示改走官方 mermaid：正文写 mermaid 代码围栏（三反引号后接 mermaid），由 `@docusaurus/theme-mermaid` 渲染，`docusaurus.config.ts` 里 `markdown.mermaid: true`。

插件仓 `repos/docusaurus-plugin-mindmap` 保留在 0.1.0 不动、不删：它是 ADR 001–006 的实现，可用可装，只是不在站点里。

## 依据（2026-11-19 实测）

- 现行配置：`repos/wiki/docusaurus.config.ts` 有 `markdown.mermaid: true` 与 `themes: ["@docusaurus/theme-mermaid", …]`；`plugins` 只剩 `@kxh4892636/docusaurus-plugin-link`；`src/theme/MDXComponents.tsx` 已不存在，站点无组件接收 `<Mindmap>`。
- 现行内容：tracked 文档 1512 篇里 `<Mindmap>` 块 **0 个**；唯一带标签的是本地 `docs/000-中转/思维导图示例.md`，被 `.gitignore` 的 `*/*中转*` 排除、不发布。已用 mermaid 的是 `docs/020-计算机/010-理论基础/010-计算机网络/004-TCP拥塞控制.md`（mermaid 代码块）。
- 同一波决策的邻仓证据：`repos/dsh-toolkit` `1f930d4`（2026-11-19）「移除导图笔记支持」、`384edd8`「状态流转与时序图改用 mermaid」。
- 撤下的理由没有写进本工作区任何 Plan 或轨迹：`6b2d31b` / `042ebb2` 都没有提交正文，wiki 域最新轨迹停在 2026-11-13，其他域文档也不提这次迁移。本页只固定可核对的事实，理由等用户补。

## 后果

- ADR 001–006、[workflow 001](../workflows/001-在wiki文档里写一个导图块并验收/001-在wiki文档里写一个导图块并验收.md)、[question 001](../questions/001-MDX把doc后缀解析成textDirective.md) / [003](../questions/003-doc引用写错为什么build不报错.md) / [004](../questions/004-导图样式为什么以字符串注入.md) 描述的是插件 0.1.0 的行为，不再是站点现状。引用它们之前先看本页。
- 只有 mindmap 插件具备的能力一并消失：标题树自动成图与布局、节点浮窗、`doc:<id>` 跨文档引用边、图内节点锚点（`#1.2`）。mermaid 是手写图语法，结构不随文档标题变化。
- 同时省掉三处维护面：remark 顺序的隐性依赖（[002](002-导图解析挂在默认remark插件之前.md)）、两份样式副本（question 004）、`doc:<id>` 静默 404 的逐条点验（question 003）。
- 站点升级只需跟官方 mermaid 主题，不再跟自研插件的 Docusaurus 主题接口。
- [005-导图自绘SVG零新依赖](005-导图自绘SVG零新依赖.md) 里「不引 mermaid」的判断只对插件仓成立，站点侧已经反过来。

## 相关

- [001-导图是文档内渲染块而非独立路由](001-导图是文档内渲染块而非独立路由.md) — 被本页取代的站点形态，插件仓里仍然成立
- [005-导图自绘SVG零新依赖](005-导图自绘SVG零新依赖.md) — 零依赖的判断对插件仓有效，站点侧已改用 mermaid
- [001-在wiki文档里写一个导图块并验收](../workflows/001-在wiki文档里写一个导图块并验收/001-在wiki文档里写一个导图块并验收.md) — 该流程的对象已从站点撤下，只剩插件仓可照做
- [2026-11-13-思维导图插件](../plans/reference/2026-11-13-文档内思维导图插件/spec.md) — 插件从立项到撤下站点的完整轨迹
