---
name: '018-dream扫描把单引号status漏计'
description: "scan 只剥双引号，单引号 status（合法 YAML）与 completed 比不相等，盘点里记成 note 0/N、spec 状态带引号。"
---

# 018-dream扫描把单引号status漏计

## 现象

dream 的存量 Plan 清单一行为：`- .agents/domains/pi/plans/reference/2026-11-09-pi-toolkit插件｜reference｜spec 'completed'｜note 0/17`。该 Plan 的 17 条 note 其实全是 `status: 'completed'`，spec 同样是单引号写法。

## 原因

`dream/scan.mjs` 的 `statusOf` 用 `/^status:\s*"?(.+?)"?\s*$/`，只剥双引号；单引号在 YAML 里同样合法，取出的值带引号（`'completed'`），与字符串 `completed` 比较不相等。于是 note 计数为 0、spec 状态原样带引号打印。

## 影响

scan 是盘点不是校验，所以不报错、只静默少算。dream 若据此判断「note 未完成」，会跳过本该归档的 Plan，或误判工作量；带单引号的旧 Plan（早期模板产物）都会命中。

## 判别

盘点行里 `spec` 带引号、或 note 完成数明显小于实际文件数，就去看该 Plan 的 note 头是不是单引号。

## 处理

未修（dream 只写领域文档，不动代码）。修法：`statusOf` 的正则同时剥 `'` 与 `"`，或直接按 YAML 标量解析。

## 验证

源码 `repos/dsh-toolkit/src/nano-flow/dream/scan.mjs:59-63`；实测 2026-11-20 本轮 scan（pi 2026-11-09-pi-toolkit插件）。

## 相关

- [012-dream扫描提示跳过的记录](012-dream扫描提示跳过的记录.md) — 同一段盘点输出里的另一类静默差异
