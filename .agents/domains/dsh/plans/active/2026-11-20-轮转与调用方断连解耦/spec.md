---
name: "轮转与调用方断连解耦"
description: "nano-flow 轮转不再复用用户 prompt RPC 的 AbortSignal，客户端断连不再中断压缩或丢消息。"
status: completed
---

# 轮转与调用方断连解耦

## 背景

2026-11-20 session-49a3964c 两次轮转都停在「轮转前检查点」那一轮之后：没有 `compaction/*`、没有「轮转后恢复」注入、用户原消息也没进日志。取证结论（本会话 /ap）：

- 轮转四步（检查点轮 → 压缩 → 恢复轮 → 放行用户消息）复用了用户那条 prompt RPC 带来的 `AbortSignal`（`repos/dsh-toolkit/src/nano-flow/session/rotate.ts` 的 `redirect`）。
- 浏览器拆掉那条请求（切会话 / 刷新 / 断连）后 signal 被 abort：等待被 reject → 压缩被跳过；恢复轮注入与 `fallback()` 都在 `SessionController.prompt` 首行 `signal.throwIfAborted()` 抛错，用户消息从未入库。
- 三步失败只写 `logger.warn`（宿主 stderr），会话日志无痕，看起来「静默」。

## 目标

轮转一旦开始，就与调用方的断连解耦：客户端走了，压缩照做、用户消息照样放行。不改调用方语义（RPC 仍可被取消，只是取消不再撤销轮转与已扣住的消息）。

## 方案要点

- `redirect` 进入轮转分支后新建一个不会 abort 的 `AbortSignal`（`new AbortController().signal`），后续检查点轮、`compactNow`、恢复轮与 `fallback` 全部用它；调用方 `signal` 只用于「是否触发轮转」之前的路径。
- `runInjectedTurn` 内已有的 `gate`（注入失败时 abort 取消等待）保留；`AbortSignal.any` 不变。
- 超时预算不变（`TURN_WAIT_TIMEOUT_MS`），不新增配置项、不新增抽象。

## 验收

1. 单测：检查点轮进行中 abort 调用方 signal，压缩桥仍被调用、恢复轮仍被注入、用户原 request 仍被放行。
2. 既有 `rotate.test.ts` 全绿；`vp check` 与 `vp test src` 全绿。

## 影响面

- `repos/dsh-toolkit/src/nano-flow/session/rotate.ts`
- `repos/dsh-toolkit/src/nano-flow/session/rotate.test.ts`

## 相关

- [012-源码按能力分层](../adr/012-源码按能力分层.md) — 轮转在 session 能力目录内

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
