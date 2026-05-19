[← 回 README](../README.md) · [← Ch 02](02-verification-bottleneck.md)

# 第 03 章：為何 90% 是新基線

> **核心句**：80% 是「**人類意志力天花板**」造成的歷史均衡點。AI 把寫測試的成本降到接近零後，這個天花板消失了。新的均衡點是 90%——不是因為 90% 完美，而是因為 80% 已經不夠應付 AI 加速的棘輪。

---

## 80% 從哪裡來

80% 不是某個科學論證得出的數字，而是 30 年來經驗加總的「**比較划算的點**」。

### 80% 的歷史定位

| 覆蓋率區間 | 經驗判斷 |
|---|---|
| < 50% | 太少，任何改動風險都很高 |
| 50-70% | 基本面，但 critical path 還會漏 |
| **70-85%** | **甜蜜點：抓到大部分 bug，維護 cost 可承受** |
| 85-95% | Diminishing returns 開始 |
| > 95% | 維護 cost 超過 marginal value，除了高風險系統 |

業界中位數一直在 60-70%，少數高品質組織能到 80-85%。**90% 是「**奢侈品**」**——因為人類寫測試實在太貴了。

### 80% 的隱形假設

80% 之所以是甜蜜點，**前提是「**寫測試的人類意志力**」是稀缺資源**。具體：

- Senior engineer 一天能寫 5-10 個高品質測試
- 寫測試比寫 feature 無聊，工程師不喜歡寫
- TDD 雖然有效但對新人有上手成本
- 寫測試 ROI 不明顯（看不到「**這個測試救了我多少次**」）

**這些前提**讓「**多寫 10% 測試**」要付出「**Sr engineer 一個月**」的代價。對中型公司來說，「**從 80% 拉到 90%**」需要兩個 quarter，這個 budget 通常被 feature work 排擠。

---

## AI 怎麼改變這個方程式

### AI 寫測試 ≈ 免費

當 Claude / GPT-4 / Cursor 等工具進入工程流後：

| 動作 | 人類時間 | AI 時間 |
|---|---|---|
| 為一個 function 寫 unit test | 15-30 分鐘 | 30 秒 |
| 為一個 class 寫完整測試套 | 2-4 小時 | 5 分鐘 |
| 為一個 module 寫 integration test | 4-8 小時 | 15 分鐘 |
| 補一個邊角 case 測試 | 10 分鐘（注意力切換） | 30 秒 |

**AI 把「寫測試的成本」壓到接近零**。

### 那「**80% 是甜蜜點**」的論證還成立嗎？

**不**。原本 80% 是甜蜜點是因為：

```
marginal cost of writing test ≈ marginal value at 80%
```

當 cost 接近 0：

```
marginal cost ≈ 0
marginal value at 90% > 0
∴ 90% 變新均衡
```

---

## AI 寫測試的具體經濟學

假設你有一個 100K 行的 codebase，目前 75% coverage：

| 情境 | 人類拉到 90% | AI 輔助拉到 90% |
|---|---|---|
| 需要新增測試 | ~5000 個測試 | 同 |
| Senior engineer 時間 | 1250 小時（~7 個月 FTE） | 100 小時 review + AI 寫 |
| 直接成本 | $200K（按 $160/hr） | $20K（含 LLM API cost） |
| Calendar 時間 | 6 個月 | 3 週 |
| 機會成本（沒做 feature） | 6 個月 product roadmap | 3 週 product roadmap |

**結論**：用 AI 寫測試，**從 75% 拉到 90%** 在 3 週、$20K 內可完成。對 Y Combinator 規模的新創，這完全在預算內。

---

## 90% 的具體 benchmark

「**90% coverage**」這句話太抽象，必須拆解：

### 分項目標

| 維度 | 90% 基線含意 |
|---|---|
| **Line coverage** | 90% 的程式行被測試執行過 |
| **Branch coverage** | 90% 的 if-else 兩支都被測過 |
| **Function coverage** | 90% 的函式有 ≥ 1 個測試 |
| **Critical path coverage** | 核心業務流程 100% 覆蓋 |
| **Mutation score** | ≥ 75%（mutation testing 抓住假測試）|

**單看 line coverage 達到 90% 不夠**。**Garry 的 90% 是綜合性的**：core path 100% + branch ≥ 90% + mutation ≥ 75%。

### 分系統優先級

不是所有 module 都需要 90%，但 critical 系統必須：

