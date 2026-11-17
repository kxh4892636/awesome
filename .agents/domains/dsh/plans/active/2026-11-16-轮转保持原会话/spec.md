---
name: "轮转保持原会话"
description: "调研把 dsh-toolkit 的会话轮转从「新建会话 + 删除旧会话」改为「原会话内重置上下文」，让轮转后子 agent 与后台 job 继续存活，并落地实现与验收。"
status: completed
---

# 轮转保持原会话

## 目标

1. **轮转不再杀死会话里的活体**：现在轮转走「新建会话 + 归档 + 物理删除旧会话」，旧会话上的子 agent、后台 job 随之终止。目标是把轮转换做法在原会话上清掉历史上下文，会话 id 不变，子 agent 与 job 继续跑。
2. **先证明路径可行再动手**：查清 DSH 会话日志（jsonl）与活体内存各自承载什么、轮转依赖的宿主能力（sessionController / tokenMeter / 会话事件流）给不给就地截断的口子，再决定实现形态。
3. **保留现有轮转的可观察行为**：上下文压力阈值触发、seed（已轮转 + 当前 Plan + 只读恢复 + 自动校验结论 + 上一轮输出块）、Plan 绑定连续、客户端不跳会话。

## 已知事实（2026-11-16 取证）

- 轮转实现在 `repos/dsh-toolkit/src/nano-flow/session/rotate.ts`：挂 `sessionController.prompt`，只在 `queue` 模式、非子 agent、预设为 nano-flow、且 `projectedTokens ≥ rotateAt × contextWindow` 时触发。触发后 `controller.create()` 建新会话 → `inheritPlacement` 过继工作区分组/置顶 → `adoptBinding` 过继 Plan 绑定 → `autoCheck` → 组装 seed（上一轮输出 + 触发原文包进 `<rotation-prior-input>`）→ 归档并 `detachSession` 旧会话 → `recordPendingDelete` → `deleteSessionLogs` 物理删除旧会话目录。
- 旧会话被删是「子 agent 与 job 直接终止」的直接原因：轮转路径只处理会话本身，没有迁移或保留旧会话上的委派与后台作业；dsh 域 CONTEXT 也把「轮转不继承对话历史」写成既定设计。
- 与 pi 的对照：pi 侧是**上下文清空**（`nano-flow-reset` 过滤历史 + seed，不换会话），dsd 侧是**轮转**（换会话）。用户这次的预期是把 dsh 收敛到 pi 那种「原会话继续」。
- Plan 轨迹 `<plan>/log.jsonl` 独立于会话日志，轮转只在会话日志层面动作，不删轨迹；会话被移除时轨迹服务只关句柄。

## 待查（决定方案走向的关键未知）

- DSH 会话日志的落盘布局与写入者：`<DSH_HOME>/sessions/<workspace-slug>/<sessionId>/*.jsonl`，谁是唯一写者，直接改文件会不会被内存状态覆盖、会不会破坏 seq 单调性。
- 活体会话的历史是内存态还是每次从 jsonl 重放：截断 jsonl 若只影响下次加载，则对当前上下文压力无效。
- 宿主是否已提供「压缩/截断/清空上下文」能力（预设文档提到 dsh 有压缩插件位），能否直接复用而不动手改文件。
- 子 agent 与后台 job 挂在父会话上的方式：轮转（甚至只是 `archiveSession(stopActivity)`）在哪一步把它们杀掉，才谈得上保留。

## 非范围

- 不改 pi 侧的上下文清空实现。
- 不在本 Plan 内改 note 看板、dream 游标、轨迹落盘格式。
- 不为了保留 job 而引入「轮转后自动重跑上一轮工具调用」这类补偿机制。

## 风险

- 直接改 jsonl 绕过宿主写入者，可能出现内存与磁盘不一致、seq 回退、客户端面板读旧缓存；需要先证明写入口径再动手。
- 误判「活体历史来自内存」会让整个方案落空，所以 001 的结论必须带代码依据，不能只写推测。
- 保留原会话意味着 Plan 绑定不必再「过继」；改动面会波及 seed 生成、`onRotated` 路由、归档/删除路径与既有 483 行 rotate 单测。

## 恢复入口

