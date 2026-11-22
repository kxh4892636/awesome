# Notes Index

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Related Note |
| --- | --- | --- |
| [001-轮转内部改用脱离调用方的信号](001-轮转内部改用脱离调用方的信号.md) | redirect 进入轮转后用独立 signal，断连不再跳过压缩或吞掉用户消息。 | — |
| [002-发版0.5.79到web-profile](002-发版0.5.79到web-profile.md) | 按 dsh 001 工作流涨版本、构建打包、装进 web profile，重启后核对版本与轮转行为。 | — |
| [003-轮转期间立即受理用户消息](003-轮转期间立即受理用户消息.md) | 轮转不再扣住用户 prompt RPC：先回 accepted 让客户端清空草稿，再在后台跑检查点与压缩，最后放行消息。 | — |
| [004-发版0.5.80到web-profile](004-发版0.5.80到web-profile.md) | 把提前受理的轮转改动涨 patch 发版，换装进 web profile。 | — |
