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

工作区自研 DeepSeek Harness 扩展的业务域：以独立插件包承载一组可独立开关的工具（嵌套 skill 发现、office 技能屏蔽等），并提供 Nano Flow Agent 预设，与上游仓库解耦。

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
本插件声明的 Agent 预设；选中它的会话才在预设作用域内挂载固定提示词、`GUIDE.md` 快照、篇幅限制块、四个 `nano_flow_*` 工具（create / check / bind / dream）与随包 skill。工具以 PTC 形态呈现：预设直接挂 `dsh-tool-executor-ptc`，不挂已弃用的呈现选型器（[ADR 011](adr/011-预设挂PTC执行器.md)）。提示词与模板与 pi 版同源（2026-11-13 整体搬回）；2026-11-13 起随包 skill 只剩 code-spec / learning / ponytail / ponytail-audit / ponytail-review / to-skill——ask、do 两个流程 skill 随 pi 删除，流程规则只由提示词与模板承担；2026-11-16 加 `ap`（async plan，由 `pp` 改名）与 `ad`（async do）两个并行 skill：都用 subagent 干活、不阻塞主会话，`ap` 只读调研、`ad` 读写执行，两者原本只可手调，但它们的 `agents/openai.yaml` 的 `policy.allow_implicit_invocation: false` 在 DSH 里不生效——DSH 只认 frontmatter 的 `disable-model-invocation`（[question 013](questions/013-随包skill缺description被静默丢弃.md)），所以 `ap` / `ad` 实际仍可被模型选中；`learning`、`ponytail-audit` 带该字段，才算真手调；`ponytail` 相反，可由模型自动选中，指「最小改动 + 一次读懂」的编码风格，`ponytail-review` 用来审一处改动。2026-11-19 再加两个手调 skill：`dream`（由 `dream/prompt.md` 整体搬成技能，承载同一套五步维护循环）与 `backup-plan`（按整体内容重命名当前 Plan 并归档，`reference` 还是 `archived` 由用户定）。
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
预设里挂宿主的压缩引擎但**不挂** `/compact` 命令适配器，且引擎配 `auto: false`，所以压缩只能由轮转触发。每发一条 queue 消息时读上下文压力，达到 `rotateAt × contextWindow` 就串行四步：①注入一轮让 agent 更新当前 Plan 的 `spec.md` 与 note 并等它结束；②从模块级桥取压缩引擎调用 `compactNow(agent, signal)`；③再注入一轮让它读 `spec.md` 与 `notes_index.md` 并跑 `nano_flow_check`；④才放行用户原来那条消息。会话 id 与活体（Plan 绑定、子 agent、后台 job）全程不变。压缩后模型看到的是 system + 一条 `compact-checkpoint`（引擎生成的摘要）+ 最后一个 surface 节点 —— 因为手动路径的保留量硬编码为 0（`retainRatio` 只影响自动路径）。四步里前后两轮靠「该会话的下一次 `turn/end`」判定结束（带注入前的最大轮号作下限、10 分钟预算），任何一步失败只 warn 并继续放行；缺 `tokenMeter`/`agents`/等待 seam 或桥未登记时轮转整体不介入（原样放行）。压缩引擎必须挂在 `cordis:group` + `isolate` 里（否则预设挂载直接报错），而 isolate 的服务是 realm 私有符号、根半取不到，所以预设 group 内再挂一行本包小插件，它 `inject` 引擎后把 `compactNow` 登记进同 bundle 的模块级槽——服务不跨 realm，同一模块的变量跨 realm。已知坑：profile 是 `busyEnter: steer`，插入的两轮跑着时用户再发消息会以 `steer` 注入那一轮；用户消息被扣在 gateway 期间 GUI 没有回执。旧的两条结论（[ADR 013](adr/013-轮转把Plan绑定过继给新会话.md)、[question 014](questions/014-轮转为何绑定到已归档Plan.md)）只对已下线的做法有效。
_Avoid_：建新会话、归档、删除会话日志、改 `session.v4.jsonl.zstd`

**恢复只读**：
用户说“恢复”“继续”且会话还没加载 Plan 时先调用 `nano_flow_bind` 绑定目标 Plan，再只读加载：读 `spec.md` 与 `notes_index.md`，报告进度与可推进队列；除 bind 的会话绑定外不写文件、不执行 note。恢复报告之后的第一条用户消息才放行。
_Avoid_：自动续跑、恢复即执行

