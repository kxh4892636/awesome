---
name: "009-轮转复用宿主compact"
description: "轮转不再自定义表面遮蔽，改为触发宿主 compact 引擎（不暴露 /compact 命令），压缩前要求 agent 更新 Plan 文档、压缩后读文档并跑 nano_flow_check。"
status: completed
related_noted: ["003-实现原会话内重置轮转", "008-移除撤回功能"]
---

# 009-轮转复用宿主compact

## 需求（2026-11-17）

1. 轮转不再自己维护上下文（不再拼 seed、不再手写 `compaction/prune` + `surfaceOp.replace`），改为**调用宿主自带的 compact 机制**。
2. `/compact` 命令**不对外暴露**，压缩只能由轮转触发。
3. 压缩**前**插一步固定动作：要求 agent 更新当前 Plan 的 `spec.md` 与 note。
4. 压缩**后**插一步固定动作：读取 `spec.md` 与 `notes_index.md`，然后执行 `nano_flow_check`。

## 预设侧事实（preset-probe，2026-11-17）

- 预设定义在 `src/nano-flow/assembly/preset.ts`（运行期由根插件 `agentPresets.register`）；挂 compaction **必须**在 `cordis:group` + `isolate` 内，否则挂载直接失败（`dsh-agent-preset-registry/src/mount.ts`）。
- 宿主分层：`dsh-base` 挂了 `token-meter`/`compaction-basic`/`command-compact`；`dsh-web-app` 的 patch 把后两个 `disabled: true`，改由各预设在 group 内挂。nano-flow 预设原没有这个 group，所以既没有 `ctx.compaction` 也没有 `/compact`。
- 取引擎的路：`agents.get(sessionId)` 拿 agent（session 与 agent 同 id），再 `agent.ctx.get('compaction')` 沿父链解析到预设作用域 —— 后来证明这条在 isolate 下不通，见下文「真机测试结果」的根因段。
- **会拦住用户消息**：gateway 调用 `controller.prompt` 时 await 它，不返回那条用户消息就进不了 inbox（客户端契约允许准入阶段扣住，未找到 prompt RPC 超时）。所以「先跑两轮再放行」在协议上可行，代价是用户看不到任何回执。
- `prompt()` 准入即返回，没有「某条消息跑完」的 promise；能用的只有 `whenIdle()`（宿主提醒它可能跨多条输入）、`agent/status`、`inbox/claimed` + 该 turn 的 `turn/end`。

## 引擎侧事实（compact-probe，2026-11-17）

- `compactNow(agent, signal, commandId?)` 需要**真 Agent**（内部走 `agent.runMaintenance`），只给 Session 不够；`ctx.agents.get(sessionId)` 就返回该 agent（session 与 agent 同 id）。
- 硬前置：agent 的 **phase 恰为 `idle`**（不是 `status==='idle'`，maintenance 期间 status 也是 idle），且会话里**没有开着的 turn**、没有未闭合的 `compaction/start`；否则 `ManualCompactionError('busy')`。
- **手动路径只保留最后一个 surface 节点**：`selectCompactableRange(session, measure, 0)` 里 retainTokens 硬编码 0（`dsh-compaction-basic/src/index.ts:418-422`），`retainRatio` 只影响自动路径。压缩后上下文 = system + checkpoint(摘要) + 最后一个节点。
- 写的事件：`compaction/start{turn:null}` → `compaction/summary{summary, shadowedRange/Seqs/TokenCount, provider, model, usage}` → 替换用的 `user/message`（`source.kind='compact-checkpoint'`，内容包在 `<compacted-summary>` 里）→ `compaction/end{turn:null}`；结束前 `sessions.flush`。**不写 `compaction/prune`**。
- 摘要是**一次真实的模型请求**（回放会话前缀 + 固定指令，`purpose:'compaction'`，`maxTokens` 默认 65536），usage 记进 `compaction/summary.usage`；路由取「显式配置 → 会话最近一次 header → agent.options」。
- 失败码封闭枚举：`busy | cancelled | changed | summary | commit | persistence`；失败会写一条带 `error` 的 `compaction/end` 释放锁。
- **没有「压缩前/后」钩子**。可用插点：`agent/pre-step`/`agent/request`（引擎自己用它们做自动压缩）、`compaction/summary-error`（waterfall）、`session/event`（post-commit、fire-and-forget）、手动路径结束后的 `sessions.flush`。
- 自动压缩属于引擎自身（`compaction-basic` 的 `config.auto` 默认 true，注册在 `agent/pre-step` + `agent/request` + `agent/request-error`）。**只挂引擎不挂命令，自动压缩照旧会触发** —— 要做到「只有轮转能触发」，必须同时把 `auto: false`。
- 既有「引擎在、命令不在」的样例：`dsh-sdk-minimal/cordis.patch.yml:61-68`。

