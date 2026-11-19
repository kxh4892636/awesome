---
name: "面板恢复默认"
description: "toolkit 面板的 Nano Flow 设置支持整组与单字段恢复默认值。"
status: in_progress
---

# 面板恢复默认

## 背景

toolkit 侧边栏面板的「Nano Flow 设置」有 9 个数值（spec / note / 描述字数 / CONTEXT / question / ADR / workflow / GUIDE 的篇幅上限与轮转阈值），只提供「保存」。改乱后没有回到出厂值的入口，用户只能逐个查默认值手输。

## 目标

- 设置区操作行加一个「恢复默认」按钮：把 9 个字段全部填回 `DEFAULT_NANO_FLOW_SETTINGS`。
- 每个输入框内部右侧加一个恢复图标：只把那一个字段填回默认值。
- 两者都只改表单草稿，仍由「保存」落盘（用户 2026-11-19 选定）。

## 方案要点

- 面板侧是纯客户端改动：默认值由 `contract.ts` 的 `DEFAULT_NANO_FLOW_SETTINGS` 提供，面板按现有 `toDisplay` 换算成界面文本。
- 2026-11-19 用户指定把默认值改为 description 144 字、GUIDE 21 KB、轮转阈值 0.55（原 233 字 / 13 KB / 0.618），同步改 `contract.ts` 与随包校验脚本的 `DEFAULT_LIMITS`；只改默认值，不碰用户已保存的 `nano-flow.json`。
- 单个图标用宿主 `ui-primitives` 的 `IconRefreshOutlineRegular`（宿主已导出），不自绘 SVG；输入框外层是宿主 `Input` 的 `span`，图标绝对定位贴在右侧，内层 input 让出右内边距。
- 不改保存流程：两个动作都只 `setDraft`，清掉上一条错误提示，二次保存才写 `$DSH_HOME/toolkit/nano-flow.json`。

## 验收

1. 打开 Nano Flow 设置，点「恢复默认」：9 个输入框回到 34 / 13 / 144 / 89 / 13 / 8 / 34 / 21 KB 与 0.55，未点「保存」前不落盘。
2. 改乱一个字段后点该框内的恢复图标：只有这个框回到默认值，其他草稿保持用户输入。
3. `vp check` 与 `vp test src` 全绿；面板单测覆盖整组与单字段两条路径。

## 影响面

- 面板：`repos/dsh-toolkit/src/client/panel.ts`、`stylesheet.ts`、`primitives.d.ts`、`test-primitives.ts`、`panel.test.ts`。
- 默认值：`src/contract.ts`、`src/nano-flow/documents/lib/base.mjs`，以及断言默认值的四个测试文件。
- 打包发版顺序照 [dsh 001 工作流](../../workflows/001-发布dsh-toolkit并真机验收/001-发布dsh-toolkit并真机验收.md)。

## 相关

- [001-面板恢复默认与单字段重置](001-面板恢复默认与单字段重置.md) — 本 Plan 唯一交付单元
- [012-源码按能力分层](../../../adr/012-源码按能力分层.md) — 客户端半改动的位置约定

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
| [001-面板恢复默认与单字段重置](001-面板恢复默认与单字段重置.md) | 设置区加「恢复默认」按钮与每框恢复图标；默认值改为 144 字、GUIDE 21 KB、轮转 0.55。 | blocked | 0.5.74 已装进 web profile，宿主仍跑旧 bundle；重启 dsh web 后才能真机验收。 | 重启 web profile 宿主，打开 toolkit 面板点「恢复默认」与框内图标验收 | — |
