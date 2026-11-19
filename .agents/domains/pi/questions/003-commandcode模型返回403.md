---
name: '003-commandcode模型返回403'
description: '历史记录：选中 claude-* 或 gpt-6-astra 时上游返回 403 MODEL_NOT_IN_PLAN 是套餐权限而非路径错；2026-11-10 起 provider 只上架 DeepSeek，该现场已不可复现。'
---

# 003-commandcode模型返回403

> 纠正（2026-11-19，[note 011](../plans/archived/2026-11-09-审查修复pi-toolkit/011-provider只上架DeepSeek并默认思考.md)，提交 `ab8b11c2`）：2026-11-10 起 provider 只上架 DeepSeek（目录 97 → 7），`/model` 不再列 Claude 与 `gpt-6-astra`，下面的 403 现场无法再复现。保留下来的结论是「网关按账号套餐门禁，403 是权限而不是端点映射或路径错」，以及 400/404 与它的区分方法。

## 适用条件与现象

`commandcode` provider 已登录、`/model` 也列出了 96 条模型，但选中 Claude 全系或 `gpt-6-astra` 时请求失败：

- Anthropic 一侧（`claude-*`）：HTTP 403，信封 `{"type":"error","error":{"type":"permission_error","message":"MODEL_NOT_IN_PLAN: …"}}`。
- OpenAI 一侧：HTTP 403，`{"message":"…","type":"permission_error","code":"FORBIDDEN"}`。

## 原因

模型清单是全量目录，与账号权限无关；这些模型属于上游更高档套餐（Provider 及以上）或需按需加购。当前账号只开通了低档（开源那批）模型。

## 解决方法

- 换到账号已开通的模型（如 `commandcode/deepseek/deepseek-v4-flash`），或在上游升级套餐 / 加购。
- 不要为了这条 403 去改 `api` / `baseUrl` 映射：错误由 Anthropic 信封返回且是 403，说明端点映射是对的。

## 验证

- 区分权限与路径：路径或协议写错会得到 400 `invalid_request_error` 或 404，不是 `permission_error`。
- 可用链路自检：`pi -p --model commandcode/deepseek/deepseek-v4-flash "回复三个字"` 正常返回；带工具调用的 `pi -p --model … "读 repos/pi-toolkit/package.json，只回答 name"` 也正常。
- 目录自检：`pi --list-models | grep -c '^commandcode'` 应为 96（受限模型仍会列出）。

## 来源

Plan `2026-11-09-commandcode-provider接入`，[003-登录凭据与真机验收](../plans/archived/2026-11-09-commandcode-provider接入/003-登录凭据与真机验收.md)；log.jsonl run 6。

## 相关

- [001-取证与方案定案](../plans/archived/2026-11-09-commandcode-provider接入/001-取证与方案定案.md) — 双协议与 baseUrl 不对称的取证，403 的判定建立在这套映射上。
- [003-登录凭据与真机验收](../plans/archived/2026-11-09-commandcode-provider接入/003-登录凭据与真机验收.md) — 403 的逐条实测记录与信封原文。
- [007-commandcode只上架DeepSeek并默认开思考](../adr/007-commandcode只上架DeepSeek并默认开思考.md) — 这条 403 证据促成目录收窄到 DeepSeek，收窄后的形态见该 ADR。
