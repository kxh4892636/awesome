---
name: '007-dream由宿主定时任务投递新会话'
description: 'dream 由宿主 Schedule 每日把 dream 指令投到新建的 nano-flow 会话，不内建定时器；游标让至少一次投递变幂等。'
status: accepted
---

# 007-dream由宿主定时任务投递新会话

dream 不是插件代码里的后台定时器，而是一条 DSH 自动化任务：标题 `nano-flow dream`，每天 08:00（Asia/Shanghai）把 dream 指令投递到一个**新建的** nano-flow 会话（原为 `prompts/dream.md` 全文，该文件已于 2026-11-19 删除，见下方纠正），由该会话的模型按指令跑五个 pass。任务与时间都存在宿主 Schedule 存储里，「自动化任务」页面可改，不进插件设置、不发版。

> 纠正（2026-11-19，[001-新增backup-plan手工技能](../plans/reference/2026-11-19-dream与backup技能化/001-新增backup-plan手工技能.md)、[004-单测与发布验收](../plans/reference/2026-11-19-dream维护纳入存量与盘点/004-单测与发布验收.md)）：投递内容已不是 `prompts/dream.md`——该文件随 0.5.76 删除，dream 指令搬成随包 skill `skills/dream/SKILL.md`，任务 prompt 换成 `/dream …`。而落进新会话的这条消息 `source.kind` 是 `schedule`（`@deepseek-ai/dsh-schedule/lib/index.js:1387`、`:2561`），技能手势扫描只受理 `user`（`@deepseek-ai/dsh-tool-skill/lib/index.js:1120`），`skill` 工具对 `disable-model-invocation` 的技能直接抛错（同文件 `:876-879`，2026-11-19 真机实测）。结论：**定时轮加载不到 dream 技能，维护实际不跑**；只有人工 `/dream` 生效，出路见 [question 017](../questions/017-定时投递的dream取不到技能.md)。

## Considered Options

- **宿主 Schedule + `target: new`（选定）**：复用宿主已有的调度、投递与「至少一次」语义；每次新会话，上下文不逐日累积；任务与插件版本解耦。
- **`target: new_once`（固定会话复用）**：省一次新建，但会话上下文逐日膨胀；该会话被轮转删除后，Schedule 会拒绝向已归档目标投递。
- **插件内定时器**：要自己处理宿主启停、时区、错过补投与并发，且插件卸载或升级即失效——等于重写一遍 Schedule 服务。
- **只保留手动复盘（原 kaizen skill）**：维护取决于用户是否记得；kaizen 已删除，可复用规则并入 `dream.md`。

## Consequences

- 宿主必须运行；错过 08:00 只补投最近一次发生点，投递是「至少一次」，所以 dream 靠游标（`$DSH_HOME/toolkit/nano-flow-dream.json`）做增量与幂等。
- 新会话继承**创建任务时**的 preset 与工作目录：任务必须在 nano-flow 会话里建（或在页面里选该 preset），否则新会话没有 `system.md`、`[nano-flow:dream]` 入口与 `nano_flow_*` 工具。
- 任务属于宿主，任何 nano-flow 会话都能用 `schedule_*` 查看或修改；创建与修改需要用户明确授权。
- 首次运行时游标文件不存在，dream 会把 `log.jsonl` 已有轮次全量处理一遍；结果必须幂等。
- 验收：`schedule_view` 显示 `status: active`、`rrule` 与 `target.create.agentPreset` 正确；到点后出现新会话并跑完五个 pass，check 通过且产生一次 `docs(dream):` 提交。

## 相关

- [023-dream指令与自动化接入](../plans/archived/2026-11-08-nano-flow重构与dsh迁移/023-dream指令与自动化接入.md) — 任务配置与首次创建记录，含创建即投递的异常观察。
- [019-dream机制设计](../plans/archived/2026-11-08-nano-flow重构与dsh迁移/019-dream机制设计.md) — 运行形态、游标与五 pass 的完整设计。
- [017-定时投递的dream取不到技能](../questions/017-定时投递的dream取不到技能.md) — 本决策在 0.5.78 上的断点：定时投递的消息打不开技能。
