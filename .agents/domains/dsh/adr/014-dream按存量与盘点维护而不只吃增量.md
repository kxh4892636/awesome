---
name: '014-dream按存量与盘点维护而不只吃增量'
description: 'dream 的输入除游标后的新 turn 外，还包括每域全部 active+reference 的存量 Plan 与领域文档盘点；无增量轮次照样复核，并可归档无用的 reference Plan。'
status: accepted
---

# 014-dream按存量与盘点维护而不只吃增量

## 背景

`nano_flow_dream` 的 scan 原本只输出游标之后的新 turn，Plan 的发现标准是 `log.jsonl` 存在，三个生命周期全扫。没有新 turn 时输出「没有新增 turn。」，存量 Plan 只在「无新增（只读）」段留每域一个计数——连路径都不给。于是没有新 turn 的轮次，dream 拿不到任何可处理对象，只能空转；领域文档的维护全靠提示词自觉。

## 决策

scan 一次给三类输入：

1. **新证据**：游标之后的新 turn，口径不变（Plan × 会话的 `seq[1] > lastSeq`）。
2. **存量 Plan**：每域**全部** `active` + `reference` 的 Plan 目录，带 spec 状态、note 完成数、目录最近改动时间、`log.jsonl` 字节数。按目录枚举，不以 `log.jsonl` 为门槛。
3. **领域文档盘点**：`CONTEXT.md` 规模与时间、`questions`/`adr`/`workflows` 条目数与最大编号、三份 `index.md` 是否齐备、`GUIDE.md` 时间，以及**「域文档可能落后」标记**（最新 Plan 的日期晚于 CONTEXT 的日期）。

`archived` 不参与扫描；没有数量上限（不沿用 pi 的「每域最新 21 个」）。dream 在没有新证据的轮次里按标记复核该域，照样出结论；并可以把 `reference` 中确认无用的 Plan 迁 `archived`（三条判据同时成立：结论已上浮、note 全 completed 且 spec completed、无唯一出处）。

## 理由

- Plan 是事实来源，领域文档是上浮产物。产物比来源旧，说明有结论没上浮——这是唯一零成本可算的信号，不需要读正文。
- 按目录枚举而非 `log.jsonl` 存在：老 Plan 大多没有轨迹（实测 29 个 active+reference Plan 里只有 8 个有 `log.jsonl`），按轨迹门槛盘点会让存量段几乎为空。
- 不设数量上限：每域 Plan 是个位数到二十，多一个配置项没有收益。
- 落差按天比较：同日几分钟的改动会把域永久刷成「可能落后」。

## 后果

- 每轮 dream 都有维护对象，「核查后无改动」是允许的终态，「没核查就结束」不是。
- `archived` 里迁入前未处理完的增量不再被扫描——迁移是人做的决定，且 `reference` 阶段已上浮过，属于已知有损。
- 落差信号只按目录时间，不看内容；它决定复核范围，不决定「必须改」。

## 相关

- [007-dream由宿主定时任务投递新会话](007-dream由宿主定时任务投递新会话.md) — 触发方式不变，变的是输入与范围
