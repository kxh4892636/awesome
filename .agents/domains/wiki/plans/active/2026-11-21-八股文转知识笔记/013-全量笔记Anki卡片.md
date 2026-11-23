---
name: 013-全量笔记Anki卡片
description: "为 020-计算机 全树 131 篇知识笔记生成本地 Anki 卡片：一 note 一卡、17 个叶子牌组、背面 wiki UUID 短链。"
status: completed
---

# 全量笔记Anki卡片

## 目标

把 `repos/wiki/docs/020-计算机` 下所有知识笔记变成可复习的 Anki 卡片，规则与既有牌组一致。

## 交付

| 项 | 结果 |
| --- | --- |
| 覆盖笔记 | 154 个 `.md` − 23 个目录 README = 131 篇，全部有 `id`、无 `anki: false` |
| 卡片 | 131 张，模型 `基础`（字段 `正面` / `背面`） |
| 牌组 | 17 个叶子牌组，挂在 `kxh::020-计算机` 下 |
| 已有卡 | React 7、Go 7 原样保留；TCP 6 张从 `010-计算机网络` 换到 `030-计算机网络`，空牌组删除 |

## 规则

- 牌组 = `kxh::020-计算机::<相对目录路径>`，ASCII 段转小写；
- 正面 = `<目录名去编号>-<文件名去编号>-<笔记 H1>`；
- 背面 = `<a href="https://wiki.kongxiaohan.cn/<uuid>">跳转wiki</a>`；
- 一篇笔记一张卡；`anki: false` 与目录 README 跳过。

## 牌组分布

| 张数 | 牌组 |
| --- | --- |
| 25 | `010-理论基础::030-计算机网络` |
| 13 | `010-理论基础::050-计算机系统` |
| 1 | `010-理论基础::040-设计模式` |
| 15 | `020-编程语言::030-javascript` |
| 8 | `020-编程语言::020-css` |
| 7 | `020-编程语言::040-go` |
| 1 | `020-编程语言::010-html` |
| 17 | `040-系统设计::002-存储与建模` |
| 11 | `040-系统设计::003-分布式数据` |
| 7 | `040-系统设计::007-设计与案例` |
| 4 | `040-系统设计::001-网络与基础设施` |
| 3 | `040-系统设计::004-架构与通信` |
| 3 | `040-系统设计::006-可靠性与安全` |
| 7 | `050-客户端::010-react` |
| 4 | `050-客户端::020-vue` |
| 3 | `030-工程化::030-构建与打包` |
| 2 | `030-工程化::050-代码治理` |

## 执行

1. `.temp/anki-build.py` 扫全树：读 frontmatter `id` 与正文首个 H1，生成 `.temp/anki/cards.json`（131 条，UUID 无冲突，无缺字段）。
2. `.temp/anki-dedupe.py` 按 UUID 与集合现有 20 张比对：新增 111、完全一致 14、同卡换牌组 6。
3. `.temp/anki-import.py` 建 17 个牌组、`addNotes` 分批写入 111 张（失败 0），`changeDeck` 移 6 张。
4. `.temp/anki-verify.py` 删空牌组 `kxh::020-计算机::010-理论基础::010-计算机网络`。

## 验收

- `deck:kxh*` 共 131 张；模型 `基础` 共 131 条；
- 17 个牌组的实际张数与目录内笔记数逐一对齐，误差 0；
- 每条背面都带唯一 UUID，与笔记 frontmatter 一一对应。

## 坑

- 该 AnkiConnect 构建要求参数嵌在 `params` 里；顶层传 `modelName` 会报 `missing 1 required positional argument`，也解释了一次「0 张卡」的误判。
- `deleteDecks` 即使牌组已空也必须 `cardsToo: true`。
- `dupe:front` 在该构建不适用（要求整数），去重改由脚本按 UUID 比对。

## 2026-11-23 清理

用户删掉 `050-客户端/020-Vue` 全目录（4 篇笔记 + README），Anki 同步：

- `findCards deck:kxh*` 131 张，按背面 UUID 与磁盘现有 127 篇比对，命中 4 张孤儿卡，`deleteNotes` 删除；
- 叶子牌组 `kxh::020-计算机::050-客户端::020-vue` 清空后 `deleteDecks(cardsToo: true)` 删除；
- 复验：`deck:kxh*` 127 张，与目录笔记数逐个牌组对齐；
- 连带修掉 `050-客户端/README.md` 里指向已删目录的链接（否则 `onBrokenLinks: throw` 会挡构建）。

## 相关

- 规则源：`repos/dsh-toolkit/src/nano-flow/skills/learning/ANKI.md`
- 笔记目录：`repos/wiki/docs/020-计算机`
