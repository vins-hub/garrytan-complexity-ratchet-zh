[← 回 README](../README.md) · [← Ch 00 原文](00-original-article.md)

# 第 01 章：複雜度棘輪精確定義（v2.0 修正版）

> **核心句**：複雜度棘輪是「**只能往一個方向動**」的機制——但這個方向是「**品質往上**」，不是「**技術債往上**」。每次 AI agent coding session 加 3 樣（tests + docs + evals）進 codebase，下一輪不能 regress 低於這個 baseline，於是 quality floor 只升不降。

---

## v1.0 錯了什麼

v1.0 把「棘輪」描述成「**部署一個 feature 就鎖住複雜度只增不減**」。

**這是錯的**。那是 **tech debt** 的定義，不是 ratchet。

Garry 原文：

> *A ratchet is a mechanism that allows motion in one direction only. A socket wrench turns a bolt forward and prevents it from turning back.*

棘輪是「**只能往前不能往後**」。**問題是「往前」是什麼方向？**

**Garry 的答案**：往「**品質更好**」的方向。

> *The quality floor goes up with every turn. Forward-only motion. That's the ratchet.*

棘輪保證的是**好的基線只能升**，不是「**所有東西都鎖住不能改**」。

---

## 精確機制：每 turn 加 3 樣

Garry 原文最關鍵的一段：

> *In agent-coded software, every coding session with an AI agent adds three things to the codebase:*
>
> *1. Tests that encode what "correct" means — automated checks that run every time someone changes the code, and fail loudly if the change breaks something*
>
> *2. Documentation that records why decisions were made — not just what the code does, but the reasoning and tradeoffs behind it*
>
> *3. Evaluation results that establish quality thresholds — structured assessments of output quality with scores, so you know if the next version is better or worse*

**翻譯**：在 agent 寫的軟體裡，**每次跟 AI agent 的 coding session 加 3 樣到 codebase**：

### 第 1 樣：Tests

**編碼「**正確**」的定義**——自動化檢查，每次有人改 code 就跑，壞了就大聲 fail。

> *"Tests encode what 'correct' means."*

注意這個詞：「**encode**」（編碼）。測試不只是「**檢查**」工具，是「**正確性規範的可執行形式**」。

### 第 2 樣：Documentation

**記錄「**為何這樣決定**」的理由**——不只是 code 在做什麼，是**背後的推理與 tradeoff**。

注意這跟一般「**註解**」的差別。一般註解講 *what*；Garry 強調的 doc 講 *why*。

### 第 3 樣：Evaluation results

**建立品質閾值的分數記錄**——有結構的輸出品質評估，附分數，**讓你知道下一版是更好或更壞**。

這比較陌生。例子：
- 用 GPT-5.5 + Claude cross-model 給某個 prompt 的輸出打分（6.8/10）
- 用 mutation testing 給測試品質打分（mutation score 78%）
- 用 LLM-as-judge 評估 chat agent 回答品質（4.2/5）

**Evaluation 是「**品質的可量化記錄**」**——能比較版本好壞，不只是「**對/錯**」。

---

## 為何這 3 樣合在一起就是棘輪？

> *The next time an agent works on the codebase, it loads all three into its context window. It can't regress below the test suite — the tests would fail. It can't ignore the documentation — it's right there in context. It can't ship quality below the evaluation baseline — the scores are recorded.*

**下一輪 agent 工作時**：

| 攻擊角度 | 防線 |
|---|---|
| Agent 想偷工，跳過某個 edge case | **Tests** 跑就 fail，PR 不能 merge |
| Agent 不知道之前為何那樣設計，重蹈覆轍 | **Docs** 在 context 裡，agent 看得到「**weight rounding 為何要強制**」|
| Agent 寫的新版品質爛 | **Evaluation 分數**對照，新版 5.3/10 < 舊版 6.8/10，明顯退步 |

**這 3 樣同時在 context window，agent 沒辦法 regress**。

而且**每 turn 都增加**——下一版 agent 又寫了更多 tests / docs / evals，下一輪沒辦法 regress 的範圍更大。

**Quality floor 只能升不能降。Forward-only motion. That's the ratchet.**

---

## Garry 的具體案例：Holder Confusion（GBrain）

