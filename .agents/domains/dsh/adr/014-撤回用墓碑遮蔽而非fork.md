---
name: "014-撤回用墓碑遮蔽而非fork"
description: "撤回改为同一会话内追加一条墓碑事件遮蔽切点之后的表面，界面用 visibility 隐藏；不再 fork 新会话、归档并删除原日志。"
status: accepted
---

# 014-撤回用墓碑遮蔽而非fork

## 背景

撤回原先走 `session/fork` + `archiveSession({stopActivity:true})` + 物理删除原会话日志。代价是：会话 id 变了（Plan 绑定与 note 板随之丢失）、归档广播 `workspace/session-stop` 杀掉子 agent 与后台 job、删日志不可逆。

## 决策

在**同一会话**里追加一条 `system/message` 墓碑：空 content、`data.recall{boundary,end}`、`surfaceOp:{op:"replace",startSeq:boundary,endSeq:end}`、`sourceEventSeqs` 覆盖全部被遮蔽节点，然后 `sessions.flush`。界面由客户端把区间内的视图节点置 `visibility:"hidden"`，用户提问行由同 key 的 seat 接管后不渲染。撤回前先清空该会话的排队输入。

## 理由

- 表面是模型看到的历史，`replace` 是宿主唯一能让它变短的手段；日志保持 append-only，重启可重放。
- 会话对象不重建，所以 Plan 绑定、置顶位、子 agent 与 job 全部原样存活。
- `visibility` 是宿主自己的过滤契约（`isVisibleChatNode`），不是猜 DOM。

## 后果

- 撤回后 turn 号不回退、消息编号不回退、`<plan>/log.jsonl` 里被撤回那轮的行不删。
- **撤回不杀活体**：后台 job 与子 agent 继续跑（旧实现靠归档广播 `workspace/session-stop` 连它们一起杀）。撤不回的是副作用，不是这些进程。它们的完成通知走续跑通道（`dsh-subagent/src/continuation-messages.ts` 注入的 user 消息），不是 `queue`，所以撤回清队列清不到它们 —— 通知会落在已被遮蔽的历史上，模型看到「job 完成了」却看不到当初为什么起它。只有「正在跑一轮」会被 409 `agent-busy` 挡住。
- 不写 `compaction/prune`，所以上下文占用会在下一次真实压缩前虚高。
- 界面隐藏只剩一条未公开行为：改写其它 definition 的 `buildViewNode`（不抢 `conversation.chat.node` 的 seat —— 用户行同样由定义层的 `visibility` 覆盖，抢座位反而会掉宿主 user 行的装饰链）。命中依赖「撤回按钮至少渲染过一次」提供当前会话 id，区间按会话隔离。
- 旧删除式撤回留下的 `hiddenSessionIds` / `pendingDeletes` 只保留为启动自愈路径，新撤回不再写。
- 附件不回填，只回填文本。

## 相关

- [007-对照dsh-recall的撤回实现](../plans/active/2026-11-16-轮转保持原会话/007-对照dsh-recall的撤回实现.md) — 这套手法的第三方原型与它踩的宿主内部依赖
- [006-宿主不能原地删除或截断会话](../questions/006-宿主不能原地删除或截断会话.md) — 为什么"日志不能截断"仍然能就地撤回
