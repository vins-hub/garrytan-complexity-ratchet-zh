# Garry Tan：AI Agent 複雜度棘輪 × 90% 測試覆蓋率（繁體中文精解版）

> 來源：[@garrytan Article 2054064931515855118](https://x.com/garrytan/status/2054064931515855118)
> 標題：**The AI Agent Complexity Ratchet: Why 90% Test Coverage Is Required**
> 系列：Garry Tan AI Explainer Series 第 7 篇
> 發表：2026-05-12 1:03 PM · 233.8K views
> 譯註版本：**v2.0**（2026-05-19 經 SELF-VALIDATION 修正後重寫）
> 授權：原文 Garry Tan · 譯註以 CC BY-SA 4.0 釋出

---

> ⚠️ **v2.0 修正聲明**
>
> v1.0 有 3 處重大錯誤：
> 1. 「**執行長必問三大組織問題**」是我虛構的，原文無此結構
> 2. 「**24 個月誰贏**」名言出自 Ali Ustun paraphrase，**不是 Garry 原話**
> 3. **複雜度棘輪定義方向錯了**——原意是「Quality floor 只升不降」，不是「技術債只增」
>
> 詳細比對：[SELF-VALIDATION.md](SELF-VALIDATION.md) · [Ch 00 原文翻譯](content/00-original-article.md)

---

## 一句話濃縮（修正版）

> **AI agent coding 寫的每行程式碼，都會「順便」寫測試 + 文件 + 評估。這 3 樣每 turn 累積進 context window，下一輪 agent 不能 regress、不能 ignore、不能降品質。Quality floor 只升不降，這就是「複雜度棘輪」——一個只能變好不能變差的系統。**

**金句（Garry 原話）**：

> *"Getting to 90% used to be a heroic effort. Now it's a Tuesday."*
> 
> 「過去達到 90% 覆蓋率是英雄壯舉，現在是星期二（隨便做做）。」

---

## 為何過去做不到 90%

「**為何 90% 不普及**」過去答案是：**人類沒這個意志力**。

工程師寫到第 14 個 edge case test 會無聊。週五下午 5 點不想再寫了。看到 gnarly integration test 就想「下次再說」。

**這不是技術問題，是人類耐力問題。**

Capers Jones 研究 10,000+ 軟體專案，發現 coverage 跟 defect removal efficiency（DRE）的關係**非線性**：

| Coverage | DRE |
|---|---|
| < 70% | 65-75% |
| **85-95%** | **92-97%** ← knee 在 85% |

從 70% 到 90% **不是 30% 改善，是一個數量級的缺陷escape 降低**。但**最後 20% 比前 70% 還難**（Mockus 等 Vista 研究）。

**過去 90% 是航太、醫材獨享的奢侈**（DO-178C MC/DC + FDA），因為人類成本太高。

---

## AI agent 改變的真正關鍵

不是「**AI 寫程式更快**」（那是 surface 觀察）。**是「AI 不感到 effort**」：

> *"They don't get bored writing the fourteenth edge-case test. They don't cut corners at 5pm on a Friday. They don't look at a gnarly integration test and think 'I'll come back to this later.'"*
>
> 「（agent）不會寫第 14 個邊角 test 就無聊。不會在週五五點偷懶。不會看到 gnarly integration test 想說『下次再說』。」

**過去阻止人類團隊停在 70% 的 effort curve，對 agent 不適用**。

**90% 從「英雄級工程」變成「星期二」**。

---

## 複雜度棘輪：精確定義（修正版）

每次 AI agent 工作 session 加 **3 樣**進 codebase：

| # | 加什麼 | 角色 |
|---|---|---|
| **1** | **Tests** | 編碼「**正確**」的定義；每次有人改 code 都跑 |
| **2** | **Documentation** | 編碼「**為何這樣決策**」的理由 + tradeoff |
| **3** | **Evaluation results** | 編碼「**品質基線**」分數 |

**下一輪 agent 工作時，這 3 樣都進 context window**。然後：

- ❌ 不能 regress below test suite——tests 會 fail
- ❌ 不能 ignore docs——它們就在 context
- ❌ 不能 ship 低於 evaluation baseline——分數記錄在那

> **Quality floor goes up with every turn. Forward-only motion. That's the ratchet.**
>
> 「品質地板每 turn 都升。只能往前走。這就是棘輪。」

**注意**：這跟「**技術債只增**」**方向相反**——技術債是「**壞東西累積**」，棘輪是「**好基線累積**」。

---

## 學習路徑

| 章節 | 內容 | 適合誰 |
|---|---|---|
| [00. 原文完整對照](content/00-original-article.md) | **Source of truth** — 英文原文 + 中文翻譯 | 想看一手資料的人 |
| [01. 複雜度棘輪精確定義](content/01-the-complexity-ratchet.md) | 3 樣每 turn 累積機制 / Forward-only / 跟 tech debt 區別 | 所有讀者 |
| [02. 驗證瓶頸](content/02-verification-bottleneck.md) | 從 production 到 verification 的歷史轉移 | 戰略層 |
| [03. 為何 90% 是新基線](content/03-why-90-percent.md) | Capers Jones / DO-178C / Six Sigma / Mockus 真實數據 | 想說服老闆 |
| [04. 三個實戰案例](content/04-three-executive-questions.md) | **(v2.0 重寫)** Holder Confusion / TTY Harness / OpenClaw Plugin | 想看 ratchet 怎麼運作 |
| [05. 測試類型矩陣](content/05-test-types-matrix.md) | 6 種測試類型 + 比例分配 | Tech Lead |
| [06. AI 當測試作者](content/06-ai-as-test-writer.md) | 6 個工作流 + Prompt 範本 | 工程師 |
| [07. 假測試陷阱](content/07-anti-patterns.md) | 10 個反模式 + 自我檢查腳本 | 避免騙自己 |
| [08. 12 週實施路線圖](content/08-implementation-roadmap.md) | 從 65% → 90% 階段計畫 | 想實際執行 |
| [09. 跟 12-factor agents 的關聯](content/09-relation-to-12-factor.md) | Building × Verification 雙翼 | 系統派 |

---

## 真正的金句（從原文）

整理 Garry 原話最值得記的 4 句：

> **1. "Tests are institutional memory that survives employee turnover."**
> 「測試是在員工離職後仍存活的組織記憶。」

> **2. "Getting to 90% used to be a heroic effort. Now it's a Tuesday."**
> 「過去達到 90% 是英雄壯舉，現在是星期二。」

> **3. "AI coding works fine. They just didn't build the ratchet."**
> 「AI 寫程式沒問題，只是他們沒造棘輪。」（針對「**vibecoding 失敗**」案例）

> **4. "The question isn't whether you can afford 90%. It's whether you can afford not to."**
> 「問題不是『**你負擔得起 90% 嗎**』，而是『**你負擔得起不做嗎**』。」

---

## 個人 credibility 數據（Garry 親身證明）

Garry 用兩個開源專案證明這套方法可行：

| 專案 | 規模 | 性質 |
|---|---|---|
| **GStack** | 93K stars, 701K LoC, 46 skills, 37 contributors | AI coding agent framework |
| **GBrain** | 14K stars, 25 contributors | Second brain for AI agents |
| **合計** | **970,000 LoC, 665 test files** | 全由 Claude Code + Codex 寫（15 個同時 Conductor session）|

**上週紀錄**：72 小時 merge 14 個 PR，29,000 行新 code，**每次 release 比上次測試更好**。

「速度跟品質要 trade off」是過去的事。**現在不用選**。

---

## 「Everything Harnessable Is Testable」（測試擴展論）

Garry 提出的重要洞察：測試**不只是 unit test**，是 5 個層次：

| 層次 | 觀察什麼 | 範例 |
|---|---|---|
| **OS** | process tree / file system / cron | migration 有沒有建對表？cron 有沒有 fire？ |
| **Terminal** | 每個 keystroke / interactive prompt | AI agent 有沒有在跑 review skill 時 ask question？|
| **Browser** | rendered page / button state / navigation | 頁面 render 對嗎？form 填對嗎？ |
| **API** | structured response / schema | model 回的 JSON schema 對嗎？|
| **Agent behavioral** | 說什麼 / call 什麼 tool / 順序 / 是否事前 ask | agent 有沒有照 protocol？刪除前有沒有確認？|

> **「If you can observe it, you can assert on it. If you can assert on it, you can ratchet it.」**

---

## 30 秒急救包（修正版）

1. **理解定義**：棘輪是「好基線只升」，不是「壞東西鎖住」
2. **問自己**：你 codebase 每個 PR 是不是都加了 (test + doc + eval)？
3. **如果不是**：去 [Ch 08 路線圖](content/08-implementation-roadmap.md) 第一週開始
4. **如果是**：去 [Ch 09](content/09-relation-to-12-factor.md) 把這跟 12-factor agents 整合

---

## 反例：Vibecoding 沒做 ratchet → 死

> *"Most vibecoded projects that skip tests start falling apart once they reach moderate complexity — a few thousand lines, a handful of interacting features."*
> 
> 「多數 vibecoded（Karpathy 提的術語）專案 skip tests，到中等複雜度（幾千行 + 幾個 feature）就解體。」

> *"By version 0.5 the codebase is a haunted house where every change breaks something unexpected."*
>
> 「V0.5 後 codebase 是『**鬧鬼的房子**』——每次改動都壞別處。」

> *"AI coding works fine. They just didn't build the ratchet."*

---

## 譯註者觀察

Garry Tan 這篇 article 是「**AI 時代 production-grade 軟體**」的最重要論述之一，但**中文社群討論度極低**。

它跟 [Dex Horthy 的 12-factor agents](https://github.com/vins-hub/12-factor-agents-zh) 跟 [Karpathy 的 +HTML 方法](https://github.com/vins-hub/karpathy-structure-as-html-zh) 構成 **2026 AI 應用方法論三大支柱**：

| 三大支柱 | 提出者 | 角色 |
|---|---|---|
| **12-factor agents** | Dex Horthy | 怎麼建造 agent |
| **+HTML output** | Karpathy / Thariq | 怎麼呈現 LLM 輸出 |
| **Complexity Ratchet + 90% coverage** | Garry Tan | 怎麼驗證 AI 產出 |

讀完這三份精解，你會擁有 production-grade AI 應用的完整方法論。

---

## 修正歷史

| 版本 | 日期 | 動作 |
|---|---|---|
| v1.0 | 2026-05-19 (16:27) | 第一版，根據 Ali Ustun LinkedIn paraphrase 寫，**有 3 處虛構** |
| **v2.0** | **2026-05-19 (17:00+)** | **用 gstack/browse 讀原文後重寫，修正核心定義 + 刪虛構章節 + 補實戰案例 + 補真實數據** |

詳見 [SELF-VALIDATION.md](SELF-VALIDATION.md)。

---

## 接下來

➡️ [Chapter 00: 原文完整對照](content/00-original-article.md) — **強烈建議先讀這個**，再讀後續詮釋

或

➡️ [Chapter 01: 複雜度棘輪精確定義](content/01-the-complexity-ratchet.md)
