[← 回 README](../README.md) · [← Ch 03](03-why-90-percent.md)

# 第 04 章：三個實戰案例 — Ratchet 怎麼運作（v2.0 重寫版）

> ⚠️ **v1.0 訂正**：本章 v1.0 是「**執行長必問三大問題**」——**那 3 個問題是我虛構的，原文無此結構**。v2.0 完全重寫為 Garry 在文章裡親自舉的 3 個實戰案例，這才是文章的精華。

> **核心句**：Ratchet 不是抽象概念，是具體機制。Garry 在原文示範了 3 個真實案例：每個都展示「**問題出現 → 加 (test + doc + eval) → quality floor 永久升一格**」的完整 turn。

---

## 為何案例比理論重要

Garry 整篇 article 的精華是這 3 個 case：

| 案例 | 系統 | 問題 | 解法 |
|---|---|---|---|
| 1. **Holder Confusion** | GBrain | LLM 抽取信念時 35% 認錯人 | 6 種 failure mode 文件化 + 17 測試 + DB 層強制 weight rounding |
| 2. **TTY Test Harness** | GStack | Claude Code 跳過互動 review 環節 | 3 層 ratchet（STOP gates + anti-shortcut + TTY 測試）|
| 3. **OpenClaw Plugin Test** | GStack | 怎麼測 plugin 真的能載入 | 359 行 end-to-end 跨兩個程式測試 |

讀完這 3 個你會理解：**ratchet 不是「**加更多 unit test**」，是「**為每個 lesson learned 鎖住一個可執行檢核**」**。

---

## 案例 1：Holder Confusion（GBrain epistemological extraction）

### 系統背景

**GBrain** 是 Garry 在做的 second brain for AI agents——讓 AI agent 有長期記憶，儲存 / index / search 一個人的 note / meeting / 對話 / research。「**你 AI 助手可以真的讀的第二大腦**」。

其中一個 feature 叫 **epistemological extraction**（認識論抽取）：

> 它讀過幾千頁，抽取「**誰相信什麼**，**信心多高**，**隨時間怎麼變**」。
>
> 例子：
> - "Garry thinks Bitcoin will hit $300K (confidence: 0.45)"
> - "Jared thinks this startup has strong retention (confidence: 0.80)"
>
> 規模：**28,000 頁** 跨多人。

### 問題

第一次 run 抽取出 **100,720 個 claims**。

Garry 用 **cross-model evaluation** 評估品質（GPT-5.5 + Claude **獨立**評分）：

> **總分：6.8 out of 10**

最大問題是 **holder confusion**（持有者混淆）。

具體例子：claim「**AI will replace 80% of software engineers by 2027**」。

**問題**：**誰**相信這件事？
- 是寫這句話的人？
- 是他在引用的某個其他人？
- 是系統的 analysis engine 從 podcast 推論的？

**V1 對這個區分 35% 答錯**。

對「**追蹤誰相信什麼**」的系統來說，搞錯 holder 等於整個系統失去意義。

### Ratchet 三段式解法

```
Step 1: Documentation
   6 種 failure mode 識別出來 + 寫成文件
   
Step 2: Tests
   17 個測試鎖住合約
   
Step 3: Architecture
   Weight rounding 在 DB layer 強制
   (不允許 0.74 這種假精度，必須 round 到 0.05 增量)
```

### Ratchet 效果

> *Now no future version of the extraction can ship without those 17 tests passing.*
>
> 「未來任何版本的 extraction 不通過那 17 個測試就不能 ship。」

> *Nobody has to remember why weight rounding matters or what holder confusion is.* 
> 
> 「沒人需要記住為何 weight rounding 重要、什麼是 holder confusion。」

> **The tests remember.**
> **「測試記得。」**

**品質地板從此永遠 ≥ 6.8/10**。一個 turn 的 ratchet 完成。

### 拆解：ratchet 的 3 樣對應

