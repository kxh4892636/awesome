---
name: "001-定案scan输入模型"
description: "scan 输出三类输入（增量 turn、全部 active+reference Plan、领域文档盘点），并定义无增量时的落差信号与输出形态。"
status: completed
related_noted: []
---

# 001-定案scan输入模型

## 问题

scan 现在只有一个输入维度：游标之后的新 turn。没有新 turn 时输出「没有新增 turn。」，存量 Plan 只留每域一个计数（[scan.mjs:305-310](../../../../../repos/dsh-toolkit/src/nano-flow/dream/scan.mjs#L305-L310)），dream 没有任何可处理对象，于是空转。

## 定案

### 三类输入

**A. 新证据** —— 游标之后的新 turn。口径完全不变：按「Plan × 会话」比 `seq[1] > lastSeq`，每会话最多列 30 条，摘要 80 字 + 工具名。

**B. 存量对象** —— 每域 **全部** `active` + `reference` 的 Plan 目录（按目录枚举，不以 `log.jsonl` 为门槛；无轨迹标 `log 无`）。每个 Plan 给：`planPath`、lifecycle、`spec.md` 的 `status`、note 完成数/总数、Plan 目录内最新文件的 mtime、`log.jsonl` 字节数。

**C. 领域文档盘点** —— 每域给：`CONTEXT.md` 字节数与 mtime；`questions/` `adr/` `workflows/` 的条目数与最大编号；三份 `index.md` 是否存在；`GUIDE.md` 的 mtime。

### 落差信号

无增量轮次靠**一次可计算的比较**找活干，不做启发式猜测：

> 域内最新 Plan 的 mtime > 该域 `CONTEXT.md` 的 mtime → 标记「域文档可能落后」，dream 优先复核该域。

理由：Plan 是事实来源，领域文档是上浮产物；产物比来源旧，说明有结论没上浮。这条只看文件时间，不读正文，成本为零，也不会误判成「必须改」——dream 复核后没有落差就不改。

### 无增量时的行为

scan 照常输出 B、C 与落差标记；dream 对每个「可能落后」的域做复核 pass（补缺、纠错、去重），核查范围与结论写进本轮输出。**允许「核查后无改动」，不允许「没核查就结束」。**

### 输出形态（草案，002 按此实现）

```text
# dream 输入
- 工作区：<rootDir>
- 游标：<statePath>
## 新证据：<domain>（N 个 Plan）
#### <planPath>
- <sessionId>：游标 lastSeq X；新增 K 个 turn，seq 上界 Y
  - turn N（a..b）：<user 80 字>；工具 a/b/c
## 存量 Plan（active + reference，全部）
### <domain>（N 个）
- <planPath>｜active｜spec in_progress｜note 3/7｜最近改动 2026-11-19
## 领域文档盘点
### <domain>
- CONTEXT 42 KB（2026-11-18）；questions 16（≤016）/ adr 14（≤014）/ workflows 1（≤001）；index 3/3；GUIDE 有
- 域文档可能落后：是（最新 Plan 2026-11-19 > CONTEXT 2026-11-18）
```

## 取舍

- **不扫 `archived`**：迁 archived 之前未处理的增量会丢。可接受——迁 archived 是人工决定，且 reference 阶段已上浮过。
- **全部 active + reference，不用 21**：现网每域 Plan 数是个位数到二十，加个上限只是多一个配置项，没有收益。pi 的 21 保持原样不动。
- **不新增配置项**：落差信号用固定口径，先跑一轮看是否够用，不够再加。
- **游标不动**：无增量轮次没有 mark 对象；改了文档就 commit，游标不变，下次仍是「无增量维护」——这正是期望行为。

## 相关

- [003-改造dream提示词](003-改造dream提示词.md) — 本定案的行为侧由提示词落地
- [002-扩展scan输出](002-扩展scan输出.md) — 本定案的代码侧由 scan.mjs 落地
