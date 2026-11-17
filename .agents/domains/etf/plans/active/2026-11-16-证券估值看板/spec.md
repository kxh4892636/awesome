---
name: "2026-11-16-证券估值看板"
description: "在看板增加 PE、PB、PS 与股息率四项估值指标：先确认上游口径与历史深度，落存储与接口，再做卡片与曲线。"
status: in_progress
---

# 证券估值看板

## 目标

1. 看板展示每个证券的 PE、PB、PS 与股息率，数据只来自一个权威来源，并标注口径日期。
2. 有历史数据时，给出指标走势与历史分位，支持纵向判断高低。
3. 复用现有单应用分层（client / server / shared）与按证券持久化的看板视图状态，不新增应用或数据源。

## 范围

- 指标：市盈率（PE）、市净率（PB）、市销率（PS）、股息率，具体口径与字段由 001 取证后定。
- 展示：切换证券显示当前值；有历史时提供曲线与分位。
- 数据：服务端落 SQLite 缓存，查询优先读缓存，缺段或过期再取上游；保留数据日期。
- 不做：多证券横向对比表、估值预警与推送、投资建议。

## 已知事实

- `repos/etf` 是单一应用（client / server / shared），Hono RPC，SQLite + Drizzle；日线缓存为 `daily_bars` 与 `trading_calendar`，看板状态按证券存在 `etf:dashboard:views:v1`。
- 上游「红色火箭」目前只接了 K 线 `/fundex-quote/line/kline`；其网页能展示 PE、PB、股息率与分位，但估值接口路径、字段与历史深度尚未取证。
- 覆盖率门槛 80%，E2E 六组，构建走 Vite+。

## 关键未知

- 上游是否有可调用的估值接口、是否覆盖 PS、更新频率（日更 / 财报期）与历史起止；若只给当前值，分位与曲线只能本地按日累积，短期不可回填 —— 由 001 先答，再决定 005 的形态。
- 口径定义：PE 用 TTM 还是静态、PB 用最新报告期净资产、股息率按近 12 个月分红还是上一年度。展示必须标注，不能混用。

## 非范围

- 不改 K 线、均线与虚拟均线的既有行为与契约。
- 不接付费数据源；上游缺失某项指标时先记录并和用户确认，不私自换源或估算。

## 风险

- 历史太浅会让分位失真；若 001 结论为「仅当前值」，005 降级为本地累积曲线，需要用户确认。
- 财报更新会让估值跳变，缓存必须带数据日期与口径，界面要显示，否则读者会误读。
- 新增表与同步要迁移和幂等，且不能拖慢或阻塞日线同步。

## 恢复入口

1. 先读 [notes_index.md](notes_index.md)，按 001 → 006 推进。
2. 关键文件：`server/libs/hongsehuojian/`、`server/market/{handler,service,repository}`、`client/features/market-dashboard/`、`migrations/`。

## 进度

- 2026-11-16：Plan 建立并绑定；用户要求先只建 plan，不做实现调研。
- 2026-11-16：001 完成。实测结论：匿名可调、`timeInterval=max` 才有数据、PE/PB/PS 为倍数、DID 为百分数、ETF 走该通道无数据；契约与词汇已落 CONTEXT。
- 2026-11-16：UI 方向定为「估值卡片：左四值 + 右曲线 + 分位刻度」（Hallmark 预检推荐 B），分位用中性色与位置表达，不与涨红跌绿混用；估值曲线范围独立于 K 线周期，分位标注「成立以来」。
- 2026-11-16：用户补齐估值数据源：`/fundex-quote/index/valuation?securityCode=…&valuationType=DID&timeInterval=max`，样例响应在仓库根 `test.json`；要求按 /ap 做主链路实现、按 /hallmark 设计 UIUX。001 转 in_progress，先确认字段、单位、历史深度与鉴权依赖。
- 2026-11-16：002/003 完成：`security_valuation` 长表 + 迁移 0002、上游 client、repository、`sync:valuation` 回填（930955.CSI 4×2299 行，2017-05-26 → 2026-11-13）、`GET /api/valuations`；`pnpm test` 202 例全绿。
- 2026-11-16：004/005 完成：看板估值卡片（四格读数 + 口径日期 + 成立以来分位刻度）与分位曲线（共享时间轴、1/3/5 年与全部、指标可点选显隐），视图状态按证券持久化并向后兼容旧 storage。证据：`pnpm test` 221 例全绿、`tsc --noEmit` 通过、sloplint 0 FAIL / 0 REFLEX / 0 WARN（`--scope app --render --url`），截图在 `.hallmark/valuation/`。
- 2026-11-16：007 完成：etf 域新增 [002-估值数据接入](../../../workflows/002-估值数据接入/002-估值数据接入.md)（取数/入库/回填/接口/9 条坑），CONTEXT「估值指标」词条挂链。取证：`pnpm run sync:valuation` 退出码 0、930955.CSI 4×2299（2017-05-26→2026-11-13）、932315.CSI 4×571（2024-07-08→2026-11-13）、库内 11480 行；`/api/valuations?symbol=930955.CSI` 561694 字节。任务给的「561 KB = 2 指数 × 4 指标」与实测不符（单只证券即 561 KB），文档按实测写。
- 2026-11-16：006 完成：估值 fixture 与 S7 旅程上线（12 点/指标、末行 2025-06-03），S5 悬停前先滚动画布；README 修掉 `apps/etf` / 根 `dev:etf` / `workflows/0001` 三处陈旧表述并补估值接口与 `sync:valuation`，CONTEXT 估值词汇核对一致未改。证据：`pnpm test` 221 例、`typecheck:e2e` 通过、`test:e2e` 7 组全绿（25.6s），截图 `e2e/evidence/s7-valuation.png`。

## 追踪 Note

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status | Reason | Next Step | Related Note |
| --- | --- | --- | --- | --- | --- |
| [008-估值区改为K线下方双面板](008-估值区改为K线下方双面板.md) | 估值区移到 K 线下方，改成与 K 线一致的双面板：上为范围与四指标选中态，下为所选指标的分位折线。 | pending | — | 派前端按双面板重构估值区，并同步状态、测试与 E2E | 004-看板估值卡片, 005-估值曲线与视图状态, 006-E2E与文档收口 |
