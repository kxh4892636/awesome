---
name: '008-用bsk操作GitHub时为什么会失败'
description: 'settings / releases 页偶发 ERR_CONNECTION_CLOSED，重开 session 即恢复；Release 资产上传被失效的控制上下文拦住，改用提交进仓的 tgz。'
---

# 008-用bsk操作GitHub时为什么会失败

## 适用条件与现象

用 `bsk`（browser-skill）在已登录的浏览器里操作 GitHub：建仓、改可见性、发 Release。

1. 打开 `github.com/<owner>/<repo>/settings` 或 `.../releases/new` 渲染不出来，控制台 `ERR_CONNECTION_CLOSED`；同一浏览器打开仓首页与 `/new` 建仓页正常。
2. Release 表单能填到「tag 已建」，但上传资产反复报 `the control context no longer belongs to the current document or control state`；ref / selector / observation 三种传法都失败。
3. `bsk` 有新版本时把提示写到 stderr，PowerShell 当 `NativeCommandError` 处理，脚本因此 `exit 1`，其实操作已完成。

## 原因

1. settings 页首次加载的网络失败，不是操作语法问题。
2. Release 上传会导航并替换文档，先前取的 observation 随文档失效。
3. 版本提示走 stderr，被 `$ErrorActionPreference` 当成错误。

## 解决方法

1. 重开 session（或重新 `bsk cooperate sync`）后再开 settings 页，多数一次即成功；每个动作前都要 `sync` 取新的 `context_id`，导航还要它返回的顶层 observation。改可见性五步：Change visibility → Change to public/private → 勾确认句 → Make … → 确认。
2. 不走 Release 资产上传：把 `pnpm pack` 的 tgz 提交进仓（[ADR 003](../adr/003-跨仓制品走仓内tgz与raw链接.md)），消费方用 raw URL。
3. bsk 的版本提示不算失败，别用 `$ErrorActionPreference = 'Stop'` 卡它。

## 验证

- settings 页能渲染并点到 Danger Zone 即已恢复。
- 可见性是否生效，看匿名访问 `https://github.com/<owner>/<repo>` 是否返回 200。

## 来源

2026-11-14 拆分单仓为多仓库 note 004 / 006（bsk 0.4.2，GitHub 网页）。

## 相关

- [007-私有仓的tgz制品为什么装不上](007-私有仓的tgz制品为什么装不上.md) — 同一处的下载侧结论