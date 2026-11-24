---
name: "018-cheatsheet用html分栏经chrome出png"
description: "cheatsheet 用 Markdown 作源、HTML/CSS 分栏、Chrome headless 截 300dpi PNG；随包 marked 解析富文本与 mermaid，固定 5 栏。"
status: accepted
---

# 018-cheatsheet 用 HTML 分栏经 Chrome 出 PNG

## 背景

`learning` 要加一个 cheatsheet 路由，把整本书压成高密度多栏速查页。参考版式是 6 栏、约 6pt、Letter 横向，两页装下约 2 万字。候选渲染链路四条：

- LaTeX：排版最好，但本机没有 xelatex，CJK 要配字体与 ctex，单次编译秒级偏慢，报错对 agent 不友好；
- Typst：原生多栏、单二进制、可直接出 PNG，理论上最合适，但本机未装；
- HTML/CSS + 无头浏览器：折行、避头尾、分栏、表格都由 CSS 承担，Chrome 已装，改字重渲约 1 秒；
- Python 直接排版（Pillow / reportlab）：本机都没装，且要自己实现换行与分栏。

实测：Chrome 在 `--force-device-scale-factor=3` 下截图 1123×794 版面得到 3369×2382，约 288dpi；Noto Sans SC 已在本机。

## 决策

- 源是 Markdown（可 diff、可被 agent 直接写），PNG 是派生物，不手写 HTML/LaTeX；
- 链路固定为 Markdown → `assets/sheet.html`（CSS 变量控制栏数、字号、页尺寸）→ Chrome headless 截图；
- 分页在页面内完成：把每节拆成标题、块、列表项等单元，按单元顺序填满定高栏，栏底只剩不足一行；列表跨栏时续写成新的列表；
- 标题字号与正文解耦（`max(14px, 正文 × 2)`），页眉高度按标题算并在其下留 8px 再起栏；栏底再留 18px，内容不顶到纸边；
- `--fit <页数>` 二分字号，让内容刚好铺满指定页数；
- 固定 A4 横向 5 栏、栏距 10px、5pt、300dpi：栏宽与行宽锁在约 32 字/行；版面不开放栏数参数，页数是唯一的旋钮（配合 `--fit` 选字号）；
- Markdown 交给随包 marked（约 100 KB，MIT）解析，覆盖 CommonMark + GFM：标题、粗斜体、删除线、链接、图片、引用、嵌套列表、表格、代码块、分隔线；
- mermaid 随包（约 2.6 MB，MIT），只在源含 mermaid 代码块时注入页面；渲染完成后再测高度分页，图按实际尺寸参与排版；
- 第三方产物放 `assets/vendor/`，加入 `.prettierignore` 与文件头 lint 豁免，构建时原样拷贝。

## 后果

- 运行依赖本机 Chrome 或 Edge（脚本按常见路径与环境变量查找）；
- 排版精度受浏览器文本度量影响，跨机器可能有 1% 级差异；
- 若将来允许装 Typst，替换的只是 `render-sheet.mjs` 这一段，Markdown 约定与预算规则不动。
- 包体多约 2.7 MB；不含 mermaid 的卡不加载这份脚本，单次出图为 1~2 秒。
- mermaid 渲染失败时保留为文本框，不中断出图；分页依赖图的实际高度，图错会连带版面偏差。

## 相关

- [../../../../repos/dsh-toolkit/src/nano-flow/skills/learning/CHEATSHEET.md](../../../../repos/dsh-toolkit/src/nano-flow/skills/learning/CHEATSHEET.md) — 路由正文
- [017-pretty-doc产出飞书文档并走svg画板](017-pretty-doc产出飞书文档并走svg画板.md) — 另一条产出链，对比见决策
