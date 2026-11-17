---
name: 'DSH'
description: '工作区自研 DeepSeek Harness 扩展的业务域：独立插件包、可开关工具与 Nano Flow 预设。'
relations:
  - to: 'nano-flow'
    description: 'DSH 工具集的嵌套 skill 工具使 Nano Flow 以嵌套 skill 形式分发的主流程 skill 能被发现与调用。'
metadata:
  owner: 'dsh'
---

# DSH

工作区自研 DeepSeek Harness 扩展的业务域：以独立插件包承载一组可独立开关的工具（嵌套 skill 发现、会话撤回、office 技能屏蔽等），并提供 Nano Flow Agent 预设，与上游仓库解耦。

## Language

### 嵌套 skill 发现

**嵌套 skill**：
位于 `.agents` 树中、处于内建一层形态之外的 `SKILL.md` 技能；由本域插件任意深度发现。
_Avoid_：子 skill、深层 skill、二级 skill

**内建一层形态**：
`.agents/skills/<skill>/SKILL.md`（及根下平铺 `.md`），DSH 内建 provider 唯一发现的技能位置。
_Avoid_：顶层 skill、直接子级 skill

**注册名**：
嵌套 skill 在 DSH 目录中的名称，取自 skill frontmatter 的 `name` 原样。
_Avoid_：命名空间名、前缀名

**声明优先**：
同来源内同名师冲突时内建一层形态胜过嵌套 skill 的取胜规则；显式声明者胜。跨来源仍按上游来源序（project > custom > user > bundled）。
_Avoid_：嵌套优先、注册顺序优先

### 工具集

**工具集**：
一个插件包内可独立开关的多项能力；`@kxh4892636/dsh-toolkit` 是唯一实例。
_Avoid_：插件集合、子插件

**工具开关**：
按工具 id 持久化的启用状态，宿主持有、面板可写；关闭即卸载该工具在宿主与界面的全部效果。
_Avoid_：配置项、feature flag

**office 屏蔽**：
把宿主机内置的 office 技能从技能注册表读取面（`list` / `snapshot` / `get`）过滤掉的开关效果；不注销 provider、不改技能文件，关闭即还原。
_Avoid_：删除 office 技能、卸载 office 插件

### Nano Flow 预设

**nano-flow 预设**：
本插件声明的 Agent 预设；选中它的会话才在预设作用域内挂载固定提示词、`GUIDE.md` 快照、篇幅限制块、四个 `nano_flow_*` 工具（create / check / bind / dream）与随包 skill。工具以 PTC 形态呈现：预设直接挂 `dsh-tool-executor-ptc`，不挂已弃用的呈现选型器（[ADR 011](adr/011-预设挂PTC执行器.md)）。提示词与模板与 pi 版同源（2026-11-13 整体搬回）；2026-11-13 起随包 skill 只剩 code-spec / learning / ponytail / ponytail-audit / ponytail-review / to-skill——ask、do 两个流程 skill 随 pi 删除，流程规则只由提示词与模板承担；2026-11-16 加 `ap`（async plan，由 `pp` 改名）与 `ad`（async do）两个并行 skill：都用 subagent 干活、不阻塞主会话，`ap` 只读调研、`ad` 读写执行，两者与 learning、ponytail-audit 一样只可手调（`disable-model-invocation`）；`ponytail` 相反，可由模型自动选中，指「最小改动 + 一次读懂」的编码风格，`ponytail-review` 用来审一处改动。
_Avoid_：流程开关、nano-flow 命令行

**固定提示词**：
`prompts/system.md` 的正文（源码在 `session/prompts/`，发布布局仍是 `dist/nano-flow/prompts/`，见 [ADR 012](adr/012-源码按能力分层.md)），与 `.agents/domains/GUIDE.md` 的会话启动快照、`prompts/limit.md` 生成的篇幅限制块一起作为系统提示词段注入。
_Avoid_：persona、系统提示

> 纠正（2026-11-08，note 006）：本节原写 `prompts/agents.md`；该文件已改名为 `prompts/system.md`，注入段名 `nano-flow:system`（原名会被当作 `AGENTS.md` 自动注入）。

**领域文档**：
`.agents/domains/{domain}/` 下的 CONTEXT、questions、adr、workflows 与 plans；文档以 YAML 头声明 `name` / `description`，索引由 `nano_flow_check` 生成，模型不改生成物。
_Avoid_：wiki、知识库、notes.json

**文档校验**：
`nano_flow_check` 对领域文档做的三件事：按 YAML 头、编号、篇幅与 `## 相关` 规则校验；生成全部索引、`CONTEXT-MAP.md` 与 spec 的追踪表；就地修「结果由既有事实唯一确定」的问题——spec 状态按 note 派生、`name` 对齐文件名并把改名传播到 `related_noted`、非 blocked 的 `reason` 清掉、多行头字段折成单行、缺 `questions/` 建目录。修完报「已自动修复 N 项」，再跑一次为 0 才算收敛；篇幅超限、`## 相关` 目标缺失或原因为空、真实编号缺口、blocked 缺 `reason` 只报错不猜。写文件经过工作区边界检查、临时文件 rename、内容相同则不写。
_Avoid_：自动纠正、静默改写

