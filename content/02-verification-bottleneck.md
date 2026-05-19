[← 回 README](../README.md) · [← Ch 01](01-the-complexity-ratchet.md)

# 第 02 章：驗證瓶頸 — 從 Production 搬到 Verification

> **核心句**：軟體工程史上的瓶頸，從來都是「**做不夠快**」。AI agent 把這個瓶頸打破，新的瓶頸是「**驗證做的東西**」。看清這個轉移，你才能配置正確的資源。

---

## 60 年瓶頸演進史

| 時代 | 瓶頸 | 解法 |
|---|---|---|
| 1960s | 記憶體 / 計算力 | 摩爾定律 |
| 1980s | UI / 介面複雜 | 圖形介面、framework |
| 2000s | Web scale | 雲端、分散式系統 |
| 2010s | 開發速度 | DevOps、CI/CD、microservices |
| 2020-2024 | 上市時間 | 敏捷、API economy、SaaS |
| **2025+** | **驗證 / 確認** | **？** |

過去 5 代瓶頸都在「**做的快慢**」這條軸上。**2025 起，這條軸的瓶頸消失了**——AI agent 可以一天寫完過去一個團隊一週的東西。

**新瓶頸是另一條軸**：「**確認做的東西有沒有錯**」。這條軸 60 年來幾乎沒進步。

---

## 為何驗證能力沒同步進步

3 個歷史原因：

### 原因 1：驗證本來就比寫程式難

寫程式：「**讓電腦做某件事**」——目標明確，工具豐富
驗證：「**確認電腦做的事符合所有可能情境**」——目標模糊，工具有限

證明軟體正確性是電腦科學「**最難**」的問題之一（Halting problem 等）。

### 原因 2：人類驗證的瓶頸在「注意力」不在「技能」

寫 1000 行 code：訓練可以加速
讀 1000 行 code：注意力極限是物理限制，訓練幫助有限

> 一個 senior engineer 在 8 小時內，認真 review 程式碼的 cap 是 ~500 LOC（業界研究）。**這個數字 30 年沒變**。

### 原因 3：測試不是新東西，但測試思維沒升級

業界從 1980s 就有測試。但測試在絕大多數組織裡仍被當作「**寫完 code 之後的補課**」，而不是「**設計時的一階考量**」。

---

## 瓶頸轉移的具體表現

### 表現 1：Code review 從「審美」變「續命」

過去：reviewer 看 code 主要是「**這寫得好不好看**」
現在：reviewer 看 code 是「**這會不會壞 production**」

當你的同事一週塞 50 個 PR 給你（因為他有 AI 幫忙），你連看完都做不到，更別說「審美」。

### 表現 2：Bug 從「漏掉」變「審不過來」

過去：bug 之所以漏掉，是因為**沒寫到**測試
現在：bug 漏掉，是因為**寫了測試但沒 review、或 review 沒看出**測試本身有問題

### 表現 3：Senior engineer 變成「人類 oracle」

過去：senior 寫架構、寫核心邏輯
現在：senior 主要工作變成「**確認 AI 做的事對不對**」

如果你公司有 5 個 senior 工程師，AI 讓他們的「**生產時間**」減少 50%，但「**驗證 AI 產出**」吃掉省下的 50% 還不夠。

### 表現 4：On-call 變得不可預測

過去：on-call 大致知道哪些系統不穩定
現在：incident 來自「**昨天 AI 改的某行 code**」，找出來都要 1 小時

---

## Garry Tan 的論述：把瓶頸移轉視覺化

```
2010s 模型：

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ design   │ ──▶│   code   │ ──▶│  review  │ ──▶│  deploy  │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
       低          ★★★ 瓶頸       中             低
       速度        ━━━━━━━━━━━

  解法：請更多工程師、用更好的工具、improve 開發流程

──────────────────────────────────────────────────────────────

2025+ 模型：

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ design   │ ──▶│   code   │ ──▶│  review  │ ──▶│  deploy  │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
       低          AI 加速 → 快   ★★★ 瓶頸     低
                                  ━━━━━━━━━━━

  解法：升級驗證能力——測試自動化 + AI 寫測試 + 90% coverage
```