> 完整細節見 [Ch 00 原文](00-original-article.md#具體案例-1holder-confusiongbrain)

| 階段 | 動作 | 對應 ratchet 3 樣 |
|---|---|---|
| V1 跑 100,720 個 claims | 抽取「誰相信什麼」 | （初始 baseline）|
| Cross-model eval（GPT-5.5 + Claude） | 6.8/10 | **Evaluation** ✓ |
| 找到 holder confusion（35% 認錯人） | 6 種 failure mode 文件化 | **Documentation** ✓ |
| V2 prompt 改 + 17 個測試 | 鎖住合約 | **Tests** ✓ |
| Weight rounding 在 DB layer 強制 | 不准 0.74 假精度 | (架構性保護) |
| **結果** | 未來版本不能 regress | Quality floor 升 ✓ |

**這就是「**一個 turn 的 ratchet**」**。下一次任何 agent 改這部分，**17 個測試會抓**，**6 種 failure mode 在 context**，**6.8/10 是品質地板**。

無人需要記住「**為何 weight rounding 重要**」或「**holder confusion 是什麼**」。

> **The tests remember.**
> 「測試記得。」

---

## 為何這跟 Tech Debt 完全不同

| 維度 | Tech Debt | Complexity Ratchet |
|---|---|---|
| **方向** | 壞東西累積 | 好基線累積 |
| **目標** | 越少越好 | 越多越好 |
| **可逆性** | 可重構還清 | Forward-only（這是 feature 不是 bug）|
| **比喻** | 銀行利息 | 棘輪扳手 |
| **產生原因** | 趕進度妥協 | 正常 development 自動產生 |
| **管理方式** | Sprint 預算還債 | 把 3 樣納入 PR template |

Garry 在原文沒直接對比這兩個，但這個區分是理解他論點的關鍵。**他不是在講「**怎麼管理 tech debt**」，是在講「**怎麼建立 quality 永遠 only up 的機制**」**。

---

## 反例：「Vibecoded」專案的死亡模式

Garry 拿 Karpathy 提的 **vibecoding** 術語當反例：

> *"Vibecoding" is Andrej Karpathy's term for coding with AI by describing what you want in natural language and letting the model generate the code. It's powerful and it's how I build.*

但：

> *Most vibecoded projects that skip tests start falling apart once they reach moderate complexity — a few thousand lines, a handful of interacting features.*

**典型死法**：
1. Skip tests, skip docs, skip evals
2. Agent 加 complexity，沒東西防 regression
3. 每個新 feature 有機率破壞 old feature
4. 沒 test → user 報才知道
5. V0.5 之後：「**haunted house**」（鬧鬼的房子）——每改一處壞別處
6. 開發者寫部落格說「**AI coding 不行**」
7. **Garry 反駁**：「**AI coding 沒問題，是他們沒造棘輪**」

---

## 棘輪不只用在傳統 code

Garry 後段有個重要延伸：**Everything harnessable is testable**。

不只 unit test。**只要能 observe，就能 assert，就能 ratchet**：

| 層次 | 例子 |
|---|---|
| **OS** | migration 有建對表？cron 有 fire？ |
| **Terminal** | AI agent 在 review 時有 ask question？|
| **Browser** | 頁面 render？form 填對？|
| **API** | JSON schema 對？|
| **Agent behavioral** | agent 照 protocol？刪除前有確認？|

Garry 親身案例：**TTY test harness**（見 [Ch 00](00-original-article.md#具體案例-2tty-test-harnessgstack)）——用 Bun TTY 功能 spawn Claude Code 進 pseudo-terminal，**監看 terminal output 確認 agent 有沒有 fire interactive question**。

**「**這不是測 code，是測 AI agent 有沒有遵守行為合約。在 TTY 層級，真的看著它工作。**」**

---

## 為何 v1.0 寫錯了

v1.0 我寫：「**部署一個 feature 就鎖住技術債只增**」。

**錯在**：
1. 方向錯——是品質升不是 debt 升
2. 主體錯——主角是 **3 樣每 turn**，不是「部署 feature」
3. 機制錯——是「**context window 載入 3 樣**」造成 agent 無法 regress，不是某種「不可逆部署」

修正：用 [Ch 00 原文](00-original-article.md) 重新校準你的理解。**Garry 棘輪的精神是「optimistic」**（一切會越來越好），不是 v1.0 的「pessimistic」（debt 越來越多）。

---

## 本章小結

| 觀念 | v1.0 描述 | v2.0 修正 |
|---|---|---|
| 棘輪方向 | ❌ 技術債只增 | ✅ 品質地板只升 |
| 棘輪主體 | ❌ Feature deployment | ✅ 每次 agent session 加 3 樣 |
| 棘輪機制 | ❌ Production 鎖住 | ✅ Context window 載入 3 樣防 regression |
| 對 tech debt | ❌ 混為一談 | ✅ 完全不同概念 |

---

## 接下來

➡️ [Chapter 02: 驗證瓶頸](02-verification-bottleneck.md)

理解了「棘輪是 good thing」之後，下一章談「為何 AI 時代驗證能力跟不上產出能力」——這是 Garry 寫整篇文章的時代背景。