| 系統類型 | 覆蓋率基線 |
|---|---|
| **Payment / billing / auth** | 95-100% |
| **核心業務邏輯** | 90-95% |
| **資料 schema / migration** | 90-95% |
| **API contract** | 90% |
| **UI / 前端** | 70-85%（用 E2E 補）|
| **Build script / 內部工具** | 50-70% |
| **Glue code / config** | 不強求 |

**90% 是「**aggregate weighted**」**，不是每個檔案都 90%。

---

## 90% 不是終點：90% + 還要什麼？

達成 90% 覆蓋率**只是**充分條件之一。**完整防線**還需要：

### 1. 測試品質保證

防止「**假測試**」：見 [Ch 07 反模式](07-anti-patterns.md)。具體：
- Mutation testing（Stryker / mutmut）
- AI 評審測試品質（[Ch 06](06-ai-as-test-writer.md)）
- 拒絕 `assert True` 類的 placeholder

### 2. CI 防線

- `--fail-under=90` 在 CI 強制
- PR 不能降低覆蓋率（diff coverage 必須 ≥ 90%）
- 不能 disable 測試（除非 marked 並有清楚 expiry）

### 3. Observability 補測試漏網

- Error tracking（Sentry）
- Performance monitoring（Datadog）
- Anomaly detection
- User behavior analytics

### 4. Postmortem 文化

每個 production incident → 「**為何測試沒抓到**？」→ 補測試 → 防 regression

---

## 反論與回應

### 反論 1：「90% coverage 還是會有 bug」

**回應**：對，但 90% bug 比 60% bug 少 4 倍以上（業界經驗）。**沒有完美防線，只有性價比更高的防線**。

### 反論 2：「coverage 不等於品質」

**回應**：完全正確。所以才有 [Ch 07 反模式](07-anti-patterns.md) 跟 mutation testing。**Garry 的 90% 是有意義的 90%**，不是 gaming。

### 反論 3：「我們是 startup，沒時間寫測試」

**回應**：這是過時思維。**現在**寫測試的時間，是過去的 1/10。「**沒時間寫測試**」基本上等於「**不會用 AI**」。

### 反論 4：「AI 寫的測試不可靠」

**回應**：AI 寫的測試需要 human review，但人類 review 一個測試 < 寫一個測試的 1/10 時間。**槓桿仍然成立**。

### 反論 5：「我們有 manual QA」

**回應**：Manual QA 規模化不了。Manual QA 抓的 bug 數量是線性，AI 產出 bug 數量是指數。**數學上贏不了**。

---

## 為什麼是 90%，不是 92% 或 88%？

90 是個 round number，溝通方便。但實際**真正的閾值**：

- **Critical path**：100%（不容妥協）
- **Business logic**：95%（極少例外）
- **Aggregate**：90%（CI gate 的數字）
- **UI / 邊緣**：85%（E2E 補）

90% **作為組織級宣示**，是個容易理解、易執行、易檢核的數字。「**今年要 90%**」比「**今年要綜合品質指標達到 P95**」好溝通 10 倍。

---

## 跟業界其他 benchmark 對照

| 標準 | 數字 | 對應 |
|---|---|---|
| Google internal | 60-70% line | 過時了 |
| Meta internal | 70-80% | 接近 |
| FAANG critical service | 85-90% | 對 |
| 醫療軟體（FDA）| 95-100% | 高於 90% |
| 航太軟體（NASA）| 100% MC/DC | 最高標準 |
| **Garry Tan AI 時代提議** | **90% (with quality)** | 新基線 |

**90% 已經不算激進**，只是把過去 FAANG critical service 的標準普及到所有 AI-augmented 組織。

---

## 本章小結

| 觀念 | 為何重要 |
|---|---|
| 80% 是人類意志力造成的天花板 | 解釋為何過去不該 push 更高 |
| AI 把寫測試成本壓到接近零 | 解釋為何現在該 push 更高 |
| 90% 是「**aggregate weighted**」 | 不是每個檔案都 90% |
| 90% ≠ 完美 | 還需要品質檢核、CI 強制、observability |
| 5 個反論的回應 | 對抗組織內阻力 |
| 90% 是 critical service 早就在做的 | 不激進，只是普及 |

---

## 接下來

➡️ [Chapter 04: 三大組織問題 — 三問的展開與診斷工具](04-three-executive-questions.md)