---

## 驗證的 3 個層次

「驗證」不是單一動作，是 3 個層次的組合：

### 層次 1：**Static verification（靜態驗證）**

- Type checking（TypeScript / Mypy）
- Linter（ESLint / Pylint / Ruff）
- Static analysis（CodeQL / Semgrep）
- AI code review（自動）

**特性**：cheap、fast、cover broad，但**抓不到行為錯誤**。

### 層次 2：**Test verification（測試驗證）**

- Unit tests
- Integration tests
- E2E tests
- Property-based tests
- Mutation tests

**特性**：抓行為錯誤，**但只能抓「你想到要測的」case**。

### 層次 3：**Production verification（生產驗證）**

- Feature flag 漸進放量
- Canary deployment
- Synthetic monitoring
- Real user monitoring
- Anomaly detection

**特性**：抓「實際使用中的問題」，**但代價是讓 user 當白老鼠**。

**Garry 的論點**：3 個層次都要做，但**90% 測試覆蓋率（層次 2）是新基線**——因為它最能匹配 AI 的產出速度。

---

## 不是不要 review，而是 review 要被 amplify

「**90% 覆蓋率**」不是叫你不要 code review。**而是把 reviewer 注意力解放出來做更高槓桿的事**：

| 沒高覆蓋率時，reviewer 要做 | 有高覆蓋率時，reviewer 可以做 |
|---|---|
| 看每行 code 有沒有 bug | 看 architecture 對不對 |
| 跑各種 case 在腦中模擬 | 看 design 假設合不合理 |
| 擔心 edge case 沒處理 | 擔心 product 邏輯對不對 |
| 「這會不會壞 X 功能」 | 「這個 feature 該不該做」|

**測試是 reviewer 的 force multiplier**。Reviewer 不需要再扮演「**人類 oracle**」，他可以扮演「**戰略 partner**」。

---

## 反論：「100% coverage 也救不了」

有人會反駁：「就算 100% line coverage，也不保證對。」

**對**。但這不否定「**90% 比 60% 好**」這件事。

關鍵區別：
- **line coverage**：每行至少被執行一次（可能根本沒 assert）
- **branch coverage**：每個 if-else 兩支都跑過
- **mutation score**：故意改 code，看測試會不會抓到（**這才是真品質指標**）

[第 05 章](05-test-types-matrix.md) 會展開測試類型矩陣。**Garry 講的 90% 是「**有意義**」的 90%，不是「**為達標而達標**」的 90%**。

---

## 對組織的意涵

當你看清「瓶頸已轉移」這件事，這些 decisions 變得清楚：

| Decision | 過去做法 | 新做法 |
|---|---|---|
| Hire 順位 | 多 hire developer | 多 hire 測試 / SRE / 平台工程師 |
| 預算分配 | 70% feature dev / 30% infra | 50% / 50% |
| Sprint 規劃 | feature 為主 | feature + verification capacity 雙軌 |
| Code review SLA | 24 小時內 | 「**測試 + AI 預審**先過、人類 review 48 小時內** |
| Senior 角色 | 寫 hard problem 的 code | 設計 testability、設計 verification 系統 |

---

## 本章小結

| 觀念 | 為何重要 |
|---|---|
| 60 年瓶頸演進，從速度到驗證 | 看清歷史脈絡 |
| 驗證能力沒同步進步的 3 個原因 | 知道對手在哪 |
| 瓶頸轉移的 4 個具體表現 | 自我診斷組織狀態 |
| 3 個驗證層次（static / test / prod） | 配置完整防線 |
| 測試是 reviewer 的 force multiplier | 改變 reviewer 角色 |
| 90% 覆蓋率 ≠ 100% 對 | 但 90% 仍然遠優於 60% |

---

## 接下來

➡️ [Chapter 03: 為何 90% 是新基線](03-why-90-percent.md)
