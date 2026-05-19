# Garry Tan：複雜度棘輪 × 90% 測試覆蓋率（繁體中文精解版）

> 來源：[@garrytan 推文 2054064931515855118](https://x.com/garrytan/status/2054064931515855118)
> 譯註版本：v1.0 · 2026-05-19 · vins-hub/vinshub
> 授權：以 CC BY-SA 4.0 釋出

---

## 一句話濃縮

> **AI agent 寫程式比人類快太多。瓶頸從「**生產**」搬到「**驗證**」。在 2026-2028 這 24 個月內，能贏的不是 agent 最快的公司，而是 guardrail 最強的公司——具體形式：把測試覆蓋率從 80% 拉到 90%。**

> *"The companies that win the next 24 months won't be the ones with the fastest agents...They'll be the ones with the strongest guardrails."* — Garry Tan (Y Combinator CEO)

---

## 為什麼這條改變了一切

過去 20 年，軟體業的瓶頸是「**寫不夠快**」。所以工程師最值錢的能力是寫程式速度、設計能力、解難能力。

2025 起，AI agent 已能 1 小時寫完人類 1 週的程式碼。**瓶頸不再是寫，是「**確保它沒寫壞**」**。

這就是 Garry Tan 命名的「**複雜度棘輪（Complexity Ratchet）**」：

> 每部署一個 feature，就增加依賴與 edge case，這些越來越難追蹤。當 AI agent 比人類快 10 倍時，這個棘輪轉得遠超人類驗證能力。**結果：你的 codebase 累積技術債的速度首次超過了人類有可能審查的速度。**

**Garry 的處方**：把「**90% 自動化測試覆蓋率**」當作組織級的新基線。

這份學習材料把這個處方拆解成可執行步驟。

---

## 心智模型

```
        AI Agent 時代之前                AI Agent 時代之後
        ━━━━━━━━━━━━━━━━━━              ━━━━━━━━━━━━━━━━━━━━

        生產速度                          生產速度
          ▲                                ▲ ▲ ▲ ▲ ▲ ▲  ← AI 10x
          │                                │
          │ 人類                            │ 人類驗證
          │ 驗證能力                        │ 能力（不變）
          │ ━━━━                           │ ━━━━
          ▲                                ▲
        生產速度                          ★ 缺口 = 複雜度棘輪 ★

        瓶頸：寫不夠快                    瓶頸：審不夠快
        解法：請更多工程師                解法：90% 自動化測試 +
                                              AI 寫測試
```

---

## 核心三問（CEO / Tech Lead 必答）

Garry 給組織領導層三個必須清楚回答的問題：

1. **What is our verification coverage on AI outputs?**
   「我們對 AI 輸出的驗證覆蓋率是多少？」
   
2. **Where does the ratchet bite hardest — and is that where we deployed our first agent?**
   「複雜度棘輪在哪裡咬最深？我們把第一個 agent 部署在那邊嗎？」

3. **Who owns catching regressions before they reach customers/citizens/patients?**
   「誰負責在 regression 影響到客戶 / 民眾 / 病患之前抓到它？」

**如果你公司沒有人能 5 秒內回答這三題，你已經被棘輪追上**。

---

## 為什麼是 90%（不是 80%、不是 100%）

| 覆蓋率 | 評估 |
|---|---|
| < 60% | 不能上 production。任何 PR 都可能 silent breakage |
| 60-79% | 過去 10 年業界中位數。AI 時代不夠 |
| **80%** | **舊典範的 best practice**。人類維護 codebase 的甜蜜點 |
| **90%** | **AI 時代的新基線**。AI 寫測試 ~= 免費，所以阻力消失 |
| 95-100% | Diminishing returns。維護 cost 超過邊際效益。少數高風險系統值得（醫療 / 金融 / 太空） |

**Garry 的論點**：過去 90% 不普及不是因為它技術上難，而是因為「**人類寫測試的意志力很貴**」。AI 把這個成本壓到接近零後，**90% 從「奢侈」變成「應有」**。

---

## 學習路徑

| 章節 | 內容 | 適合誰 |
|---|---|---|
| [01. 複雜度棘輪](content/01-the-complexity-ratchet.md) | 核心概念 + 為何 AI 時代是 inflection point | 所有工程 leader |
| [02. 驗證瓶頸](content/02-verification-bottleneck.md) | 從「production bottleneck」到「verification bottleneck」 | 戰略層 |
| [03. 為何 90% 是新基線](content/03-why-90-percent.md) | 數學論證 + AI 經濟學論證 | 想說服老闆的人 |
| [04. 三大組織問題](content/04-three-executive-questions.md) | 三問的展開與診斷工具 | CEO / VPE / EM |
| [05. 測試類型矩陣](content/05-test-types-matrix.md) | Unit / Integration / E2E / Property / Mutation / Fuzz | Tech Lead |
| [06. AI 當測試作者](content/06-ai-as-test-writer.md) | 用 AI 寫測試的具體 SOP | 工程師 |
| [07. 反模式：假測試陷阱](content/07-anti-patterns.md) | 80% 覆蓋率 ≠ 80% 品質 | 避免騙自己 |
| [08. 12 週實施路線圖](content/08-implementation-roadmap.md) | 從 65% → 90% 的階段計畫 | 想實際執行的人 |
| [09. 跟 12-factor agents 的關聯](content/09-relation-to-12-factor.md) | Verification 是 agent 工程化的必要條件 | 系統派 |

---

## 30 秒急救包

如果你現在沒時間讀全部，做這 3 件事：

1. **查現狀**：你的 main repo 跑 `coverage report`，看數字。如果 < 80%，這是你的第一個 sprint 目標
2. **訂目標**：在 CI 加 `--fail-under=85`（從現在的數字+5%），逐月拉到 90%
3. **指派 owner**：選一個 Tech Lead 全權負責測試品質，賦予阻擋 PR 的權限

剩下的可以慢慢讀。

---

## 為什麼這份指南對 2026 的工程組織重要

1. **AI agent 平均產出已超越中階工程師**：寫 code 不再是 bottleneck
2. **大多數公司還用 2018-2023 的 testing playbook**：80% coverage、手動 review、定期 audit
3. **複雜度棘輪在 6-12 個月內咬死你**：當 incident 多到讓 SRE 加班，已經晚了

Garry Tan 的方法論不是 nice-to-have，是 **defensive moat for the next 24 months**。

---

## 譯註者觀察

Garry 這條推文之所以重要，是因為他是**少數同時看 YC 上百家新創 + 自己用 AI 大量寫程式**的人。他看到的不是理論，是 portfolio 公司的真實發病。

他的處方有個特點：**便宜、可量化、可立即執行**。不需要新工具、不需要新框架、不需要組織重構——只需要**把 coverage 數字當作 production 指標**。

這份指南把這個處方擴展成 12 週可執行路線圖。

---

## 接下來

➡️ [Chapter 01: 複雜度棘輪 — 為何 AI 時代是 inflection point](content/01-the-complexity-ratchet.md)
