---
name: "007-对照dsh-recall的撤回实现"
description: "第三方插件 dsh-recall v0.3.0 用「一条 system/message 墓碑 + 客户端 visibility:hidden」在不改宿主的前提下实现就地撤回；本 note 记其机制、依赖的宿主内部行为与移植清单。"
status: completed
related_noted: ["006-撤回在同一会话内完成", "003-实现原会话内重置轮转"]
---

# 007-对照dsh-recall的撤回实现

源码克隆在 `.temp/dsh-recall`（v0.3.0，10 文件 148K，MIT）。下文 `client.js:N` / `index.js:N` 指该仓 `lib/` 下文件。

## 机制：一次写入 + 四处客户端接管，全程不换会话

### 宿主半（`lib/index.js`，320 行）

1. 注册一个 `POST /recall` 路由（`webServer.register`，prefix，其它方法 405），inject `["webServer","sessions","agents"]`，无工具、无 `plugin:` 记录。
2. 取活体 agent → 拒子 agent（`origin==='subagent'` 或 `agents.isOwnedBy`）→ 拒 `status==='running'`（409 `agent-busy`）。
3. **切点**：`{boundary}`（按 seq，必须是 `surfaceOp==='append'` 的 user/assistant 消息）或 `{messageId}`（线性扫日志按 message.id 找 seq）。**不按 turn**。
4. **唯一一次写**（`index.js:247-260`）：

```js
session.append("system/message", {
  turn, step, recall: { boundary, end },
  message: { id: `recall-${seq}-${boundary}`, role: "system",
             source: { kind: "system-prompt" }, content: [] }
}, { surfaceOp: { op: "replace", startSeq: boundary, endSeq: end },
     sourceEventSeqs: shadowed });
```

   - 遮蔽范围 = 表面从该节点**到尾部**（`nodes.slice(startIdx)`），按表面位置切。
   - 选 `system/message` 的理由：它是 surface-eligible 但**空内容不派生消息**（宿主 `surface.ts:138-142`），所以模型侧不会多出一条幽灵消息。
   - 额外字段 `data.recall` 不在事件类型定义里，宿主不校验未知键、v4 持久化保留额外 JSON 字段。
   - turn/step 必须是正整数（v4 准入），缺失时向前找最近的 `turn/start`。
5. `sessions.flush` 后返回 `{boundary, seq}`。**不建会话、不 fork、不归档、不删日志、不碰队列/steering**。

### 客户端半（`lib/client.js`，836 行）

| # | 动作 | 关键行 |
| --- | --- | --- |
| 1 | 注册 `recall` definition（`target:'chat'`）：命中 `system/message` + `data.recall` 时把 `{boundary,end}` 记进模块级区间表，并生成一条「已撤回」通知节点 | `589-614, 565-576` |
| 2 | 抢用户行 seat：`conversation.chat.node` + `key:'user'` + `priority:-1`；区间内 `return null`，否则**委托框架原组件**渲染并叠一个撤回按钮 | `809-819, 478-480, 443-459` |
| 3 | 遍历 `uiConversation.events.entries()`，把其它所有 definition 的 `buildViewNode` 包一层：节点 seq 命中区间 → `{...node, visibility:'hidden'}` | `671-700`、候选 seq `646-654` |
| 4 | 新墓碑到达 → dispose 并重新 `register` 自己的 definition，触发宿主全量重装配 | `721-734` |

- 隐藏靠**宿主公开契约**：`isVisibleChatNode` 要求 `visibility==='visible'`（本机 `dsh-client-ui-chat/src/client/contract/chat-visibility.ts:10-22`，调用点 `chat-snapshot-builder.ts:137,525`、`process-groups.ts:43`）。
- 不能用「`buildViewNode` 返回 null」代替：撤回已物化的 target 会抛错（宿主 `conversation/assembler.ts:1059` 要求改用 hidden）。
- 成功后把被撤回消息的**文本与图片**写回 composer 草稿（`227-321`）；**不**清队列、不跳转、不轮询，只看 HTTP 响应。

## 与我们轮转的对照

| 维度 | 我们的轮转（0.5.56） | dsh-recall v0.3.0 |
| --- | --- | --- |
| 写入 | `compaction/prune` + 一条 `user/message`（seed 正文） | 只一条 `system/message`（空 content + `data.recall`），**不写 prune** |
| 表面操作 | `replace` 覆盖 `nodes[1..]` | `replace` 覆盖切点到尾部 |
| 模型侧 | 看到 seed 文本 | 什么都看不到（空内容不派生消息） |
| 界面侧 | 历史行照旧显示 | 区间内行 `visibility:hidden`，用户行 seat 返回 null |
| tokenMeter | 写 prune，压力记账准确（但数值该用 heuristicTokens） | 不写 prune → 折叠按 0 delta，压力虚高不报错 |
| 会话/活体 | 不换会话、不杀 job | 同 |

## 移植要付的代价（它踩的坑）

1. `agents.isOwnedBy(...)`（`index.js:98`）**在 0.2.1-rc.3 已被删除**；我们照搬会在子 agent 判定处直接炸，得换成 `dsh-api-session-controller/src/agent.ts:98` 的 `hasApiSessionSubagentOwner` 或直接只判 `origin==='subagent'`。
2. 读 `session.log`（TS 私有字段，`dsh-session/src/index.ts:556`）与 `session.surface.deriveEventMessage`（不在 `SessionSurface` 接口上）—— 运行时可读但随时可能没了。
3. `buildViewNode` 被外部改写 —— `dsh/docs/subsystems/` 没有这条缝的说明，属未公开行为。
4. 候选 seq 靠枚举猜测：`anchorSeq` / `data.seq` / `data.finalNode.seq` / `data.closing.finalNode.seq`。
5. `createDrafts` 只传两个参数（宿主签名三个，第三个控制是否上传图片）。

## 移植结果（2026-11-17，0.5.57）

已按本文实现，差异如下：

- 宿主半不用 `session.log` 与 `surface.deriveEventMessage`，改用 `snapshotEvents()` + `surface.nodes`；
- 不做 `isOwnedBy` 归属判定（宿主已删该 API），改为 `agents.get(id).status === 'running'` 时拒 409 `agent-busy`；
- 切点按「最后一轮第一条用户消息」而不是任意消息（保持原对外语义）；
- 补上了参考实现没有的队列清理。

## 我们要做到同样效果，需要补齐

- 宿主半：一条墓碑 + `replace` + `flush`（比轮转更简单：不要 seed、不要 autoCheck、不要 Plan 绑定）；
- 客户端半：上面 4 处接管（我们已有 client 半与 `slots`/`uiWorkspace` 用法，需再加 `uiConversation` 依赖）；
- 队列清空（recall 没做）：`controller.updateQueue({sessionId,itemId,action})` 逐条移除；
- turn 号回退：不做（对照实现也没做，用户已确认不需要轨迹标记）。