| Ratchet 3 樣 | 在這案例的具體形式 |
|---|---|
| **Tests** | 17 個測試 |
| **Documentation** | 6 種 failure mode 文件化 |
| **Evaluation** | Cross-model eval 6.8/10 baseline，下一版要 ≥ 這個 |

完整對齊 [Ch 01](01-the-complexity-ratchet.md) 的精確定義。

---

## 案例 2：TTY Test Harness（GStack interactive review）

### 系統背景

**GStack** 是 Garry 的開源 AI coding agent framework——93K stars / 701K LoC / 46 skills。

核心 feature 之一：**interactive plan review**。

> 你叫它 review 你的架構，它**一段一段走 plan**，**問問題**、**戳 edge case**、**挑戰你的假設**。像有個真的會讀 code 的 engineering manager。

### 問題

Claude Code 有時候會**跳過整個互動環節**：

- 讀完 plan 檔
- 把所有 findings 一次性 dump 出來
- 直接 exit
- **沒問用戶任何問題**

**Interactive review 的整個 point 就是來回對話。跳過 = 失去意義。**

### 反問：怎麼測這個？

> *How do you even test that? You can't unit test "did the AI have a conversation."*
>
> 「**這要怎麼測**？你沒辦法 unit test『**AI 有沒有跟你對話**』。」

**沒有任何 traditional testing framework 涵蓋這個**。

### Garry 的解法：TTY Test Harness（PR #1354）

用 **Bun 的 TTY 功能**建測試 harness：

1. **Spawn Claude Code 進 pseudo-terminal**（pty）
2. **餵特定 repo 場景**
3. **觸發 review skill**
4. **即時 watch terminal output**
5. **觀察 agent 有沒有 fire interactive question 在 finish 之前**
6. 如果 dump findings 後直接 exit，**測試 fail**

> *That's not testing code. That's testing whether an AI agent follows a behavioral contract. At the TTY level. By literally watching it work.*
>
> 「這不是測 code。是測 **AI agent 有沒有遵守行為合約**。**在 TTY 層級**。**真的看著它工作**。」

### Ratchet 3 層回應

```
Layer 1: STOP gates 在 skill instructions
   - 明文規則：「你 MUST 在進下一段之前 ask user」
   - Anti-rationalization 條款：明確命名 failure mode
   - 讓 model 沒辦法說服自己「**這次跳過 OK**」

Layer 2: Anti-shortcut clause
   - 一句話：「**The plan file is the OUTPUT of the interactive review, 
            not a substitute for it**」
   - 「plan 檔是 interactive review 的『**輸出**』，不是『**替代**』」
   - 封死 model 一直在 exploit 的特定 loophole

Layer 3: Gate-tier floor tests（核心）
   - TTY harness 測試
   - 在 controlled scenario spawn Claude Code
   - 如果 agent 沒問至少一個 interactive question 就 fail
```

### Ratchet 效果

> Anthropic ship 新 model 版本，或我改 skill prompt，**測試會抓**「interactive contract」的任何 regression。
>
> **Agent 不能靜悄悄停止問問題。測試在看 terminal**。

**Quality floor 永遠 ≥「**至少問一個 interactive question**」這個合約**。

### 為何這案例革命性？

過去測試思維：「**code 在做我想要的事嗎？**」
TTY harness 思維：「**AI agent 在遵守我定義的協議嗎？**」

這是「**測試的擴展定義**」——從「**code 行為**」延伸到「**agent 行為**」。

---

## 案例 3：OpenClaw Plugin Test（PR #880）

### 背景

GStack 生態加了個新的 **OpenClaw plugin**。

問題：「**plugin 真的能載入跑嗎**」**怎麼測**？

### 傳統做法（不夠）

- Unit test：plugin code 編得過
- 但**這只證明 compile 過**，**沒證明 runtime 跑得起來**

### Garry 的解法：359 行 end-to-end test

測試流程（在 PR #880 裡）：

