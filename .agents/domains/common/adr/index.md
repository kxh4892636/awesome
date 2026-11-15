# ADR

<!-- 由 nano_flow_check 生成，请勿手改 -->

| Name | Description | Status |
| --- | --- | --- |
| [001-发布包生产依赖不使用catalog协议](001-发布包生产依赖不使用catalog协议.md) | 生产依赖字段写显式版本，避免 npm pack 保留 catalog: 导致安装失败。 | accepted |
| [002-容器仓只做工作区管理](002-容器仓只做工作区管理.md) | 九个包从 pnpm 单仓拆成 repos/ 下各自独立的 git 仓；容器仓不再有 package.json、锁文件与共享 node_modules，一仓一锁一工具链。 | accepted |
| [003-跨仓制品走仓内tgz与raw链接](003-跨仓制品走仓内tgz与raw链接.md) | 跨仓依赖不用 npm registry、不用 Release 资产、不用 git 源码：插件把 pnpm pack 出的 tgz 提交进自己仓的 release/，消费方按 public raw URL 装。 | accepted |
