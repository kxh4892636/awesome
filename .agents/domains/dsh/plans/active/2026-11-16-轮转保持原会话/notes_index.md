# Notes Index

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Related Note |
| --- | --- | --- |
| [001-摸清会话日志与活体上下文的边界](001-摸清会话日志与活体上下文的边界.md) | 查清 DSH 会话是「内存事件日志 + 派生表面 + append-only 落盘」，因此改 jsonl 不降上下文；可行路径是 compaction/prune + surfaceOp replace 原地遮蔽，结论取 B。 | 002-对照pi的上下文清空实现 |
| [002-对照pi的上下文清空实现](002-对照pi的上下文清空实现.md) | pi 的「不换会话清空」是写一条 reset 自定义条目 + 每次请求在 context 钩子里过滤消息；DSH 写死 Model-visible means logged、无 per-request 钩子，只能换成持久的 surface replace。 | 001-摸清会话日志与活体上下文的边界, 003-实现原会话内重置轮转 |
| [003-实现原会话内重置轮转](003-实现原会话内重置轮转.md) | dsh-toolkit 轮转改为原会话内遮蔽历史：compaction/prune + surfaceOp replace 注入 seed，删掉归档/删除旧会话的连带动作；单测 369 条、lint 与构建通过。 | 001-摸清会话日志与活体上下文的边界, 002-对照pi的上下文清空实现, 004-发布并真机验收轮转 |
| [004-发布并真机验收轮转](004-发布并真机验收轮转.md) | 0.5.54 已装入 web profile 并通过真机验收：重置改为原会话内遮蔽历史，会话 id 不变、GUI 历史在、子 agent 与后台 job 在重置瞬间存活；宿主重启后重放折叠无异常。 | 003-实现原会话内重置轮转 |
| [005-拆除轮转死代码](005-拆除轮转死代码.md) | rotation 路由与客户端 rotation-follow 属旧轮转遗留的死路径，已拆除并随 0.5.55 发布；测试 370→357 条、check 与 build 通过，README 描述同步对齐。 | 004-发布并真机验收轮转 |
| [006-撤回在同一会话内完成](006-撤回在同一会话内完成.md) | 撤回工具现在走新建会话，与轮转旧做法同源；调研能否复用「原会话内遮蔽历史」的思路，让撤销不换会话、不杀子 agent 与后台 job。 | 003-实现原会话内重置轮转, 005-拆除轮转死代码, 007-对照dsh-recall的撤回实现 |
| [007-对照dsh-recall的撤回实现](007-对照dsh-recall的撤回实现.md) | 第三方插件 dsh-recall v0.3.0 用「一条 system/message 墓碑 + 客户端 visibility:hidden」在不改宿主的前提下实现就地撤回；本 note 记其机制、依赖的宿主内部行为与移植清单。 | 006-撤回在同一会话内完成, 003-实现原会话内重置轮转 |
