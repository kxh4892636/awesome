---
name: '007-commandcode只上架DeepSeek并默认开思考'
description: 'Command Code provider 只上架 DeepSeek 并全部走 openai-completions，思考默认开、强度用 reasoning_effort（off 也走该字段）；能力由代码常量派生，能力不回写快照。'
status: 'accepted'
---

# commandcode 只上架 DeepSeek 并默认开思考

## 背景

[2026-11-09-commandcode-provider接入](../plans/archived/2026-11-09-commandcode-provider接入/001-取证与方案定案.md) 接入时选的是「全量 96 条自动生成、Claude 走 Anthropic 协议」：11 条 `claude-*` 用 `anthropic-messages`，其余 85 条用 `openai-completions`。

真机验收暴露两件事：账号只开通低档套餐，Claude 全系与 `gpt-6-astra` 一律 403 `MODEL_NOT_IN_PLAN`（[question 003](../questions/003-commandcode模型返回403.md)）；目录里真正能跑的是开源那批 DeepSeek 模型。于是 2026-11-10 用户改要求：只取 DeepSeek，且默认按思考模型配置。

## 决策

- **只上架 DeepSeek**：`parseCatalog` 在入口按厂商过滤（`deepseek/<model>` 或裸 `deepseek-*`），`restoreStored` 离线恢复快照时同样丢弃他厂条目。实测目录 97 → 7 条。
- **协议收窄**：删掉不可达的 Anthropic 分支，7 个模型全部 `openai-completions` + `https://api.commandcode.ai/provider/v1`。
- **能力是代码常量**：删掉「按 id 猜能力」的规则表，改 `deepSeekCapability()` 一处常量——`reasoning: true`、`thinkingLevelMap`（off→off、minimal/low→low、medium/high/xhigh→high、max→max）、`compat.supportsReasoningEffort` 与 `requiresReasoningContentOnAssistantMessages`、`input: ["text", "image"]`。
- **只发标准 OpenAI 字段**：不设 `thinkingFormat`，因此不发 DeepSeek 原生的 `thinking.type`——实测该字段被网关忽略（发 `disabled` 仍产出完整 reasoning）。关闭思考走 `reasoning_effort: "off"`，它是网关 400 校验时列出的合法枚举之一。
- **能力派生、不落盘**：快照只存目录与价格，`fromStored` 用同一份常量重推导，旧快照离线恢复后也获得思考能力。

## 后果

- 目录从 97 条缩到 7 条；以前选过 Claude/GPT/Qwen 的会话刷新后要回 `/model` 重选（README 记了这条迁移提示）。
- 7 个模型都标了 image，是用户明确决定：厂商模型表只对 `deepseek-v4-flash-vision-exp` 与 `deepseek-v4.1-flash` 写了 vision，其余 5 个贴图可能被上游拒绝。
- `reasoning_effort` 的档位差异在同题对照里噪声大于差异（low 36/235 字、high 322/215 字、max 336/342 字），但字段被网关 400 校验过，属「接受并生效」而非静默丢弃。
- 放弃多厂商目录：与 2026-11-09 参考 Plan 的「全量 96 条」相反，换来的是不必再维护双协议与按 id 猜能力的两处漂移源。

## 已排除的备选

- **维持全量 96 条 + 双协议**：能列出的模型里绝大多数当前账号跑不了，还长期背负 Anthropic/OpenAI 两条路径的差异维护。
- **按 id 关键字猜能力（`CAPABILITY_RULES`）**：任何新模型都要补一条规则，猜错就是 400；只留 DeepSeek 后每种能力都能用一条常量表达。
- **用 `thinking.type` 关闭思考**：实测被网关忽略，`reasoning_effort: "off"` 才是生效路径。

## 相关

- [011-provider只上架DeepSeek并默认开思考](../plans/archived/2026-11-09-审查修复pi-toolkit/011-provider只上架DeepSeek并默认思考.md) — 决策来源、真实网关 16 次请求的档位实测与门禁结果。
- [003-commandcode模型返回403](../questions/003-commandcode模型返回403.md) — 促成收窄的套餐权限证据（403 属权限而非路径）。
- [001-取证与方案定案](../plans/archived/2026-11-09-commandcode-provider接入/001-取证与方案定案.md) — 被本决策取代的原始端点表与凭据链路取证。
