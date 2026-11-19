# Questions

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description |
| --- | --- |
| [001-本地行情只覆盖最近1000根](001-本地行情只覆盖最近1000根.md) | 上游 count=-1000 截断历史，需用 sync:market 全量补历史。 |
| [002-dev服务因监听临时目录在Windows上崩溃](002-dev服务因监听临时目录在Windows上崩溃.md) | Windows 上 Vite watcher 监听原子写临时目录报 EBUSY，dev 进程带崩后端。 |
| [003-E2E单点失败连带后续旅程假红](003-E2E单点失败连带后续旅程假红.md) | Playwright 丢弃失败 worker 会杀掉它在同一 worker 里重启的后端，后续 spec 全红。 |
