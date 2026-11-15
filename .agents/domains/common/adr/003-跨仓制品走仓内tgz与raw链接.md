---
name: '003-跨仓制品走仓内tgz与raw链接'
description: '跨仓依赖不用 npm registry、不用 Release 资产、不用 git 源码：插件把 pnpm pack 出的 tgz 提交进自己仓的 release/，消费方按 public raw URL 装。'
status: accepted
---

# 003-跨仓制品走仓内tgz与raw链接

拆仓后 wiki 仍要拿到两个 docusaurus 插件（link / mindmap），但工作区已经没有 `workspace:*` 链接。本机 `npm` 未登录、`@kxh4892636/*` 在 registry 上全部 404，只能从 GitHub 拿制品。

做法：在插件仓 `pnpm pack` 出 `kxh4892636-<名>-0.1.0.tgz`，提交进该仓 `release/`（`.gitignore` 用 `!release/*.tgz` 放行），消费方依赖写 raw URL：

```json
"@kxh4892636/docusaurus-plugin-link": "https://raw.githubusercontent.com/kxh4892636/docusaurus-plugin-link/main/release/kxh4892636-docusaurus-plugin-link-0.1.0.tgz"
```

装的是构建产物：不触发构建、不需要 allowlist，`pnpm-lock.yaml` 的 specifier 与 version 都记这个 URL。

## Considered Options

- **仓内 tgz + raw URL（选定）**：零交互、匿名可拉、单独 clone 也能装；代价是改插件要重新 pack 提交，版本号写在文件名里，升版就得改 URL。
- **发 npm registry**：最标准，但要登录并占号，而本机从未发布过 `@kxh4892636/*`。
- **GitHub Release 资产**：私有仓的资产与 raw 都要登录态，pnpm 匿名拉取 404（[007](../questions/007-私有仓的tgz制品为什么装不上.md)）；浏览器上传资产又被 bsk 的控制上下文拦下，跑不通。
- **git 源码依赖（`github:<owner>/<repo>`）**：pnpm 11 会给 git 依赖装 devDependencies 并跑 `prepare`，消费方还要在 `allowBuilds` 里写带 commit 的完整 spec，插件每提交一次就得改配置；两个插件当时也没有 devDependencies，装不起来。
- **同级 `file:` 依赖**：只在 `repos/` 这套拓扑下成立，单独 clone wiki 就失败，只留作本地联调。

## Consequences

- 来源仓必须 public（匿名 HTTP 取 raw）；九个子仓已于 2026-11-14 全部转 public。
- 消费方锁的是 URL 里的版本号：插件升级要改 `package.json` 的 URL 并重新 install。
- 想改插件立刻在 wiki 生效，只能临时换 `file:../<插件名>` 并重建插件 `dist`；release URL 锁的是旧产物。

## 相关

- [002-容器仓只做工作区管理](002-容器仓只做工作区管理.md) — 拆仓后为什么没有 workspace 链接
- [007-私有仓的tgz制品为什么装不上](../questions/007-私有仓的tgz制品为什么装不上.md) — 选 public + raw 的直接原因
- [008-用bsk操作GitHub时为什么会失败](../questions/008-用bsk操作GitHub时为什么会失败.md) — Release 资产上传跑不通的现场