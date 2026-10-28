# Astra Advisor

[English](README.md) | [繁體中文（香港）](README.zh-HK.md) | [简体中文](README.zh-CN.md)

本 GPT-6 路由版本由 Chamotrans 维护，基于 [DannyMac180/astra-advisor](https://github.com/DannyMac180/astra-advisor)。

**GPT-6 Astra 负责规划，根据需要动态选择有助于交付的限定范围委派，并负责验证和验收。**

Astra Advisor 是按模型能力分配工作的 Codex 软件交付插件。你提供目标、限制和仓库背景；Astra 判断哪些独立工作适合与主会话并行，并为每项范围明确的交付成果选择受支持的原生子代理模型和推理强度（effort）。

## 云端限制

ChatGPT Work 云端的 `create_thread` 必须省略 `model` 和 `thinking`，因此目前无法保证可以任意指定模型或推理强度。Astra 默认不会在那里发出指定模型的调度请求。
只有当前工具结构提供所需控制项时，才可使用原生 Codex 子代理。

## 延伸阅读

原作者 Daniel McAteer 的 [Attention Heads](https://attentionheads.substack.com/) 探讨 AI、认知和代理工程，并以证据支持分析。
其中 **Agentic Engineering Field Notes** 系列介绍使用 AI 的实践方法。[订阅](https://attentionheads.substack.com/subscribe)后可通过电子邮件接收新文章。

## 快速开始

如果已从其他来源注册名为 `astra-advisor` 的 marketplace，请先移除该 marketplace，再添加本衍生版本。现有本地安装可以继续使用其本地来源。

在支持插件的当前版 Codex CLI 或 ChatGPT 桌面应用中安装插件。安装后新建任务，选用 GPT-6 Astra，推理强度可以采用当前 Codex 主机支持的任意选项：

~~~sh
codex plugin marketplace add Chamotrans/Astra-Advisor-GPT6 --ref main
codex plugin add astra-advisor@astra-advisor
~~~

使用以下提示开始任务：

~~~text
Use $astra-advisor:orchestration to plan, build, verify, and review this work.
~~~

## 路由如何运作

Astra 在主 GPT-6 Astra 会话中担任架构设计者和验收负责人，推理强度由用户选择。
完成能力预检后，Astra 会在实现或委派前，将主模型和推理强度记录为“已观察”或“无法观察”。Skill 不会更改主会话。

当委派有助于交付时，Astra 使用工具提供的通用 `collaboration.spawn_agent`，明确指定 `model`、`reasoning_effort` 和 `fork_turns: none`。
它根据风险、上下文和独立工作需要，在 `gpt-6-astra`、`gpt-6.1-sol`、`gpt-6.1-terra`、`gpt-6-sol` 和 `gpt-6-luna` 之间选择。
本插件没有预定义的角色 TOML、配套安装程序、固定角色与模型映射，也没有固定的子代理数量上限。Astra 会为每个子代理指定具体且范围明确的交付成果，并在其运行期间继续处理有用的主代理工作。

以实时工具元数据为准。目前记录的推理强度快照如下：

| 模型 | 已知推理强度 |
|---|---|
| `gpt-6-astra` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6.1-sol` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6.1-terra` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6-sol` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6-luna` | `low`、`medium`、`high`、`xhigh`、`max` |

Terra 6.1 可用于限定范围的代码修复、视觉推理、计算机操作及根据所提供材料写作。
其列出的推理强度是在 2026-10-27 于 Codex 中观察到的；API 的推理强度设置有所不同，请始终以实时原生工具结构为准。

如果所选模型、推理强度、控制项或工具不可用、有冲突或无法观察，Astra 会停止该委派并说明限制，不会静默改用其他模型、推理强度、角色或虚构工具。所选设置与运行时确认的设置会分别报告。

对于具有一定规模的实现，Astra 会检查完整差异并重新运行所要求的检查，然后将累积的变更交给全新的只读审核者复核。
审核者可以使用上述五个受支持模型中的任意一个，配以实时工具支持的推理强度。只有审核者返回 `ship`，Astra 才会验收；`fix-first` 需要重新进行主代理验证和全新审核，`rethink` 则需要修订计划。

## 运行状态与成本记录（0.2.0）

每次委派都会说明名称、限定范围的任务、所选模型和推理强度，以及选择理由。
结果会报告实际状态及运行时观察到的设置；如果无法观察，也会明确说明。全新审核者同样需要提供这些更新。请求的设置不等于已确认的运行设置。

每个任务结束时均提供 API 等值成本记录。如果原生工具提供 token 用量，就会使用已版本化的价格快照估算美元成本，并与同一 token 工作量全部按 Astra 价格计算的结果比较。
记录会区分整个任务、仅委派部分及部分覆盖范围。缺少主代理或审核者用量时，不得声称覆盖整个任务；没有观察到用量时，会说明无法计算的原因。

差额是**相同 token 用量的 API 价格比较**，不是全程使用 Astra 的实际用量、任务实际净节省、质量、速度，也不是 ChatGPT 订阅费用或使用额度的变化。没有子代理就没有委派节省；推理强度不会作为 token 单价的乘数。

在提供新的、经验证的价格快照前，当前模型价格均不可用。计算器默认使用不含价格的[未验证价格状态](plugins/astra-advisor/pricing/2026-10-05-unverified.json)。
已归档的[九月快照](plugins/astra-advisor/pricing/2026-09-04.json)仅供历史账目测试数据使用，不得将旧价格重新标记为新模型价格。

可以尝试已明确标注的示例工作量（并非本次任务的成本记录）：

~~~sh
python3 plugins/astra-advisor/scripts/cost_receipt.py plugins/astra-advisor/examples/illustrative-usage.json
sh plugins/astra-advisor/scripts/verify.sh
~~~

计算器输出 JSON，并接受 `--pricing PATH` 以使用另一份经验证的价格快照。
输入包含代理和唯一的原子调用、用量来源、覆盖范围声明，以及明确的价格适用资格。它会验证缓存输入和推理输出的子集，拒绝重叠的汇总数据，并区分未知用量与零用量。
输入规范和成本记录政策详见[操作参考](plugins/astra-advisor/skills/orchestration/references/operations.md)（英文）。

## ChatGPT 应用任务

新建独立的应用任务需要用户明确要求。对于用户明确要求的 Codex 应用任务，`mcp__codex_app__create_thread` 支持 `model` 和 `thinking`。
如果目标为项目，必须先调用 `mcp__codex_app__list_projects`；默认使用本地，仅在明确要求时才使用 worktree。云端 `create_thread` 省略这两个控制项，因此仍受上述限制。
不得用 API 密钥、嵌套 CLI 或虚构工具绕过限制。

## 更新

更新本衍生版本并重新安装插件，再新建任务。如果将 marketplace 改回上游仓库，本版本的 GPT-6 路由会被替换。

~~~sh
codex plugin marketplace upgrade astra-advisor
codex plugin add astra-advisor@astra-advisor
~~~

如需本地开发，可以将此 checkout 安装为 marketplace：

~~~sh
cd /absolute/path/to/astra-advisor
codex plugin marketplace add /absolute/path/to/astra-advisor
codex plugin add astra-advisor@astra-advisor
~~~

操作详情请参阅[编排流程操作指南](plugins/astra-advisor/skills/orchestration/references/operations.md)（英文）。

## 隐私与参与贡献

提交 commit 时，请使用公开账号名称和 GitHub 提供的 noreply 电子邮件地址。
分享补丁、issue、截图或日志前，须移除个人路径、联系信息、客户材料、凭据和会话记录。公开分享的示例和成本记录必须使用模拟数据，不得使用真实会话记录。
忽略规则有助于防止误加文件，但不会清理已跟踪文件或 Git 历史。公开的项目署名和 MIT 版权声明予以保留。详见 [LICENSE](LICENSE)。