```
1. Build plugin from source
2. Spawn 真的 OpenClaw instance in isolated profile
3. Install plugin via CLI
4. Run `plugins inspect` → verify runtime 載入了
5. Set config slot
6. Validate config
7. Run `plugins doctor` → confirm zero diagnostics
```

**完整 end-to-end，跨兩個獨立程式**。

> *359 lines of test code. The kind of test a human would almost never write by hand because the setup is too tedious. Claude wrote it in about five minutes. That's the effort wall disappearing in real time.*

「359 行測試 code。**人類幾乎不可能手寫這種測試**——setup 太繁瑣。Claude 5 分鐘寫完。**這就是 effort wall 即時消失**。」

### 為何這個案例最具標誌性

它示範了 Garry 整篇 article 的 **terminal insight**：

> **The brutal last 20% that made 90% coverage impractical for human teams is exactly the kind of work AI agents are best at.**
>
> 「過去讓 90% 覆蓋率不切實際的『**最後 20% 殘酷工作**』，**正是 AI agent 最擅長的**。」

**人類 hate 寫的 boilerplate-heavy / setup-heavy / end-to-end test**——agent **完全不介意**。

這把過去「**90% 是航太 / 醫材獨享奢侈**」的格局打破。**現在每個 startup 都應該 90%**。

---

## 三個案例的共同 pattern

| Step | Case 1 (Holder Confusion) | Case 2 (TTY Harness) | Case 3 (OpenClaw Plugin) |
|---|---|---|---|
| **發現問題** | V1 35% 認錯 holder | Claude Code 跳過 interactive | 不確定 plugin 載入 |
| **加 Doc** | 6 種 failure mode 文件化 | Anti-shortcut clause | (PR 描述 + skill doc) |
| **加 Test** | 17 個合約測試 | TTY harness + STOP gate | 359 行 end-to-end |
| **加 Eval** | Cross-model 6.8/10 baseline | 「至少問一個 question」可驗證 | `plugins doctor` 0 diagnostics |
| **Ratchet 效果** | 未來 extraction ≥ 6.8/10 | 未來 review 必問 question | 未來 plugin 必通過 e2e |
| **Quality floor 升** | ✓ | ✓ | ✓ |

每個案例都符合「**3 樣加進 codebase，下一輪 agent 不能 regress**」的精確機制（[Ch 01](01-the-complexity-ratchet.md)）。

---

## 對你 codebase 的啟發

讀完這 3 個案例，問自己：

1. **過去 3 個月你 codebase 出過什麼 incident / bug / feedback？**
2. **每個都有對應的 (test + doc + eval) 鎖住嗎？**

如果沒有，那 incident 會再發生——因為 ratchet 沒造好。

具體 action：
- 拉 incident log
- 每個 incident 寫一段 ratchet 三件套（test / doc / eval）
- 加進 codebase + CI gate
- **下一個 incident 不會是同一個**

這就是「**把 ratchet 從理論變成你 codebase 的實踐**」。

---

## 為何 v1.0「執行長三大問題」是錯的

v1.0 Ch 04 結構（**虛構**）：

```
❌ Q1: What is our verification coverage on AI outputs?
❌ Q2: Where does the ratchet bite hardest?
❌ Q3: Who owns catching regressions?
```

**問題**：
1. 這 3 個問題**原文不存在**——我憑空編的「顧問式 framing」
2. 問題本身**朝錯方向**——把 ratchet 當「**防爆機制**」（咬最深的地方需要 own），原意是「**升品質機制**」
3. 引導讀者**找 risk 區域去防**，而原意是「**讓每個 turn 自動加 3 樣**」——是常態，不是 risk 響應

**v2.0 改成「**3 個實戰案例**」**才忠於原文：你不是在管理 risk，你是在**每次 agent session 自動讓品質升一格**。

---

## 接下來

➡️ [Chapter 05: 測試類型矩陣](05-test-types-matrix.md) — 6 種測試類型怎麼搭配

（v1.0 Ch 05-09 內容大致正確，是延伸應用，不需重寫）
