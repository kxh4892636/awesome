---
name: '009-子仓的pnpm-workspace.yaml能整删吗'
description: '不能整删：九个仓的 pnpm-workspace.yaml 还装着 overrides / allowBuilds / peerDependencyRules / minimumReleaseAgeExclude，pnpm 只在它这里读这些设置；只删顶上的 packages: [] 一行。'
---

# 009-子仓的pnpm-workspace.yaml能整删吗

## 适用条件与现象

拆仓后每个 `repos/<名>/pnpm-workspace.yaml` 顶上都是 `packages: []`，看着像 monorepo 残留，想整文件删掉——「子仓已经不是 monorepo 了」。

## 原因

文件不只在声明工作区成员，还是 pnpm 设置的唯一载体：

- `overrides`：`vite@* → npm:@voidzero-dev/vite-plus-core@1.0.0`、`vitest@* → 5.0.1`、`app-builder-lib>@electron/get`
- `allowBuilds`：9 条构建脚本开关（`esbuild: false`、`electron: true` …）
- `peerDependencyRules`、`minimumReleaseAgeExclude`

pnpm 只从命令行、环境变量和 `pnpm-workspace.yaml` 读设置；`.npmrc` 只认 auth / registry / TLS / proxy / local-address，而这些仓里根本没有 `.npmrc`。整删之后：vite / vitest 解析回真包、构建脚本被 allowBuilds 拦、锁文件记录的 `overrides` 与配置不符，`pnpm install --frozen-lockfile` 判定过期。

`packages: []` 本身在 pnpm 12 下没有可观测副作用（见验证），删它只是去掉「本仓是工作区根」的语义，不该顺手把设置一起删。

## 解决方法

只删 `packages: []` 一行与紧随的空行，四段设置原样保留。不写 `packages` 键的文件在 pnpm 12 下照常提供设置，`pnpm install` 与 `pnpm add` 都正常；`pnpm-lock.yaml` 不用动。

## 验证

在子仓里：

- `pnpm config get overrides`、`pnpm config get allowBuilds`、`pnpm config get minimumReleaseAgeExclude` 都有输出（2026-11-16 实测九仓皆如此）。
- 文件顶部不再有 `packages:`；`pnpm install --frozen-lockfile` 通过，`git diff pnpm-lock.yaml` 为空。
- 反向核查：只有设置段、没有 `packages` 键的 `pnpm-workspace.yaml` 同样能 `pnpm config get` 到设置并 `pnpm install` 成功（.temp 临时目录实测，pnpm 12.21.0）。

## 来源

common `2026-11-14-拆分单仓为多仓库` seq 4-195（session-b2cf7b58，turn 1）：用户提出「子仓不是 monorepo，pnpm-workspace.yaml 没意义」，当场核查出设置段的存在；该提问的用户答复未到达，尚未执行任何删除。行号与内容按 2026-11-16 现场复核。

## 相关

- [002-容器仓只做工作区管理](../adr/002-容器仓只做工作区管理.md) — 子仓为什么要自带工具链与锁文件
- [002-拆分单仓为多个独立仓](../workflows/002-拆分单仓为多个独立仓/002-拆分单仓为多个独立仓.md) — 独立化改造时这份文件怎么生成
