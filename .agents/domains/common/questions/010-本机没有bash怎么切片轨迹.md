---
name: '010-本机没有bash怎么切片轨迹'
description: '随包 dream 文档让人用 bash + python3 切片 log.jsonl，本机只有 pwsh；改走 here-string 管道喂 python3，dsh 轨迹的 seq 是区间二元组。'
---

# 010-本机没有bash怎么切片轨迹

## 问题

两包随包的 dream 文档都让人用 `bash` 切片轨迹：

- dsh：`repos/dsh-toolkit/src/nano-flow/skills/dream/reference/轨迹.md`，给的是 `@bash … <<'PY'` 起手式。
- pi：`repos/pi-toolkit/src/nano-flow/prompts/dream.md`，同样写 `bash` 加 `python3`。

本机（Windows）没有 `bash`：`Get-Command bash` 无结果，`@bash` 也不是本会话能用的工具。照抄示例会先卡在起手式，误判成「轨迹读不了」。

## 答案

`python3` 可用（`C:\Users\kxh\.local\bin\python3.exe`），shell 换成 pwsh，脚本走 here-string 管道喂给 `python3` 的 stdin：

```powershell
$p = '.agents/domains/dsh/plans/reference/<plan>/log.jsonl'
@'
import json, sys
for line in open(sys.argv[1], encoding='utf-8'):
    try: r = json.loads(line)
    except Exception: continue
    if r.get('type') != 'turn': continue
    print('## turn', r.get('turn'), r.get('seq'))
    print('USER:', (r.get('user') or '')[:2000])
    print('ASSISTANT:', (r.get('assistant') or '')[:4000])
'@ | python3 - $p
```

要点：

- pwsh 的 here-string `@'` … `'@` 必须顶格收尾，脚本里不要再带 `<<'PY'` 那套 bash 写法。
- dsh 的 turn 记录里 `seq` 是**区间二元组** `[lo, hi]`（如 `[103, 139]`），与 scan 报的 `103..139` 对应；按 `turn` 过滤比按 `seq` 省事。
- 同一脚本对 pi 轨迹打不出东西是正常的：pi 是 `type: "run"` + `entries`（见 [dsh 012](../../dsh/questions/012-dream扫描提示跳过的记录.md)）。

## 验证

```powershell
Get-Command bash        # 无结果
python3 --version       # 有输出
```

## 相关

- [006-两包随包skill是否同源](006-两包随包skill是否同源.md) — 轨迹文档与共享 skill 一样两包各带一份（pi 侧在 `prompts/dream.md`），要改得两边同改。
- [012-dream扫描提示跳过的记录](../../dsh/questions/012-dream扫描提示跳过的记录.md) — 「非 turn 记录」多半是 pi 轨迹，与本条的解析差异同源。