1. 先读 [notes_index.md](notes_index.md)，按 001 → 002 → 003 → 004 推进。
2. 关键代码：`repos/dsh-toolkit/src/nano-flow/session/rotate.ts`（轮转）、同目录 `rotate.test.ts`、`repos/dsh-toolkit/src/host/undo/purge.ts`（会话日志删除）、`repos/dsh-toolkit/src/host/nano-flow-facade.ts`（`sessionsRoot`）、`repos/dsh-toolkit/src/nano-flow/assembly/`（挂载点）。
3. 交付口径：001/002 以「有代码或实测依据、结论可被第三方复核」验收；003 以单测 + 类型检查 + 构建通过验收；004 以真机跑一轮触发轮转、子 agent 与 job 存活为验收。

## 进度

- 2026-11-16 19:00：Plan 建立并绑定。用户诉求：轮转改为原会话内进行，轮转后子 agent 与 job 不终止；先调研实现路径。
- 2026-11-16 19:30：001 完成，结论 **B**：会话是「内存事件日志 + 派生表面 + append-only 落盘」，所以①改 jsonl 不降上下文、②删日志目录会毁活体会话（append 新建无 header 帧的文件 + 丢 lock inode）。可行路径是 `compaction/prune` + 带 `surfaceOp:{op:"replace"}` 的一条 user 消息原地遮蔽整段历史，sessionId 不变；宿主把它写成公开契约。子 agent / job 的真正杀手是 `archiveSession({stopActivity:true})` 发出的 `workspace/session-stop`，拆掉轮转的连带动作本身就保住它们。**用户前提「改 jsonl 轮转」与事实不符，已在 002/003 前纠正。**
- 2026-11-16 20:10：002 完成。pi 的「不换会话清空」= 写一条 `nano-flow-reset` custom 条目 + 每次请求在 `context` 钩子里按时间戳过滤消息，磁盘不删任何东西；**dsh 写死 Model-visible means logged、无 per-request 钩子**，等价物只能是持久的 surface replace。用户已确认「界面历史仍在、只有模型失忆」的观感可接受，方案定案为持久遮蔽。
- 2026-11-16 19:20：003 完成并验证。`rotate.ts` 改为原会话内遮蔽历史（`compaction/prune` + 带 `surfaceOp.replace` 的 seed 消息 + `sessions.flush`），删掉归档/删除旧会话的整条链路与 `<rotation-prior-input>` 包装；新增「回合结束才动表面」「重置后要产出新 assistant 才允许再重置」两道闸；`plan-log` 的 `user` 字段加 8 KB 上限。`pnpm test` 369 条全过，`vp check` 0 error 0 warning，`pnpm build` 通过。
- 2026-11-16 19:33：004 真机验收通过。`rotateAt` 调到 0.1 触发重置：会话 id 不变、GUI 历史仍在、job 与子 agent 在重置瞬间存活，`compaction/prune` + `surfaceOp.replace` 各一条、遮蔽 149,947 tokens；重启宿主后重放折叠无异常（turn 10 请求 52k tokens 对比重置前 201k）。4 条 note 全部 completed，等用户确认后迁 `reference`。
- 2026-11-16 19:25：提交 `6b5e9bd`，版本 bump 到 **0.5.54**，`pnpm pack` 后装入 web profile（`dsh plugin --profile web add`），`--dump-config` 组合正常。**真机验收待宿主重启**（bundle 只在启动时组合），清单见 004。

- 2026-11-16 19:36：用户放行「死代码清理」，新开 005（`rotation` 路由 + 客户端 `rotation-follow` 永远拿 null）。执行段交给 subagent `deadcode-sweeper` 串行跑：删码 → 门禁 → 提交 → bump 0.5.55 → 装 web profile（不重启宿主）。Plan 暂不迁 `reference`，等 005 收口。

- 2026-11-16 19:40：005 完成。rotation 路由与客户端跟随已拆除（640 行、13 文件，测试 370→357），check/build 通过，0.5.55 已装 web profile；README 里「建新会话 + 物理删旧会话」的陈旧描述一并改成同会话遮蔽。5 条 note 全 completed，等用户确认后迁 `reference`；生效需重启宿主。

- 2026-11-16 19:44：`/ponytail-review` 走查后修三处（失败阶段区分、seed 不再要求等放行、删死声明 `create`），提交 `000c098` + `9ec5ecb`，**0.5.56** 已装 web profile，门禁 357 测试 / check 0/0 / build 通过。两处遗留（遮蔽不可回滚的窗口、废弃的 `snapshotEvents` 读接口）记在 003 的已知限制里。

