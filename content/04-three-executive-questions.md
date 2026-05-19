[← 回 README](../README.md) · [← Ch 03](03-why-90-percent.md)

# 第 04 章：三大組織問題 — 三問的展開與診斷工具

> **核心句**：Garry Tan 的三大問題不是修辭，是診斷工具。任何組織只要 5 秒內回答不出來，就已經被棘輪追上。本章把這三問展開成可量化、可執行的組織體檢表。

---

## 三問原文

> 1. What is our **verification coverage** on AI outputs?
> 2. Where does the **ratchet bite hardest** — and is that where we deployed our first agent?
> 3. **Who owns** catching regressions before they reach customers/citizens/patients?

每一問都對應組織的一個 critical 維度：

| 問 | 維度 | 失敗後果 |
|---|---|---|
| Q1 | **能力**：你有沒有看 AI 產出的能力 | 不知道 AI 在做什麼 |
| Q2 | **戰略**：你把資源放在哪 | 棘輪在 A，agent 在 B，白做 |
| Q3 | **究責**：誰負責 | 沒人 own → 永遠救火 |

---

## Q1：What is our verification coverage on AI outputs?

### 字面拆解

**Verification coverage on AI outputs** ≠ **code coverage**。Garry 問的不只是「**多少行被測試到**」，是**完整的驗證鏈**：

| 層次 | 對 AI 產出的驗證 |
|---|---|
| L1：寫進 PR | 有沒有 type check、lint、static analysis pass？ |
| L2：自動測試 | 有沒有 unit / integration / E2E 跑過？ |
| L3：人類審 | 有沒有 senior 看過邏輯與設計？ |
| L4：staging | 有沒有在類似 prod 環境跑過？ |
| L5：canary | 有沒有用真實流量小規模驗證？ |
| L6：監控 | Production 有沒有監控覆蓋？ |

**Verification coverage = 6 個層次都做到的比例**。

### 診斷表：你的 verification coverage 真實分數

填這個表（誠實）：

| 維度 | 目標 | 你目前 | gap |
|---|---|---|---|
| L1: 100% PR 通過 lint + type check | 100% | __% | __ |
| L2: aggregate test coverage | 90% | __% | __ |
| L2: mutation score | 75% | __% | __ |
| L2: critical path test coverage | 100% | __% | __ |
| L3: 100% PR 有人類 reviewer | 100% | __% | __ |
| L3: 平均 review SLA | < 24h | __h | __ |
| L4: PR 必先過 staging | 100% | __% | __ |
| L5: prod 部署用 canary / feature flag | 100% | __% | __ |
| L6: critical service 有 alert | 100% | __% | __ |

**任何維度 < 80% 都是「**verification 缺口**」**。

### Q1 的隱含結論

如果你回答不出每個維度的數字，**你的組織連自己的 verification gap 都不知道**。第一步是**測量**：用 [Ch 08 路線圖](08-implementation-roadmap.md) 的 Week 1-2 行動。

---

## Q2：Where does the ratchet bite hardest — and is that where we deployed our first agent?

### 字面拆解

兩個子問題：
1. 棘輪在哪裡咬最深？（識別風險區）
2. 你的第一個 AI agent 在這裡嗎？（驗證部署優先級）

如果 1 跟 2 不匹配，**你的 AI 部署戰略錯了**。

### 找出 ratchet bite hardest 的方法

3 個資料源：

#### 資料源 1：Incident 歷史

過去 12 個月所有 P0 / P1 incident，按系統分類：

```
系統 A: 8 次 P1
系統 B: 3 次 P1
系統 C: 12 次 P1  ← bite hardest
系統 D: 1 次 P1
系統 E: 5 次 P1
```

C 是 ratchet 咬最深的地方——bug 一直冒。

#### 資料源 2：MTTR（Mean Time To Recovery）

```
系統 A: 平均 30 min
系統 C: 平均 3 hours  ← 不只 bug 多，而且難修
系統 D: 平均 15 min
```

C 同時兼具「**bug 多 + 難修**」——典型 ratchet 困境。

#### 資料源 3：「不敢動」清單

問 senior engineers：「哪些 module 你**不敢**改？」答案會集中在 1-3 個 module。**那就是 ratchet bite hardest 的地方**。

### Q2 的具體 action

**將 3 個資料源交集**，挑出 top 3 ratchet 區。**你的第一個 AI agent 應該部署在這裡——但是部署成「**測試生成 agent**」，不是「**feature 生成 agent**」**。

| 順序 | 你做的事 |
|---|---|
| 第 1 個 agent | 為 top 1 ratchet 區寫測試的 agent |
| 第 2 個 agent | 為 top 2 ratchet 區寫測試的 agent |
| 第 3 個 agent | 為 top 1 區寫 monitoring / alert 的 agent |
| 之後 | 才開始 feature-writing agent |

