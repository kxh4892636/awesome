# Notes Index

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Related Note |
| --- | --- | --- |
| [001-定案scan输入模型](001-定案scan输入模型.md) | scan 输出三类输入（增量 turn、全部 active+reference Plan、领域文档盘点），并定义无增量时的落差信号与输出形态。 | — |
| [002-扩展scan输出](002-扩展scan输出.md) | scan.mjs 收敛为 active+reference，新增存量 Plan 与领域文档盘点两段输出，无增量也不早退。 | 001-定案scan输入模型 |
| [003-改造dream提示词](003-改造dream提示词.md) | dream skill 按一个维护循环重写为 5 步，细则下沉到 reference，无增量轮次也出结论。 | 001-定案scan输入模型 |
| [004-单测与发布验收](004-单测与发布验收.md) | scan 扩展的单测全绿，0.5.78 打包装进 web profile，CLI 双路径冒烟确认无增量仍有存量与盘点输出。 | 002-扩展scan输出, 003-改造dream提示词 |
| [005-归档无用的reference-plan](005-归档无用的reference-plan.md) | dream 在维护时把 reference 中确认无用的 Plan 迁入 archived，三条判据同时成立才动。 | 003-改造dream提示词 |
