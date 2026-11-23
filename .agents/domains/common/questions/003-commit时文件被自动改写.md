---
name: '003-commit时文件被自动改写'
description: '提交钩子会在 commit 前对暂存文件执行 vp check --fix 并重新暂存。'
---

# 003-commit时文件被自动改写

## 适用条件与现象

`git commit` 输出 `Running tasks for staged files… vp check --fix`，提交后工作区文件或暂存内容发生变化。

## 原因

当时根 `vite.config.ts` 的 `staged: { "*": "vp check --fix" }` 在提交前对暂存文件执行格式化与 lint 修复，并重新暂存。

> 纠正（2026-11-22，dream 复核）：容器仓已无 `vite.config.ts`（[ADR 002](../adr/002-容器仓只做工作区管理.md)），九仓 `.git/hooks` 没有任何自定义钩子、也没有 `.husky`，所以现在提交不会再自动改写。若某天又出现改写，按同一逻辑查该仓 vite-plus 配置里的 `staged` 段。

## 解决方法

以钩子修复后的结果为准；提交后再看一次 `git status` 确认没有残留改动，不要重复手动格式化。需要保留未格式化内容时不要暂存该文件。

## 验证

`git show --stat HEAD` 与 `git status --short` 一致，且没有未暂存的额外改动。

## 来源

[common ADR 002](../adr/002-容器仓只做工作区管理.md)（容器仓已无根配置）；实测 2026-11-22：容器根只有 `.agents/`、`scripts/`、`workspace.yaml` 与说明文件，九仓 `.git/hooks` 只剩 `*.sample`。