## 用户裁决（2026-11-17）

1. **同意 `auto: false`**：宿主自动压力压缩也关掉，压缩只由轮转触发。
2. **接受代价**：每次触发多 3 次模型请求（2 轮 agent 动作 + 1 次摘要生成），用户消息在 gateway 被扣住期间无回执；pre-step 轮跑着时用户再发消息会以 `steer` 注入那一轮。

执行段交给 `rotate-on-compact`：重写 `rotate.ts` 为四步串行、预设挂 `compaction-basic`（isolate group + auto:false，不挂 `command-compact`）、删掉 seed/prune 全套。

## 结果（2026-11-17，0.5.61）

提交 `6530405`（含版本号），已装 web profile。`rotate.ts` 486 → 374 行，测试 276 条、`vp check` 0/0、`pnpm build` 通过。

| 块 | 内容 |
| --- | --- |
| 四步 | `runInjectedTurn(PRE)` → `compactNow` → `runInjectedTurn(POST)` → 放行用户消息；注入走包装前的 `original`，不进闸门、不改用户原文 |
| 等待 | 新增 `waitForTurnEnd` seam（`index.ts` 用 `session/event` 实现），带**注入前最大轮号下限**与 10 分钟预算；不用 `agent.status`、不用 `whenIdle()` |
| 预设 | `cordis:group` + `isolate:{compaction:true}` 内挂 `compaction-basic`（`auto:false`），不挂 `command-compact`；`dist` 里 `command-compact` 0 命中 |
| 删掉 | `clearSurface`、`buildSeed`/`SeedParts`、`lastAssistantText`/`truncateTail`、`boundPlan`/`autoCheck`/`workingDirectory`/`domainRoot`、`attachRotation` |
| 失败策略 | 第 1/3 步抛错或超时只 warn；第 2 步含 `busy` 只 warn；第 4 步在任何分支都执行 |

遗留风险（写进 CONTEXT）：等待只能认「注入后第一个 `turn/end`」，若期间有别的轮次先结束会提前开跑压缩 → `busy` → 只 warn（自愈但难查）；`busyEnter: steer` 会把用户消息注入插入的那两轮；用户消息被扣期间无回执，客户端中断会让它不入 inbox。

## 真机测试结果（2026-11-17 18:05，宿主 17:49 重启，0.5.62，rotateAt=0.15）

**没触发。** 证据：`nano-flow-mount.json` 的 `rotation` 段四个 seam 全 true（`tokenMeter/agents/sessionController/waitForTurnEnd`）；会话里 `turn/start` 40 条、`turn/end` 39 条（唯一未闭合的是当前这一轮）；上一条请求 `totalTokens` ≈ 396k ≫ 150k 阈值；但会话日志里**既没有任何 `轮转前检查点` / `轮转后恢复` 注入，也没有新的 `compaction/*` 事件**。

**根因：isolate realm。** 预设里 `compaction-basic` 必须挂在 `cordis:group` + `isolate:{compaction:true}` 内，而 isolate 会把服务发布在**realm 私有 symbol** 下（`dsh-agent-preset-registry/src/mount.ts:60-86` 的 `leakedServices` 注释写明：isolate 内的 provider 在根 store 里"correctly absent"）。agent 的 ctx 绑在**预设作用域**（group 的父级），所以 `agent.ctx.get('compaction')` 解析不到引擎 → `targetOf` 返回 undefined → 0.5.62 加的那条 warn 触发（打到宿主 stderr，我们没有日志文件可读）。宿主自己的 `command-compact` 也是挂在**同一个 group 内**才看得到引擎，印证了"消费者必须在同一 realm"。

