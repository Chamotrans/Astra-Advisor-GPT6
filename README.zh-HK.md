# Astra Advisor

[English](README.md) | [繁體中文（香港）](README.zh-HK.md) | [简体中文](README.zh-CN.md)

本 GPT-6 路由版本由 Chamotrans 維護，以 [DannyMac180/astra-advisor](https://github.com/DannyMac180/astra-advisor) 為基礎。

**GPT-6 Astra 負責規劃、按需要動態選擇有助交付的限定範圍委派，並負責驗證及驗收。**

Astra Advisor 是按模型能力分配工作的 Codex 軟件交付插件。你提供目標、限制及儲存庫背景；Astra 判斷哪些獨立工作適合與主工作階段並行，並為每項範圍明確的交付成果選擇受支援的原生子代理模型及推理強度（effort）。

## 雲端限制

ChatGPT Work 雲端的 `create_thread` 必須省略 `model` 及 `thinking`，因此目前無法保證可任意指定模型或推理強度。Astra 預設不會在該處發出指定模型的調度請求。
只有當前工具結構提供所需控制項時，才可使用原生 Codex 子代理。

## 延伸閱讀

原作者 Daniel McAteer 的 [Attention Heads](https://attentionheads.substack.com/) 探討 AI、認知及代理工程，並以證據支持分析。
其中 **Agentic Engineering Field Notes** 系列介紹使用 AI 的實務方法。[訂閱](https://attentionheads.substack.com/subscribe)可透過電郵接收新文章。

## 快速開始

若已從其他來源註冊名為 `astra-advisor` 的 marketplace，請先移除該 marketplace，再加入本衍生版本。現有本機安裝可繼續使用其本機來源。

在支援插件的現行 Codex CLI 或 ChatGPT 桌面應用程式安裝插件。安裝後開啟新任務，選用 GPT-6 Astra，推理強度可採用當前 Codex 主機支援的任何選項：

~~~sh
codex plugin marketplace add Chamotrans/Astra-Advisor-GPT6 --ref main
codex plugin add astra-advisor@astra-advisor
~~~

以以下提示開始任務：

~~~text
Use $astra-advisor:orchestration to plan, build, verify, and review this work.
~~~

## 路由如何運作

Astra 在主 GPT-6 Astra 工作階段中擔任架構設計者及驗收負責人，推理強度由使用者選擇。
完成能力預檢後，Astra 會在實作或委派前，記錄主模型及推理強度為「已觀察」或「無法觀察」。Skill 不會更改主工作階段。

當委派有助交付時，Astra 使用工具提供的通用 `collaboration.spawn_agent`，明確指定 `model`、`reasoning_effort` 及 `fork_turns: none`。
它按風險、上下文及獨立工作需要，在 `gpt-6-astra`、`gpt-6.1-sol`、`gpt-6.1-terra`、`gpt-6-sol` 及 `gpt-6-luna` 之間選擇。
本插件沒有預設角色 TOML、配套安裝程式、固定角色與模型對應，亦沒有固定的子代理數目上限。Astra 會給予每個子代理具體且範圍明確的交付成果，並在其執行期間繼續處理有用的主代理工作。

以即時工具元資料為準。目前記錄的推理強度快照如下：

| 模型 | 已知推理強度 |
|---|---|
| `gpt-6-astra` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6.1-sol` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6.1-terra` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6-sol` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6-luna` | `low`、`medium`、`high`、`xhigh`、`max` |

Terra 6.1 可用於限定範圍的程式修正、視覺推理、電腦操作及按提供材料寫作。
其列出的推理強度是在 2026-10-27 於 Codex 觀察所得；API 的推理強度設定有所不同，請始終以即時原生工具結構為準。

若所選模型、推理強度、控制項或工具不可用、有衝突或無法觀察，Astra 會停止該委派並說明限制，不會默默改用其他模型、推理強度、角色或虛構工具。所選設定與執行時確認的設定會分開報告。

對於具一定規模的實作，Astra 會檢查完整差異並重新執行所要求的檢查，然後將累積的變更交由全新的唯讀審核者覆核。
審核者可使用上述五個受支援模型中的任何一個，配以即時工具支援的推理強度。只有審核者回覆 `ship`，Astra 才會驗收；`fix-first` 須重新進行主代理驗證及全新審核，`rethink` 則須修訂計劃。

## 執行狀態與成本紀錄（0.2.0）

每次委派均會說明名稱、限定範圍的任務、所選模型及推理強度，以及選擇理由。
結果會報告實際狀態及執行時觀察到的設定；若無法觀察，亦會明確說明。全新審核者同樣須作這些更新。請求的設定不等於已確認的執行設定。

每個任務結束時均提供 API 等值成本紀錄。若原生工具提供 token 用量，便會使用已版本化的價格快照估算美元成本，並與同一 token 工作量全部按 Astra 價格計算的結果比較。
紀錄會區分整個任務、僅委派部分及部分涵蓋範圍。缺少主代理或審核者用量時，不得聲稱涵蓋整個任務；沒有觀察到用量時，會說明無法計算的原因。

差額是**相同 token 用量的 API 價格比較**，不是全程使用 Astra 的實際用量、任務實際淨節省、品質、速度，亦不是 ChatGPT 訂閱收費或使用額度的變動。沒有子代理便沒有委派節省；推理強度不會作為 token 單價的乘數。

在提供新而經核實的價格快照前，目前模型價格均不可用。計算器預設使用不含價格的[未核實價格狀態](plugins/astra-advisor/pricing/2026-10-05-unverified.json)。
已封存的[九月快照](plugins/astra-advisor/pricing/2026-09-04.json)只供歷史帳目測試資料使用，不得將舊價格改標為新模型價格。

可試用已明確標示的示例工作量（並非你本次任務的成本紀錄）：

~~~sh
python3 plugins/astra-advisor/scripts/cost_receipt.py plugins/astra-advisor/examples/illustrative-usage.json
sh plugins/astra-advisor/scripts/verify.sh
~~~

計算器輸出 JSON，並接受 `--pricing PATH` 以使用另一份經核實的價格快照。
輸入包含代理及唯一的原子呼叫、用量來源、涵蓋範圍聲明，以及明確的價格適用資格。它會驗證快取輸入及推理輸出的子集，拒絕重疊的彙總數據，並區分未知用量與零用量。
輸入規格及成本紀錄政策詳見[操作參考](plugins/astra-advisor/skills/orchestration/references/operations.md)（英文）。

## ChatGPT 應用程式任務

另開應用程式任務須有使用者明確要求。若使用者明確要求 Codex 應用程式任務，`mcp__codex_app__create_thread` 支援 `model` 及 `thinking`。
若目標為專案，須先呼叫 `mcp__codex_app__list_projects`；預設使用本機，只有明確要求時才使用 worktree。雲端 `create_thread` 省略這兩個控制項，因此仍受上述限制。
不得以 API 金鑰、巢狀 CLI 或虛構工具繞過限制。

## 更新

更新本衍生版本並重新安裝插件，再開啟新任務。若將 marketplace 改回上游儲存庫，本版本的 GPT-6 路由會被取代。

~~~sh
codex plugin marketplace upgrade astra-advisor
codex plugin add astra-advisor@astra-advisor
~~~

如在本機開發，可將此 checkout 安裝為 marketplace：

~~~sh
cd /absolute/path/to/astra-advisor
codex plugin marketplace add /absolute/path/to/astra-advisor
codex plugin add astra-advisor@astra-advisor
~~~

操作詳情請參閱[協調流程操作指南](plugins/astra-advisor/skills/orchestration/references/operations.md)（英文）。

## 私隱與參與貢獻

提交 commit 時，請使用公開帳號名稱及 GitHub 提供的 noreply 電郵地址。
分享修補檔、issue、截圖或日誌前，須移除個人路徑、聯絡資料、客戶材料、憑證及工作階段逐字紀錄。公開分享的範例及成本紀錄必須使用模擬資料，不得使用真實工作階段紀錄。
忽略規則有助防止誤加檔案，但不會清理已追蹤檔案或 Git 歷史。公開的專案署名及 MIT 版權聲明會予保留。詳見 [LICENSE](LICENSE)。