- 2026-11-16 19:46：**Plan 迁 `reference`**（用户确认）。5 条 note 全 completed，0.5.56 已装 web profile；域 CONTEXT 的链接与「失败只 warn」一句同步改成现状。

- 2026-11-16 20:12：用户要求 Plan **迁回 active**，并提出新问题：撤回工具同样靠新建会话，能否套用在原会话内遮蔽的思路。新开 006 承接；本轮只做只读调研，动手前等放行。

- 2026-11-17 10:50：用户放行「开始执行所有 NOTE」。契约先在 `src/contract.ts` 定死（`UndoTurnResponse` 去 `childSessionId`，改 `{boundary,seq,prompt}`；新增 `UndoRecallMark`），再分两条写链并行：`undo-host`（宿主半就地墓碑 + 清队列 + 修 rotate.ts 的 heuristicTokens）与 `undo-client`（visibility 隐藏 + seat 接管 + 乐观更新）。全量门禁与发布由主会话统一做。

- 2026-11-17 11:05：006/007 收口。撤回改成就地墓碑 + 客户端 `visibility` 隐藏，0.5.57 已装 web profile；测试 357→395、`vp check` 0/0、build 通过。决策记入 [ADR 014](../adr/014-撤回用墓碑遮蔽而非fork.md)，问题 006 的解法同步改写。生效需重启宿主。

- 2026-11-17 11:25：`/ponytail-review` 走查撤回改动，修两处：①已撤回区间表按会话隔离（墓碑 `data.recall` 带 `sessionId`，活动会话由撤回按钮提供），否则 A 会话撤回会让 B 会话同 seq 的行消失；②删掉多余的 `conversation.chat.node` user 座位接管（定义层 `visibility` 已覆盖用户行，抢座位反而掉装饰链），净 −109 行。测试 394 / `vp check` 0/0 / build 通过，提交 `3d479c2` + `7c477aa`，**0.5.58** 已装 web profile。

- 2026-11-17 12:05：用户决定**不要撤回功能**，008 整体移除：17 文件、净 −4757 行，测试 394→276，代码零残余（grep 空），0.5.60 已装 web profile 并推送。域文档同步：CONTEXT 删三条词条、ADR 014 删除、问题 006 改写为「表面可短」留给轮转。

- 2026-11-17 16:50：用户要求轮转改为复用宿主 compact（不暴露 `/compact`），压缩前后各插一步固定动作。调研确认可行：手动压缩只保留最后一个 surface 节点、需真 Agent 且 phase 恰为 idle、无前后钩子但我们是调用方；用户批准 `auto:false` 与全部代价。009 进入执行，由 `rotate-on-compact` 重写实现与预设。

- 2026-11-17 17:00：009 完成。轮转改为调用宿主 compact：四步串行（改文档 → 压缩 → 读文档+check → 放行用户消息），预设挂 `compaction-basic`（`auto:false`，不挂 `command-compact`），`rotate.ts` 486→374 行，测试 276、check 0/0、build 通过；提交 `6530405`，**0.5.61** 已装 web profile。

- 2026-11-17 17:15：`/ponytail-review` 走查 0.5.61 的轮转改动，修两处：①压缩前那一轮没跑完（超时/注入被拒）时**跳过压缩**，不再制造一次必然 `busy` 的假失败；②过阈值却取不到压缩引擎时留一条 warn（否则静默不轮转、上下文一路涨）。提交 `3a90ae0` + `df94382`，测试 276 / check 0-0 / build 通过，**0.5.62** 已装。

- 2026-11-17 18:00：用户报历史加载失败（session-e09f）。根因是 0.5.57 撤回墓碑写在 `turn/end` 之后，v4 冷读判 `system/message does not match an open turn and step`。新开 010：三条墓碑移回所属回合最后一个 step 内并重排 seq，日志恢复可加载；规则记入问题 016。

- 2026-11-17 18:20：真机测试发现轮转根本没触发，根因是预设 compaction 引擎在 isolate realm 里、根半 `agent.ctx.get('compaction')` 取不到。改用**同 bundle 模块级桥**：group 内加一行本包小插件登记 `compactNow`，根半从槽里取。提交 `501e672` + `25cb215`，284 测试全过，**0.5.63** 已装；等用户重启后真机验证（看 `mount.json` 的 `stages.bridge` 与日志里的 `compaction/start|summary|end`）。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
