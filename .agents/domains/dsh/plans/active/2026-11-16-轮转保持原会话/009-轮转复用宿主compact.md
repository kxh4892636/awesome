---
name: "009-轮转复用宿主compact"
description: "轮转不再自定义表面遮蔽，改为触发宿主 compact 引擎（不暴露 /compact 命令），压缩前要求 agent 更新 Plan 文档、压缩后读文档并跑 nano_flow_check。"
status: completed
related_noted: ["003-实现原会话内重置轮转", "008-移除撤回功能"]
next_step: "subagent 按四步流程重写 rotate.ts + 预设挂 compaction-basic(auto:false)；主会话验收门禁、发 0.5.61、装 web profile、提交"
---

# 009-轮转复用宿主compact

## 需求（2026-11-17）

1. 轮转不再自己维护上下文（不再拼 seed、不再手写 `compaction/prune` + `surfaceOp.replace`），改为**调用宿主自带的 compact 机制**。
2. `/compact` 命令**不对外暴露**，压缩只能由轮转触发。
3. 压缩**前**插一步固定动作：要求 agent 更新当前 Plan 的 `spec.md` 与 note。
4. 压缩**后**插一步固定动作：读取 `spec.md` 与 `notes_index.md`，然后执行 `nano_flow_check`。

## 已核实的宿主分层（决定了"不暴露命令"可行）

- 引擎：`dsh-compaction`（抽象类）+ `dsh-compaction-basic`（实现），API 为 `compactIfNeeded` / `compactNow` / `compactRegion`。
- 命令：独立插件 `dsh-command-compact`；不装它就没有 `/compact`。
- `compactNow(agent, signal, sourceCommandId?)` 要求 **idle agent**，会追加 `compaction/start` → 摘要 → `compaction/end`，保留尾部而不是清空。

## 待查

- `CompactionAgentContext` 长什么样、从 prompt 包装里怎么拿到；compact 的事件与失败语义。
- Nano Flow 预设现在挂哪些插件、在哪个文件里挂；要挂引擎的话改哪里。
- 插件侧能否在 `sessionController.prompt` 包装里"先发起一轮自己的动作并等它结束"，再压缩、再放行用户消息。

## 预设侧事实（preset-probe，2026-11-17）

- 预设定义在 `src/nano-flow/assembly/preset.ts:48-123`（运行期由根插件 `agentPresets.register`），**当前没有任何 compaction 行**，且有 `preset.test.ts:20-26` 断言「名字不含 compaction」。
- 宿主分层：`dsh-base` 挂了 `token-meter`/`compaction-basic`/`command-compact`；`dsh-web-app` 的 patch 把后两个 `disabled: true`，改由**各预设**在 `cordis:group` + `isolate: {compaction, toolResultPruner}` 里挂（`presets/standard.patch.yml:72-88`）。nano-flow 预设没这个 group，所以 nano-flow 会话里既没有 `ctx.compaction` 也没有 `/compact`。
- 挂载硬约束：预设里挂 compaction **必须**在 isolate group 内，否则挂载直接失败（`dsh-agent-preset-registry/src/mount.ts:204-215`）。
- 取引擎的路：根上下文 `ctx.get('compaction')` 取不到（根作用域 disable 了）；可行路径是 `agents.get(sessionId)` 拿 agent，再 `agent.ctx.get('compaction')` 沿父链解析到预设作用域。轮转现在只用 `sessions`，要加 `agents` 依赖。
- `compactNow(agent, signal, commandId?)` 内部走 `agent.runMaintenance`，非 idle **同步抛**，异常一律包成 `ManualCompactionError('busy')`；事务层还要求「会话没有开着的 turn」。
- 插一轮再压缩的可行性：`prompt()` 准入即返回（只把消息放进 inbox），没有「某条消息跑完」的 promise；能用的只有 `agent.whenIdle()`（宿主文档明确提醒它可能跨多条输入）、`agent/status`、`inbox/claimed`（带 turn）+ 该 turn 的 `turn/end`。
- **会拦住用户消息**：gateway 在调用 `controller.prompt` 时 await 它，我们不返回那条用户消息就进不了 inbox（客户端契约允许准入阶段扣住，且没找到 prompt RPC 超时）。所以「先跑两轮再放行」在协议上可行，代价是用户看不到任何回执。
- **一个坑**：profile 配的是 `busyEnter: steer`，agent 忙时客户端发 `steer`，而包装对 `mode !== queue` 直接放行 —— 我们插的那两轮跑起来时，用户再发消息会以 steer 注入到「更新文档」那一轮里。

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

## 可实施形态（待用户确认后照做）

每次触发（用户消息到达且过阈值）在**插件里串行四步**，用户消息扣在 gateway 等待：

1. 注入一轮 prompt：「更新当前 Plan 的 spec.md 与 note」，等该轮 `turn/end`；
2. `agent.ctx.get('compaction').compactNow(agent, signal)`；
3. 注入一轮 prompt：「读 spec.md 与 notes_index.md，并执行 nano_flow_check」，等该轮结束；
4. 放行用户消息。

前置改动：预设挂 `compaction-basic`（必须在 `cordis:group` + `isolate:{compaction:true}` 内，`auto:false`），**不挂** `command-compact`；轮转依赖从 `sessions` 扩到 `agents`；`preset.test.ts` 里「不含 compaction」的断言要改。

两个已知坑：①profile 是 `busyEnter: steer`，pre-step 轮跑着时用户再发消息会以 steer 注入到「更新文档」那一轮；②用户消息被扣住期间 GUI 没有任何回执，两轮可能几十秒到几分钟。

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

## 边界

- 先给方案与代价，动手前等用户放行。
- 不改 pi 侧。
