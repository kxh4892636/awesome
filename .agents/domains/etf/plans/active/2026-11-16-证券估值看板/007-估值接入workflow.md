---
name: "007-估值接入workflow"
description: "把 PE、PB、PS、DID 的取数、口径与踩坑按实际实现整理成 etf 域 workflow，并在 CONTEXT 挂链。"
status: completed
reason:
related_noted: ["002-估值存储与全量回填", "003-估值查询接口", "006-E2E与文档收口"]
---

# 007-估值接入workflow

## 触发

用户要求：估值数据（PE / PB / PS / DID）成功接入之后再补 workflow，不在实现前预写。

## 交付

- 新增 [002-估值数据接入](../../../workflows/002-估值数据接入/002-估值数据接入.md)（`.agents/domains/etf/workflows/002-估值数据接入/002-估值数据接入.md`，9424 字节）：触发条件 / 前提 / 取数（接口、四个 `valuationType`、匿名、`timeInterval=max`、单位换算、体积）/ 入库（`security_valuation` 长表、幂等 upsert、口径日期裁剪）/ 回填 `sync:valuation` 与实测输出 / `/api/valuations` 与验证命令 / 已知坑 9 条 / 失败处理 / 代码与决策 / 相关。
- [CONTEXT.md](../../../CONTEXT.md) 的「估值指标」词条加一处相对链接指向该 workflow，未新增重复定义。
- README 估值章节已覆盖命令与单位，未再改。

## 取证（2026-11-16，全部实跑）

| 项 | 实测 |
| --- | --- |
| `cd repos/etf && pnpm run sync:valuation` | 退出码 0；930955.CSI（红利低波100）四指标各 2299 行，2017-05-26 → 2026-11-13；932315.CSI（中证红利质量）各 571 行，2024-07-08 → 2026-11-13 |
| 库内合计 | `SELECT symbol,metric,COUNT(*) ... GROUP BY` 返回 8 行、合计 11480 行，重跑不变 |
| `GET /api/valuations?symbol=930955.CSI` | 200，`RawContentLength=561694`，`pe.points[0].dataDate=2017-05-26` |
| 上游单指标体积 | PE 209355 / PB 209167 / PS 204245 / DID 204266 字节（各 2300 行） |
| 空数据形态 | `timeInterval=5Y` 与无效 `valuationType` → `data:null`；`515070.SH`（ETF 走 index 通道）→ `data:{}` |
| 上游行序 | `items=2300`，`items[0]=20261116`（当日未完成）、末项 `20170526`：降序且含当日，故入库 2299 行 |

## 与任务口径的偏差

任务给的一条事实写作「响应约 561 KB（2 个指数 × 4 指标全历史）」，与实测不符：561694 字节是**单只证券**四指标全历史（930955.CSI）；两只证券各拉一次约 2×561 KB，上游原始流量约 820 KB/证券。文档按实测写，未沿用原表述。

## 验收

- 文档事实逐条对得上代码（`valuation.ts` 的 `scale=100`、`isEmptyObject`、`/valuation` + `timeInterval=max`；`valuation-service.ts` 的 `expectedValuationDate` 与裁剪；迁移 0002 的主键）与运行结果。
- 三条 `## 相关` 链接均为相对 `.md` 路径并带原因。
- `workflows/index.md` 未手写，由 `nano_flow_check` 生成。
- `nano_flow_check(domain: "etf")` 连跑：第一次自动修复 1 项（spec 状态按 note 派生 pending → completed）并重建 3 个索引（notes_index、spec 追踪表、workflows/index.md）；之后又跑两次均为 0 项修复。
