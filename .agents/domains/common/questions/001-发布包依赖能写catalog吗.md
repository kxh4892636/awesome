---
name: '001-发布包依赖能写catalog吗'
description: '发布包的生产依赖字段不能写 catalog:，安装方会报 EUNSUPPORTEDPROTOCOL。'
---

# 001-发布包依赖能写catalog吗

## 适用条件与现象

在 `repos/*/package.json` 的 `dependencies`、`peerDependencies` 或 `optionalDependencies` 中写 `catalog:`；`npm pack` 出的 tarball 保留该写法，安装方报 `EUNSUPPORTEDPROTOCOL: Unsupported URL Type "catalog:"`。

## 原因

`catalog:` 只由 pnpm 自己的打包路径改写为具体版本，`npm pack` 原样写入。

## 解决方法

会被安装方解析的字段写显式版本区间（如 `^2.9.0`）；`devDependencies` 仍可使用 `catalog:`。新增发布包时补一条 `npm pack` + `npm install` 的验证。

> 纠正（2026-11-15，common/2026-11-14-拆分单仓为多仓库 seq 2214-2344）：拆仓后容器仓不再有 catalog，`catalog:` 已无处可写，各子仓一律写显式版本（[ADR 002](../adr/002-容器仓只做工作区管理.md)）；「`devDependencies` 仍可使用 `catalog:`」只适用于拆分前。

## 验证

`npm pack` 后检查 tarball 内 `package.json` 的 `dependencies` 不含 `catalog:`；`repos/nano-flow/test/distribution.spec.ts` 覆盖该路径。

## 来源

[ADR 001](../adr/001-发布包生产依赖不使用catalog协议.md)