**反直覺**：別人都在用 AI 寫 feature，**你應該先用 AI 補防線**。

---

## Q3：Who owns catching regressions before they reach customers?

### 字面拆解

**Who owns** 是 organizational accountability。「**測試是大家的事**」=「**沒人負責**」。

### 三種 ownership 模式

| 模式 | 結構 | 問題 |
|---|---|---|
| **Distributed**（散落） | 每個工程師寫自己的測試 | 無人總攬，品質參差，棘輪悄悄轉 |
| **Centralized**（集中） | QA 團隊負責所有測試 | 開發跟測試脫節，QA 反應慢，瓶頸 |
| **★ Hybrid**（混合） | 每個 PR 作者寫測試 + 中央 platform team 設標準與工具 | 規模化、有 oversight |

**Garry 推 Hybrid**。具體形式：

#### 角色設計

```
[Engineering team]                  [Platform / Quality team]
   ↓                                    ↓
寫 feature code                     設計測試 framework / fixture
寫 first-pass tests                   設計 CI gate 標準
跑 local tests                        負責 mutation testing infra
PR submit                             跑 nightly extensive test suite
                                      設計 verification SOPs
                                      負責 80% → 90% migration
                                      負責 AI test generation 工具
                                      負責 metric 收集與報告
                                      ★ HAS POWER TO BLOCK ★
```

**關鍵點**：Platform/Quality team **必須有阻擋 PR 的權力**。沒這權力，他們建議會被無視。

### Q3 的具體 action

3 個 hire / appoint 任務：

| Hire | 職責 | Headcount（每 50 工程師） |
|---|---|---|
| Quality Engineer | 設計測試標準 / 工具 / fixture | 1-2 個 |
| Verification Architect | 設計 verification system（CI / mutation / canary）| 1 個 |
| Reliability lead | 連結 verification 跟 production reliability | 1 個 |

**這些不是新角色**，他們以前都叫 "QA"、"Build engineer"、"SRE"。**Garry 的洞察**：在 AI 時代，**這些角色從「**second-class**」變成「**first-class**」**——薪水、地位、影響力都要跟 senior dev 持平。

---

## 三問檢核表

填這個表，每題 0-10 分（10 = 完全做到）：

| 問題 | 子項 | 分數 |
|---|---|---|
| **Q1**：verification coverage | L1-L6 6 個層次平均 | __/10 |
| Q1 | 知道每個系統的 coverage 數字 | __/10 |
| Q1 | 有 mutation testing | __/10 |
| **Q2**：知道 ratchet 在哪 | 有 incident 歷史分析 | __/10 |
| Q2 | 有 MTTR 數據 | __/10 |
| Q2 | 第一個 AI agent 部署在 ratchet 區 | __/10 |
| **Q3**：有人 own | 有 quality team | __/10 |
| Q3 | Quality team 有阻擋 PR 權 | __/10 |
| Q3 | Senior 級別的 verification ownership | __/10 |

**總分判讀**：

| 區間 | 評估 |
|---|---|
| 80-90 | 你已進入 AI 時代 |
| 60-79 | 還在追趕，加速 |
| 40-59 | 棘輪在咬你，急救 |
| < 40 | 緊急狀態，立即啟動 [Ch 08 路線圖](08-implementation-roadmap.md) |

---

## 三問背後的世界觀

這三問的共同精神：

> **AI 不是省人力工具，是 leverage 放大鏡——你過去 strong 的地方會被放大，weak 的地方也會被放大**。

過去 verification 弱的組織，AI 時代會被棘輪咬死。
過去 verification 強的組織，AI 時代會把優勢放大成壟斷。

**這是「**未來 24 個月誰贏**」的本質**。

---

## 給 CEO / CTO 的 1 分鐘版本

如果你只有 1 分鐘跟 CEO 簡報這章，講這個：

> 「**老闆，我們有三個必須回答的問題：**
> 
> **(1) 我們對 AI 產出的驗證覆蓋率是多少？**——如果 5 秒內答不出來，我們已經輸了。
> 
> **(2) 我們最大的技術風險在哪？我們的第一個 AI 部署在那邊嗎？**——如果 agent 部署的位置 ≠ 風險位置，我們配置錯誤。
> 
> **(3) 誰負責在 production 出問題前抓到？**——如果沒人 own，我們的命運是運氣。
> 
> 這三個問題之外，所有 AI 策略討論都是次要的。」

---

## 本章小結

| 問 | 量化形式 | 立即 action |
|---|---|---|
| Q1：verification coverage | 6 個層次 ≥ 90% | 測量 + 設 CI gate |
| Q2：ratchet bite hardest | Incident / MTTR / 不敢動清單交集 | 部署測試 agent 到該區 |
| Q3：who owns | Hybrid model + Quality team with power | Hire / appoint Q-arch |

---

## 接下來

➡️ [Chapter 05: 測試類型矩陣](05-test-types-matrix.md)