**Plan**：
一个会话默认维护的工作单元，包含 `spec.md`、`notes_index.md`（生成）与若干 `NNN-中文标题.md` 的 note；note 用 `pending` / `in_progress` / `blocked` / `completed` 表达进度，`blocked` 时以 `reason` 说明在等什么。交付完成但未验收一律 `blocked`，只有用户验收后才置 `completed`。
_Avoid_：story、DAG、任务列表

**交付单元**：
note 的划分单位：一个可独立验收的交付物；同一单元内的返工、补测、修正与验证并入原 note，只有验收条件或状态需要分别表达（会单独 `blocked`、单独验收）时才拆（[ADR 010](adr/010-note按交付单元划分.md)）。
_Avoid_：原子子任务、按主题聚合

**批量写**：
Plan 与领域文档的写入节奏：每轮最多两次（轮首记本轮目标与方向，轮末补进度、状态、证据与 spec），每次批量写完之后跑一次 `nano_flow_check`，同一批次不重复跑；面板与索引在一轮之内允许滞后。
_Avoid_：状态先行、实时逐条写

**会话↔Plan 绑定**：
`$DSH_HOME/toolkit/nano-flow-plan-bindings.json` 中按会话 id 记录的唯一 Plan（`planPath` / `cwd` / `boundAt`）；同一会话只保留一个绑定，只有 `nano_flow_bind` 写它，再次调用即切换。供 Note 看板与 `log.jsonl` 使用，不参与 note 状态判定。

绑定的 Plan 目录由 `resolvePlanDirectory` 统一解析：绝对 `planPath` 直通；相对路径先试绑定时的 `cwd`，该目录不在就退回当前工作区。Plan 迁走后目录解析不到，看板返回空、`log.jsonl` 跳过写入（不重建旧路径），不会产生孤儿日志。死条目不由 bind 回收（2026-11-13 按 `index.ts` 的 `pruneDeadBindings` 修正，原写「bind 收尾回收」）：插件启动时按「持久化列表 ∪ 仍存活的会话」清一次，拿不到列表时不动表。
_Avoid_：当前 Plan、任务上下文

**生效上限**：
toolkit 面板「Nano Flow 设置」维护的数值来源，持久化在 `$DSH_HOME/toolkit/nano-flow.json`；启动时由它生成「篇幅限制」块，不把具体数字写死在提示词里。
_Avoid_：魔法数字、硬编码阈值

**上下文重置**：
预设不挂压缩插件；每发一条 queue 消息时读上下文压力，达到 `rotateAt × contextWindow` 就在**原会话内**遮蔽掉历史：保留表面第一个节点（系统提示），对 `nodes[1..]` 先追加 `compaction/prune` 记遮蔽价，紧跟一条带 `surfaceOp:{op:"replace"}` 的 `user/message` 把 seed 顶上去，最后 `sessions.flush()`。会话 id、工作区槽位、置顶、Plan 绑定、子 agent 与后台 job 全部原样存活——旧做法是新建会话 + 归档 + 删旧会话，`workspace/session-stop` 会把它们一并杀掉（[2026-11-16 Plan](plans/active/2026-11-16-轮转保持原会话/spec.md)）。seed = 「已重置」+ 当前 Plan（有绑定时附「绑定未变，无需再 bind」）+ 只读恢复要求 + 自动校验结论 + 末段上一轮模型输出 `<reset-last-output>` 块；用户原文不再包装，直接跟在 seed 后发出。两道闸：回合没结束不动表面（否则会抽掉进行中那一步的历史）、重置后必须产出新的 assistant 消息才允许再重置（防遮蔽价失真时连击）。失败只 warn 并回落成普通 prompt；若遮蔽已落地而这条 prompt 被拒，日志会分阶段标出（历史已丢，消息可能要重发）—— 遮蔽是 append-only，没有回滚路径。遮蔽是持久的：日志不删，人类可见记录（append-origin 事件）与 `<plan>/log.jsonl` 仍是全史，但模型、重启后的续聊、fork 与导出都看不到被遮蔽的部分。旧轮转的两条结论（[ADR 013](adr/013-轮转把Plan绑定过继给新会话.md)、[question 014](questions/014-轮转为何绑定到已归档Plan.md)）只对已下线的做法有效。
_Avoid_：建新会话、归档、删除会话日志、改 `session.v4.jsonl.zstd`

**恢复只读**：
用户说“恢复”“继续”且会话还没加载 Plan 时先调用 `nano_flow_bind` 绑定目标 Plan，再只读加载：读 `spec.md` 与 `notes_index.md`，报告进度与可推进队列；除 bind 的会话绑定外不写文件、不执行 note。恢复报告之后的第一条用户消息才放行。
_Avoid_：自动续跑、恢复即执行

