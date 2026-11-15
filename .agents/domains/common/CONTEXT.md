---
name: 'Common'
description: '由多个业务域共同贡献的基础业务域，承载跨业务域共享的基础约定。'
relations: []
---

# Common

Common 是由多个业务域共同贡献的基础业务域，承载跨业务域共享的基础约定。

## Language

**共享基础能力**：
由多个业务域共同贡献、且服务于多个业务域的基础约定；只服务于单一业务域的内容仍归该业务域所有。
_Avoid_：公共工具、杂项

**同源实现**：
同一份规则在两个包里各带一份副本（pi-toolkit 与 dsh-toolkit 的 nano-flow 校验脚本即此例）：副本必须逐字一致，改规则时两包同改；允许的差异只在入口壳与模块注释。
_Avoid_：重复代码、共享包

### 受管 skill（由 Nano Flow 贡献）

**受管 skill**：
由 CLI 包分发，并由其 `self skill` 命令安装、检测、更新或卸载的完整 skill 树；版本跟随分发它的 CLI。
_Avoid_：插件、独立 skill 包

**skill 安装状态**：
受管 skill 相对当前 CLI 包的状态：`not_installed`、`current`、`outdated` 或 `modified`。
_Avoid_：同步状态、健康状态

### CLI 执行模态（由 Nano Flow 贡献）

**真实执行**：
未启用 `--dry-run` 时的默认执行模式，允许命令产生其定义的持久化副作用。
_Avoid_：apply mode、normal mode

**预演**：
由全局 `--dry-run` 选项启用的执行模式；写命令只返回原本将执行的操作，不产生持久化副作用。
_Avoid_：模拟执行、试跑、只读模式

### 工作区形态（由拆分多仓贡献）

**容器仓**：
只做工作区与多仓管理的 git 仓（远端 `kxh4892636/awesome`，本地目录仍叫 `kxh-kit`）：根上只有 `.agents/`、`scripts/`、`workspace.yaml`、说明文件与运行目录，不放任何代码依赖。
_Avoid_：monorepo、主仓、根包

**子仓**：
`repos/<名>` 下各自带 `.git`、`pnpm-lock.yaml` 与工具链的独立仓，共九个；每个都能单独 `pnpm install` 与构建，版本各自演进。
_Avoid_：submodule、子包、workspace 包

**目录历史**：
用 `git subtree split --prefix=<目录>` 从容器仓抽出的、只含该目录提交的重写历史；提交号全新，旧分支不能沿用。
_Avoid_：迁移历史、filter-repo 历史

**工作区清单**：
根 `workspace.yaml`，逐个登记子仓的 `name` / `url` / `branch` / `path`；新机器克隆与新增子仓都以它为唯一索引。
_Avoid_：pnpm workspace、仓库列表、submodule 清单
