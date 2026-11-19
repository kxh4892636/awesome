# GUIDE

`awesome` 工作区的会话启动快照（本地目录仍叫 `kxh-kit`）。这里只放索引与指针：结论在下列文档里，需要细节就点进去，不要在这里复述。

## 工作区形态

- 容器仓 `kxh4892636/awesome` 只做工作区与仓库管理：根上只有 `.agents/`、`scripts/`、`workspace.yaml`、说明文件与运行目录，**没有 `package.json` / 锁文件 / `node_modules`**（[common ADR 002](common/adr/002-容器仓只做工作区管理.md)）。
- 九个包各是一个独立仓，平铺在 `repos/<名>`，各自带 `.git`、`pnpm-lock.yaml` 与工具链；清单是根 `workspace.yaml`（name / url / branch / path）。改代码要进对应子仓，容器仓 `git status` 看不到子仓改动。
- 跨仓依赖只走「仓内 `release/*.tgz` + public raw URL」：不发 npm、不用 Release 资产、不用 git 源码（[common ADR 003](common/adr/003-跨仓制品走仓内tgz与raw链接.md)）。
- 只存在于磁盘的私有内容（如 `repos/wiki/docs/010-记录`）不进任何 git；搬目录必须整目录搬移，验收与踩坑见 [common 002 工作流](common/workflows/002-拆分单仓为多个独立仓/002-拆分单仓为多个独立仓.md)。

## 领域

- **pi**（`pi`）：当前主力扩展域，`@kxh4892636/pi-toolkit` 一个包五个开关（nestedSkills / sessionName / webTools / askQuestions / nanoFlow），Nano Flow 也在这里实现。词表见 [pi/CONTEXT.md](pi/CONTEXT.md)。
- **DSH**（`dsh`）：`@kxh4892636/dsh-toolkit` 工具集与 DSH 版 Agent 预设（宿主侧完整形态）。词表见 [dsh/CONTEXT.md](dsh/CONTEXT.md)。
- **nano-flow**：`nnf` CLI 只剩 `anki` / `self` / `fixture`；`workspace` 命令组与文档流程 skill 已于 2026-11-09 下线，流程改由 pi / dsh 的 nano-flow 实现提供。词表见 [nano-flow/CONTEXT.md](nano-flow/CONTEXT.md)。
- **common**：跨域共享约定（工作区形态、容器仓与子仓、受管 skill、执行模态、同源实现、发布包依赖、Vite+ 工具链）。词表见 [common/CONTEXT.md](common/CONTEXT.md)。
- **wiki**（`wiki`）：`repos/wiki` 是 Docusaurus 个人知识站，栏目 = `docs/` 一级编号目录；思维导图是正文里的 `<Mindmap>` 渲染块，节点 = markdown 标题、编号即 id、文档引用写作 `[说明](路径.md) doc:<id>`（[adr 001](wiki/adr/001-导图是文档内渲染块而非独立路由.md)、[adr 003](wiki/adr/003-节点编号即id.md)）。词表见 [wiki/CONTEXT.md](wiki/CONTEXT.md)。
- **etf**（`etf`）：`repos/etf` 是单应用行情产品（client / server / shared），看板 = 摘要 + K 线/虚拟均线 + 估值区（PE/PB/PS/股息率读数、成立以来分位、分位曲线）；数据落本机 SQLite 的 `daily_bars` 与 `security_valuation`，日常按需刷新、历史缺段走 `sync:market` / `sync:valuation`。词表见 [etf/CONTEXT.md](etf/CONTEXT.md)。
- music / video 各自是产品域。全部入口见 [CONTEXT-MAP.md](CONTEXT-MAP.md)。

## 不可违反的约定