**note 面板**：
nano-flow 预设会话在输入框上方 dock 显示的 note 面板（中文状态名 + 名称，hover 显示描述 + 有值时的「原因」「下一步」三行）；已绑定 Plan 时**常驻**且**默认展开**（点标题行收起/再展开，状态存内存，重挂载回到展开），未完成列表为空时显示「全部完成」空态（无绑定或目录失效才不显示）。客户端按当前 `sessionId` 每 5s 轮询宿主 `GET /api/toolkit/nano-flow/notes`，宿主按该会话的绑定定位 Plan，快照不写会话日志，样式与 Todo 面板一致。
_Avoid_：待办面板、todo 面板、任务列表

**Plan 轨迹日志**：
nano-flow 预设会话每轮结束时向 `<plan>/log.jsonl` 追加的一条 compact 轨迹（user、assistant 最终文本、工具参数/结果摘要、usage、seq 区间）；绑定只由 `nano_flow_bind` 与轮转过继写（[ADR 009](adr/009-Plan绑定只经bind工具.md)、[013](adr/013-轮转把Plan绑定过继给新会话.md)），未绑定时服务只跟踪事件、不落盘；绑定后写当前轮与后续轮次，绑定前已完成的轮次不回填；切换后后续轮次写新 Plan，旧 Plan 不再追加。启动时从宿主持久化日志回填已绑定且归该 Plan 的历史轮次，绝不写会话事件。日志只存本地（`.gitignore` 忽略 `**/plans/**/log.jsonl`），轮转不删、只增量追加。会话被宿主删除后事件自然停止；读句柄只在服务 dispose 时统一关闭，不随轮转累积。
_Avoid_：会话转录、原始事件日志、audit log

### 记忆维护（dream）

**记忆分层**：
nano-flow 记忆的四层：L0 `<plan>/log.jsonl` 轨迹、L1 note 的结论、L2 领域文档与「## 相关」互链、L3 `GUIDE.md` 会话快照（`system.md` 由用户维护）；dream 只写 L2/L3 正文，其中 L3 正文只由 dream 父会话整体重写、`nano_flow_create` 仅在缺失时补空模板，生成物只由 `nano_flow_check` 写。
_Avoid_：记忆库、知识图谱、第五层

**dream**：
后台记忆维护回合：随包技能 `skills/dream/SKILL.md`（2026-11-19 起从 `dream/prompt.md` 搬来，`disable-model-invocation`，只能人工点名调用；同日按 `to-skill` 重写成一个维护循环「取输入 → 定范围 → 维护 → 归档 → 收尾」，细则下沉到 `reference/`），宿主定时任务只投一句「调用 dream 技能」到新建的 nano-flow 会话。一轮的输入是 scan 给的三样：游标之后的新 turn（**新证据**，新结论只从这里来）、每域全部 `active` + `reference` 的**存量 Plan**（spec 状态、note 完成数、轨迹字节）、**领域文档盘点**（规模、时间、`index` 齐备、以及「域文档可能落后」标记）；没有新证据的轮次照样按标记复核该域并出结论。五个 pass（纠错、提炼、建链、晋升、剪枝）落到领域文档与 `GUIDE.md`；`reference` 中判定无用的 Plan 由 dream 迁 `archived`（结论已上浮 + note 全 completed + 无唯一出处，三条同时成立，单次最多 3 个，`active` 不自动归档）。
_Avoid_：自动复盘、kaizen、离线批处理

> 纠正（2026-11-19，[001-新增backup-plan手工技能](plans/reference/2026-11-19-dream与backup技能化/001-新增backup-plan手工技能.md)、[004-单测与发布验收](plans/reference/2026-11-19-dream维护纳入存量与盘点/004-单测与发布验收.md)）：原写「只能人工或定时任务点名调用」——定时那条不成立。投递消息 `source.kind` 是 `schedule`，技能手势扫描只认 `user`，`skill` 工具又拒 `disable-model-invocation`，所以定时轮取不到技能，只有人工 `/dream` 生效（[question 017](questions/017-定时投递的dream取不到技能.md)）。

**dream 游标**：
`$DSH_HOME/toolkit/nano-flow-dream.json`（v2）按「Plan + 会话」记录已处理到的 turn seq 上界；只由 `nano_flow_dream` 的 `mark` 动作在 `git commit` 成功后推进（实现是随包 `dream-scan.mjs`），重复投递靠它保持幂等。seq 换会话会重新计数，所以旧 v1 的 Plan 级 `lastSeq` 一律当空游标全量处理。
_Avoid_：进度条、checkpoint

### 插件交付

**DSH 插件**：
以工作区独立包开发、经 `dsh plugin --profile <name> add` 安装的 DSH 扩展；本域产物。
_Avoid_：上游补丁、改动 DSH 仓库

**发现根**：
参与嵌套 skill 扫描的 `.agents` 根：项目 `<gitRoot>/.agents` 与用户 `$DSH_AGENTS_HOME`（缺省 `~/.agents`）。
_Avoid_：skills 根、扫描目录