**note 面板**：
nano-flow 预设会话在输入框上方 dock 显示的 note 面板（中文状态名 + 名称，hover 显示描述 + 有值时的「原因」「下一步」三行）；已绑定 Plan 时**常驻**且**默认展开**（点标题行收起/再展开，状态存内存，重挂载回到展开），未完成列表为空时显示「全部完成」空态（无绑定或目录失效才不显示）。客户端按当前 `sessionId` 每 5s 轮询宿主 `GET /api/toolkit/nano-flow/notes`，宿主按该会话的绑定定位 Plan，快照不写会话日志，样式与 Todo 面板一致。
_Avoid_：待办面板、todo 面板、任务列表

**Plan 轨迹日志**：
nano-flow 预设会话每轮结束时向 `<plan>/log.jsonl` 追加的一条 compact 轨迹（user、assistant 最终文本、工具参数/结果摘要、usage、seq 区间）；绑定只由 `nano_flow_bind` 与轮转过继写（[ADR 009](adr/009-Plan绑定只经bind工具.md)、[013](adr/013-轮转把Plan绑定过继给新会话.md)），未绑定时服务只跟踪事件、不落盘；绑定后写当前轮与后续轮次，绑定前已完成的轮次不回填；切换后后续轮次写新 Plan，旧 Plan 不再追加。启动时从宿主持久化日志回填已绑定且归该 Plan 的历史轮次，绝不写会话事件。日志只存本地（`.gitignore` 忽略 `**/plans/**/log.jsonl`），轮转不删、只增量追加。会话被删除或撤回时轨迹服务关掉该会话的读句柄并丢掉状态，迟到事件不再重建，句柄不随轮转累积。
_Avoid_：会话转录、原始事件日志、audit log

### 记忆维护（dream）

**记忆分层**：
nano-flow 记忆的四层：L0 `<plan>/log.jsonl` 轨迹、L1 note 的结论、L2 领域文档与「## 相关」互链、L3 `GUIDE.md` 会话快照（`system.md` 由用户维护）；dream 只写 L2/L3 正文，其中 L3 正文只由 dream 父会话整体重写、`nano_flow_create` 仅在缺失时补空模板，生成物只由 `nano_flow_check` 写。
_Avoid_：记忆库、知识图谱、第五层

**dream**：
后台记忆维护回合：宿主定时任务把 `dream/prompt.md` 全文（发布布局 `dist/nano-flow/prompts/dream.md`）投递到新建的 nano-flow 会话，按游标把 `log.jsonl` 的新轮次做纠错、提炼、建链、晋升、剪枝，落到领域文档与 `GUIDE.md`。
_Avoid_：自动复盘、kaizen、离线批处理

**dream 游标**：
`$DSH_HOME/toolkit/nano-flow-dream.json`（v2）按「Plan + 会话」记录已处理到的 turn seq 上界；只由 `nano_flow_dream` 的 `mark` 动作在 `git commit` 成功后推进（实现是随包 `dream-scan.mjs`），重复投递靠它保持幂等。seq 换会话会重新计数，所以旧 v1 的 Plan 级 `lastSeq` 一律当空游标全量处理。
_Avoid_：进度条、checkpoint

### 会话撤回

**撤回**：
把会话回到上一轮结束的状态并取回该轮用户原文的操作，**就地**完成：同一会话追加一条 `system/message` 墓碑（空 content + `data.recall{boundary,end}`），带 `surfaceOp:{op:"replace"}` 覆盖切点到表面结尾，`sourceEventSeqs` 覆盖全部被遮蔽节点；会话 id、Plan 绑定、置顶位、子 agent 与后台 job 全部不变。撤回前先清掉该会话排队中的输入，否则待发消息会立刻开一轮，把刚遮蔽的历史重新长出来。
_Avoid_：回滚、撤销历史、时间旅行

**切点**：
被撤回那一轮**第一条用户消息**的 seq（该轮没有用户文本时退回第一条 assistant 消息），必须仍是表面上的 append 节点；它同时是 replace 区间的起点。早先已被遮蔽（撤回过、被压缩过）的消息不能当锚。
_Avoid_：断点、分割点、fork 边界

**界面隐藏**：
客户端半读墓碑里的 `data.recall`（含 `sessionId`），把**当前会话**里 `boundary..end` 区间的聊天视图节点置 `visibility:"hidden"`（宿主 `isVisibleChatNode` 按它过滤）；用户提问行也走这条定义层通道，不抢 `conversation.chat.node` 的 seat（抢了会掉宿主的装饰链）。区间按会话隔离，当前会话未知时不隐藏。改的是视图节点、不动事件日志：日志、导出与搜索索引里那些事件仍在（搜索按表面过滤，所以搜不到但翻得到）。
_Avoid_：删消息、删除行、清空历史

### 插件交付

**DSH 插件**：
以工作区独立包开发、经 `dsh plugin --profile <name> add` 安装的 DSH 扩展；本域产物。
_Avoid_：上游补丁、改动 DSH 仓库

**发现根**：
参与嵌套 skill 扫描的 `.agents` 根：项目 `<gitRoot>/.agents` 与用户 `$DSH_AGENTS_HOME`（缺省 `~/.agents`）。
_Avoid_：skills 根、扫描目录