- 生成物只由 `nano_flow_check` 写：`CONTEXT-MAP.md`、各 `index.md`、`notes_index.md`、spec 的 `## 追踪 Note`。它还就地修「由既有事实唯一确定」的问题并报「已自动修复 N 项」，再跑一次为 0 才收敛（[dsh 文档校验](dsh/CONTEXT.md)）。
- Plan 是唯一权威状态；note 四态 `pending / in_progress / blocked / completed`，`blocked` 必带 `reason` 写清在等什么（[006](dsh/adr/006-note阻塞态以reason表达.md)）。
- 一条 note = 一个可独立验收的交付单元，同单元返工并入、不新开（[010](dsh/adr/010-note按交付单元划分.md)）。
- 每轮最多两次批量写（轮首记方向、轮末补进度与证据），每批之后跑一次 `nano_flow_check`；面板与索引一轮之内允许滞后。
- 恢复已有 Plan 的那一轮只读：先 `nano_flow_bind`，再只报告，等用户第一条后续消息放行。
- Plan 绑定只由 `nano_flow_bind` 与轮转过继写（[009](dsh/adr/009-Plan绑定只经bind工具.md)、[013](dsh/adr/013-轮转把Plan绑定过继给新会话.md)）；过继只按「目录存在」判断，Plan 迁 reference 后的空壳目录照样被继承（[014](dsh/questions/014-轮转为何绑定到已归档Plan.md)）。`GUIDE.md` 正文只由 dream 重写。
- 上下文续接：DSH 是会话轮转（新会话 + seed，物理删除旧会话）；pi 是上下文清空（`nano-flow-reset` 过滤历史 + seed，不换会话）。两者 seed 都只带当前 Plan 与上一轮输出，所以**每轮结束前必须把检查点写回 spec 与 note**。
- 领域文档的校验规则由包内 `check-domain.mjs` 执行（YAML 头 + 小写 `questions/` + 三位编号）。pi 与 dsh 各带一份（`repos/pi-toolkit/src/nano-flow/scripts/` 与 `repos/dsh-toolkit/src/nano-flow/scripts/`），`lib/` 逐字一致，**改规则两包同改**（[common 004](common/questions/004-两包校验规则是否同源.md)）。随包旧 skill 的布局（`story.md`、`QUESTIONS.md`、四位编号、story 模型）不适用于本工作区的领域文档。

## 记忆维护

- 两种触发：DSH 由宿主每日 08:00 把 `dream.md` 投递到新会话，游标在 `$DSH_HOME/toolkit/nano-flow-dream.json`，只在 commit 成功后推进（[007](dsh/adr/007-dream由宿主定时任务投递新会话.md)）；PTC 会话里 `run_code` 的 `process.env` 为空，不显式传 `statePath` 就报「找不到游标路径」（[dsh 011](dsh/questions/011-dream找不到游标路径.md)）。pi 用 `/dream` 手动跑一次全量，每域取最新 21 个 `active` + `reference` Plan，无游标（[pi 004](pi/adr/004-dream由命令手动触发且不依赖seq游标.md)）。
- 五个 pass：纠错 → 提炼 → 建链 → 晋升 → 剪枝；只写当前 Plan 所属域与 `common`；收尾跑 `nano_flow_check` 并 `git commit`（`docs(dream): …`），工作区不干净就拒绝。
- 分层：L0 `<plan>/log.jsonl` 轨迹、L1 note、L2 领域文档 + `## 相关`、L3 本文件。pi 的轨迹是会话条目原样副本（每 run 一行，[pi 005](pi/adr/005-plan轨迹用会话条目副本按run落盘.md)），读它必须用 `bash` + `python3` 切片，`read` 会撞 50 KB 上限；scan 报「非 turn 记录」多半就是 pi 轨迹（[dsh 012](dsh/questions/012-dream扫描提示跳过的记录.md)）。
- 篇幅上限来自面板设置（每轮注入的「篇幅限制」块）；`GUIDE.md` 默认 13 KB。

## 热点入口