**修法（待做）**：同一 bundle 被挂两次共享模块级状态 —— 在 compaction group 内加一个本包的小 row（`compactBridgeOnly`），它 `inject:['compaction']` 后把 `(agent, signal) => ctx.compaction.compactNow(agent, signal)` 注册进模块级槽；根半的轮转从槽里取用。服务不跨 realm，但同一模块的变量跨 realm。

## 桥接修复（2026-11-17，0.5.63）

提交 `501e672` + `25cb215`，284 测试 / `vp check` 0-0 / build 通过，已装 web profile。

- 新增 `src/nano-flow/assembly/compaction-bridge.ts`：模块级槽 `setCompactionBridge/clearCompactionBridge/getCompactionBridge`，登记时返回撤销函数且**只在槽里仍是自己时清空**（preset 按会话各挂一次，先卸载的不该抹掉后注册的）。
- 预设的 compaction group 内加第二行 `nano-flow-compaction-bridge`（`name: entryUrl`，与顶层 `nano-flow` 行同一个模块文件），`inject:['compaction']` 后把 `(agent, signal) => engine.compactNow(agent, signal)` 登记进槽。
- `rotate.ts` 的 `targetOf` 改从槽里取桥，仍用 `agents.get(sessionId)` 取 agent 当参数；无桥时 warn 文案改为 `exposes no compaction bridge`。
- `mount-report` 新增 `bridge` stage（`{requested, engine}`），真机验证先看它。

## 真机验证（2026-11-17 20:30，重启后首次触发）

四步在会话日志 `session-e09f821c…/session.v4.jsonl.zstd` 里逐条对得上（seq 为日志序号）：

| 步 | 证据 | 结论 |
| --- | --- | --- |
| ① 压缩前检查点 | turn 42 的输入是「轮转前检查点…」，seq 3884 `turn/end` | 桥接生效、三道 seam 可用、0.15 阈值越线 |
| ② 压缩 | seq 3885 `compaction/start{turn:null}` → 3886 `compaction/summary` → 3887 `user/message{source.kind:compact-checkpoint}` → 3888 `compaction/end` | 宿主真的压了，摘要即 `<compacted-summary>` 全文 |
| ③ 压缩后恢复 | seq 3890 `turn/start{turn:43}`，输入是「轮转后恢复…」；本轮已读 `spec.md`+`notes_index.md`、`nano_flow_check` 通过 | 注入轮次照常跑，文档已回填 |
| ④ 放行用户消息 | seq 4070 `turn/end`(43, 20:33:13) → 4072 `turn/start`(44) 同一秒内接上，用户原话「我重启了」以新 rpcId 落进 turn 44 | 放行正常，共扣住 2 分 17 秒 |

- 「①结束 → ②开始」间隔 23 ms，说明等待用的是同一回合的 `turn/end`，没有被别的轮次抢先。
- 压缩触发时机是 ① 那轮结束之后、③ 开始之前，符合设计（压缩必须 phase=idle 且无开着的 turn）。
- 摘要覆盖率：压缩前约 396k tokens 的历史被压成一段摘要 + 最后一个 surface 节点，③ 的输入因此是「系统提示 + 摘要 + 那段注入原文」——这就是为什么 ③ 必须自己重读 `spec.md` 与 `notes_index.md`。

**压缩幅度（`assistant/message.usage.totalTokens`，每轮首个请求）：**

| 轮 | 时刻 | totalTokens |
| --- | --- | --- |
| 42（压缩前检查点） | 20:31:08 | 416,569 |
| 43（压缩后恢复） | 20:31:40 | 26,161 |
| 44（放行的用户原话） | 20:33:22 | 55,212 |

## 0.5.64：压缩前提示词精简（用户手改）

用户把 ① 那一轮的正文从「更新…：进度、证据、下一步，然后结束这一轮」删成「更新…，然后结束这一轮」。随之把 `rotate.test.ts` 里 pin「下一步」的断言删掉（提示词只该测「提到哪些文件/动作」）。门禁 284 测试全过，已装 web profile。

**这条路径对重启后的会话同样成立**：宿主重启后会话从日志重放，历史照旧计入压力，所以重启后的第一条消息就会触发轮转 —— 这次触发正是「我重启了」这句话本身。用户侧观感是这句话 2 分 17 秒后才出现（20:30:56 发出、20:33:13 落进 turn 44），期间 GUI 无任何回执。

## 边界

- 先给方案与代价，动手前等用户放行。
- 不改 pi 侧。
