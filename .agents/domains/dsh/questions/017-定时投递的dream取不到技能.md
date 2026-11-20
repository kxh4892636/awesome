---
name: '017-定时投递的dream取不到技能'
description: '定时任务投递的消息 source.kind 是 schedule，技能手势只认 user、skill 工具又拒 disable-model-invocation，于是定时轮 dream 取不到技能、维护不跑。'
---

# 017-定时投递的dream取不到技能

## 现象

宿主定时任务（`nano-flow dream`，每日 08:00 Asia/Shanghai）到点确实新建了 nano-flow 会话，但那轮 dream 没有跑：会话里看不到 `skills/dream/SKILL.md` 的内容，也没有坚持「核查了哪些域、改了什么」的收尾报告；手工在新会话敲 `/dream` 却正常。

## 原因（2026-11-19 核对 0.5.78 与宿主包）

1. 定时任务投递的消息 `source` 是 `{ kind: "schedule" }`（`@deepseek-ai/dsh-schedule/lib/index.js:1387`、`:2561`）。
2. 技能手势扫描只受理用户手势：`/name` 只在 `message.source.kind === "user"` 时解析（`@deepseek-ai/dsh-tool-skill/lib/index.js:1120`），所以提示词里的 `/dream` 是普通文字，不注入技能内容。
3. 模型也补不上：`dream` 带 `disable-model-invocation: true`，`skill("dream")` 直接抛 `skill "dream" is not available for model invocation`（同文件 `:876-879`，2026-11-19 真机同一句）。
4. 2026-11-19 起 dream 指令已从 `prompts/dream.md` 搬进技能，所以「把整份提示词当任务 prompt 投递」的旧办法也一并失效。

## 处理（2026-11-20 走第二条）

实际选的是「任务 prompt 改成普通指令」：prompt 现在是 `/dream 调用 dsh-toolkit 中 nano-flow 的 dream skill，维护领域文档。`，模型自己按名找到 `repos/dsh-toolkit/src/nano-flow/skills/dream/SKILL.md` 读全文再执行。2026-11-20 22:00 那轮实测跑通（scan + 五 pass + 校验 + commit），代价是路径靠模型自己猜得对。下面是当初的三条候选。

- **让 dream 可被模型选中**：去掉 `disable-model-invocation` 与 `agents/openai.yaml` 的 `policy.allow_implicit_invocation: false`。定时轮能跑，代价是日常会话里模型可能自己挑中它。
- **任务 prompt 改成普通指令**：写「读 `dist/nano-flow/skills/dream/SKILL.md` 并按它执行」，不依赖技能注入；代价是把技能路径写死在宿主任务里，升级要同步改。
- **放弃定时**：只留人工 `/dream`，也就是当前实际状态。

判别方法：看定时到点的新会话里有没有 dream 技能内容与五 pass 结论。没有即命中本条，不必去查游标或 scan。

## 验证

- 源码两处判据见上（手势扫描的 `source.kind`、`skill` 工具的 `isModelInvocable` 拒绝）。
- 真机：2026-11-19 turn 10 调 `skill("dream")` 得到上面那句报错（[004-单测与发布验收](../plans/reference/2026-11-19-dream维护纳入存量与盘点/004-单测与发布验收.md) 的「未闭合」一节）。
- 任务现状：prompt 已换成 `/dream …`（[001-新增backup-plan手工技能](../plans/reference/2026-11-19-dream与backup技能化/001-新增backup-plan手工技能.md)）。

## 相关

- [007-dream由宿主定时任务投递新会话](../adr/007-dream由宿主定时任务投递新会话.md) — 触发方式的决策，本条是它在 0.5.78 上的断点
- [013-随包skill缺description被静默丢弃](013-随包skill缺description被静默丢弃.md) — 同一套 frontmatter 与 agents/openai.yaml 的准入门槛，缺了就是静默失效