- 两条 `status: completed` 但仍在 `active` 的 Plan，等用户确认后才迁 `reference`：wiki [2026-11-13-思维导图插件](wiki/plans/active/2026-11-13-思维导图插件/spec.md)（8 条 note 全 completed，`<Mindmap>` 导图块已上线、整站 build 通过）；etf [2026-11-16-证券估值看板](etf/plans/active/2026-11-16-证券估值看板/spec.md)（7 条 note 全 completed，估值卡片与 `sync:valuation` 已上线；221 例单测、E2E 七组全绿，数据链路与踩坑在 [etf 002 工作流](etf/workflows/002-估值数据接入/002-估值数据接入.md)）。
- 本工作区形态的来源：[common 2026-11-14-拆分单仓为多仓库](common/plans/reference/2026-11-14-拆分单仓为多仓库/spec.md)（9 条 note 全 completed，九仓拆出并推到 public，容器仓改名 awesome）。
- 已迁 `reference` 的其它近期 Plan：dsh [2026-11-13-Windows测试假红与轮转绑定](dsh/plans/reference/2026-11-13-Windows测试假红与轮转绑定/spec.md)（375 例全绿，0.5.51 已装 desktop / web 两个 profile）、music [2026-11-11-自动播放倒计时](music/plans/reference/2026-11-11-自动播放倒计时/spec.md)（6 条 note 全 completed、已上线）、dsh [2026-11-13-nano-flow功能与链路梳理](dsh/plans/reference/2026-11-13-nano-flow功能与链路梳理/spec.md) 与 [2026-11-13-dsh与pi对齐](dsh/plans/reference/2026-11-13-dsh与pi对齐/spec.md)（12 条全 completed，0.5.47→0.5.48 发布验收）；pi 侧最近收口的是 [2026-11-09-pi-toolkit插件](pi/plans/reference/2026-11-09-pi-toolkit插件/spec.md)（含 013 `/dream` 与 014 轨迹落盘）与 [2026-11-09-commandcode-provider接入](pi/plans/reference/2026-11-09-commandcode-provider接入/spec.md)。
- 更早的 dsh Plan：[2026-11-09-nano-flow优化迭代](dsh/plans/reference/2026-11-09-nano-flow优化迭代/spec.md)（绑定、面板提示、写入节奏、note 划分、代码治理 16 条全 completed，收口在 [源码按能力分层](dsh/adr/012-源码按能力分层.md)）；轮转与 seed 设计在 [2026-11-08-nano-flow重构与dsh迁移](dsh/plans/archived/2026-11-08-nano-flow重构与dsh迁移/spec.md)。
- 源码：pi 侧 `repos/pi-toolkit/src/nano-flow/`（`prompts/system.md`、`prompts/dream.md`、`scripts/check-domain.mjs`、`skills/`）；dsh 侧 `repos/dsh-toolkit/src/nano-flow/` 按能力分六目录（assembly / session / documents / planning / dream / skills），预设挂 `dsh-tool-executor-ptc`（[011](dsh/adr/011-预设挂PTC执行器.md)）。
- 交付工作流：[pi 001 安装并真机验收](pi/workflows/001-安装pi-toolkit并真机验收/001-安装pi-toolkit并真机验收.md)、[dsh 001 发布并真机验收](dsh/workflows/001-发布dsh-toolkit并真机验收/001-发布dsh-toolkit并真机验收.md)（含 nano-flow 侧走查清单）、[etf 002 估值数据接入](etf/workflows/002-估值数据接入/002-估值数据接入.md)、[music 001 部署曲库与站点](music/workflows/001-部署曲库与站点到服务器/001-部署曲库与站点到服务器.md)、[common 001 Vite+ 大版本升级](common/workflows/001-vite-plus大版本升级/001-vite-plus大版本升级.md)、[common 002 拆分单仓为多个独立仓](common/workflows/002-拆分单仓为多个独立仓/002-拆分单仓为多个独立仓.md)。
- 常见坑：[dsh 问题索引](dsh/questions/index.md)——恢复只读、预设出现但工具缺失、历史加载失败、PATH 上的 dsh 装错 profile、宿主悬浮提示的锚点与宽度、dream 游标路径、scan 的「需要留意」、Windows 上单测假红的平台假设；[etf 问题索引](etf/questions/index.md)——上游 `count` 截断历史、Windows 上 Vite 监听临时目录把 dev 撞崩、E2E 单点失败连带后续旅程假红；[wiki 问题索引](wiki/questions/index.md)——MDX 把 `doc:` 后缀吃成指令、本地起站只监听 `::1`、`doc:` 写错 build 不报错、导图样式两份副本；[common 问题索引](common/questions/index.md)——私有仓制品装不上、bsk 操作 GitHub 失败、发布包依赖、覆盖率门槛、提交时文件被改写、两包校验规则与随包 skill 的同源程度；[pi 问题索引](pi/questions/index.md)——轨迹残留旧目录、overlay 格子底色、commandcode 403 套餐权限；[music 问题索引](music/questions/index.md)——音频 e2e 的墙钟与真浏览器、node 环境下 zustand 持久化断言；[nano-flow 问题索引](nano-flow/questions/index.md)——受管 skill 不更新、`nnf` 在 bash 里找不到。

## 用法

新会话先确认目标与所属域；`/ask`、`/do` 两个流程 skill 已随 pi 删除（2026-11-13），目标或方案没定就直接问清再动手。过程结论写 note，稳定结论等 dream 上浮到领域文档，别在两处重复定义。

## 篇幅限制

超过上限时按顺序处理：

1. **压缩**：只删 note、领域文档、代码或命令能重新查到的内容，并在删除处留指针。
2. **拆分**：压完仍有事实无处可查时，拆成各自新编号的文件（`NNN-…`），不用 `note-1.md` 这类分片名。
   - spec 不拆分：细节下沉到 note 或领域文档，`spec.md` 留相对链接。
   - CONTEXT 不拆分：上限够大，继续压缩。

- spec.md ≤ 34 KB
- 单条 note ≤ 13 KB
- description ≤ 144 字
- CONTEXT.md ≤ 89 KB
- question ≤ 13 KB
- ADR ≤ 8 KB
- workflow ≤ 34 KB
- GUIDE.md ≤ 13 KB
