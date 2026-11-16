# Notes Index

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Related Note |
| --- | --- | --- |
| [001-摸清会话日志与活体上下文的边界](001-摸清会话日志与活体上下文的边界.md) | 查清 DSH 会话是「内存事件日志 + 派生表面 + append-only 落盘」，因此改 jsonl 不降上下文；可行路径是 compaction/prune + surfaceOp replace 原地遮蔽，结论取 B。 | 002-对照pi的上下文清空实现 |
| [002-对照pi的上下文清空实现](002-对照pi的上下文清空实现.md) | pi 的「不换会话清空」是写一条 reset 自定义条目 + 每次请求在 context 钩子里过滤消息；DSH 写死 Model-visible means logged、无 per-request 钩子，只能换成持久的 surface replace。 | 001-摸清会话日志与活体上下文的边界, 003-实现原会话内重置轮转 |
| [003-实现原会话内重置轮转](003-实现原会话内重置轮转.md) | dsh-toolkit 轮转改为原会话内遮蔽历史：compaction/prune + surfaceOp replace 注入 seed，删掉归档/删除旧会话的连带动作；单测 369 条、lint 与构建通过。 | 001-摸清会话日志与活体上下文的边界, 002-对照pi的上下文清空实现, 004-发布并真机验收轮转 |
| [004-发布并真机验收轮转](004-发布并真机验收轮转.md) | 0.5.54 已打包并装入 web profile，静态与构建门禁全过；真机验收需要重启宿主后按清单跑一轮重置，确认会话 id 不变、子 agent 与后台 job 存活。 | 003-实现原会话内重置轮转 |
