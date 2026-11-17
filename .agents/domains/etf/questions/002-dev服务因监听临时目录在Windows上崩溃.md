---
name: '002-dev服务因监听临时目录在Windows上崩溃'
description: 'Windows 上 Vite watcher 监听原子写临时目录报 EBUSY，dev 进程带崩后端。'
---

# 002-dev服务因监听临时目录在Windows上崩溃

## 适用条件与现象

在 Windows 上跑 `pnpm run dev`（`concurrently` 起 Vite + 后端），进程莫名退出、退出码 1；日志里出现 `EBUSY`，路径指向源文件旁的 hidden 临时目录，例如 `client/features/x/.index.tsx.<uuid>.tmpdir/index.tsx.tmp`。前端一崩，后端跟着死，浏览器报连接失败。

## 原因

编辑器与部分工具用「写临时文件再原子替换」保存文件，会在源文件同目录生成 `.<文件名>.<uuid>.tmpdir/<文件名>.tmp`。Vite 的 chokidar watcher 把它当源文件监听；Windows 的文件锁让 `stat`/`open` 抛 `EBUSY`，watcher 抛错未被捕获 → `vp dev` 退出 → `concurrently` 带走后端。与产品代码无关，改哪个文件都可能触发。

## 解决方法

`repos/etf/vite.config.ts` 里给开发监听加排除（只影响 dev 监听，不影响构建产物）：

```ts
server: { watch: { ignored: ["**/.*.tmpdir/**", "**/*.tmp"] } },
```

重启 `pnpm run dev` 即可。若再次出现，先看日志里的忽略路径与 `ignored` 是否对得上（模式是相对项目 root 的 glob）。

## 验证

改文件触发保存后，dev 进程保持运行：`http://localhost:5173` 与 `http://127.0.0.1:8080` 健康检查均 200。2026-11-16 实测：未加排除时连续崩两次（每次都以 `EBUSY` + 退出码 1 结束），加上后编辑期间不再崩。

## 来源

Plan `2026-11-16-证券估值看板`（session-b2019a1f-db96-4212-a7ee-b51e4a5adcb4 turn 11、12）；`repos/etf/vite.config.ts:15`。
