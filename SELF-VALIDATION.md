# 自我驗證報告：Garry Tan 原文比對

> 日期：2026-05-19
> 動作：用 gstack/browse 實際讀取 [@garrytan post 2054064931515855118](https://x.com/garrytan/status/2054064931515855118) 原文，比對本 repo 內容

---

## 重大發現：這是 Article 不是 Tweet

原文不是一條短推文，是 **Garry Tan 在 X 發表的長篇 Article**：

- **標題**：「**The AI Agent Complexity Ratchet: Why 90% Test Coverage Is Required**」
- **時間**：2026-05-12 1:03 PM · 233.8K views
- **位置**：他 AI Explainer 系列的第 7 篇
- **同系列**：(1) Fat Skills, Fat Code, Thin Harness — (2) Resolvers — (3) The LOC Controversy — (4) Naked Models Are Stupider — (5) The Skillify Manifesto — (6) Meta-Meta-Prompting — **(7) The Agent Complexity Ratchet ← 這篇**

---

## 原文核心論點（精準摘要）

### 1. 個人 credibility 數據

- 過去一年用 AI coding
- 兩個開源專案：**GStack**（讓 AI coding agent 變強，93K stars）+ **GBrain**（second brain for AI，14K stars）
- 兩專案合計 **970,000 行 code, 665 個測試檔案**
- 全部由 **Claude Code + Codex** 在他指導下寫的
- 多數時間 **15 個同時 Conductor session**
- 上週：72 小時內 merge 14 個 PR，29,000 行新 code，**每次 release 比上次測試更好**

### 2. 「複雜度棘輪」精確定義

> A ratchet is a mechanism that allows motion in **one direction only**. A socket wrench turns a bolt forward and prevents it from turning back.

每次 agent coding session 加 **3 樣東西**：
1. **Tests** that encode what "correct" means — fail loudly if change breaks
2. **Documentation** that records why decisions were made — reasoning, tradeoffs
3. **Evaluation results** that establish quality thresholds — scores

下一次 agent 工作時，這 3 樣都進 context window，所以：
- 不能 regress below test suite — tests would fail
- 不能 ignore docs — 就在 context
- 不能 ship 低於 evaluation baseline — scores 都記錄

**「Quality floor goes up with every turn. Forward-only motion. That's the ratchet.」**

### 3. 軟體史的相位變化

過去 50 年軟體工程的組織原則：**prevent errors, because errors are catastrophic**
- 第一次就要寫對
- Miss edge case = production crash
- 不好的 migration = 客戶資料丟失
- 整套流程（code review / staging / QA / release train）= 防錯
- 結果：慢 + 複雜度天花板 = 一個團隊腦袋能裝多少

**現在**：AI agent 能讀 code / 理解 context / 診斷錯誤 / 寫 fix。**「Software is squishy now」**——大多數 code-level error agent 下一個 turn 就能修。

### 4. 「Everything harnessable is testable」

不只是 unit test。測試擴展到整個 stack：
- **OS level**：process tree / file system / cron
- **Terminal level**：每個 keystroke / 互動 prompt
- **Browser level**：rendered page / button states / navigation
- **API level**：JSON schema 驗證
- **Agent behavioral level**：agent 說了什麼 / call 了什麼 tool / 順序 / 是否在 action 前 ask

**只要能 observe，就能 assert，就能 ratchet**。

### 5. 具體案例（精彩部分我原版全缺）

#### 案例 1：Holder Confusion（GBrain epistemological extraction）

- 第一次 run 拉出 100,720 個 claims
- Cross-model eval（GPT-5.5 + Claude 雙評）：6.8/10
- 最大問題：**holder confusion** — 「AI 會取代 80% 工程師」是**誰**的觀點？作者？引用對象？分析引擎推論？
- Version 1：35% 認錯人
- Solution：6 個 failure mode 文件化 → V2 prompt 改 → 17 個測試鎖住合約 → weight rounding 在 DB layer 強制（不准 0.74 這種假精度）

#### 案例 2：TTY Test Harness（GStack interactive review）

- 問題：Claude Code 有時把整個 plan 一次倒出，**跳過互動環節**
- 反問：「**怎麼測「AI 有沒有跟你對話」**？」
- Solution：用 Bun TTY 功能（PR #1354），spawn Claude Code 在 pseudo-terminal，餵特定 repo 場景，觸發 review skill，**watch terminal 即時 output**，看 agent 有沒有 fire interactive question
- Ratchet 三層回應：
  1. **STOP gates**：「YOU MUST ask user before proceeding」+ anti-rationalization 條款（命名失敗模式讓 model 不能說服自己跳過）
  2. **Anti-shortcut clause**：「plan 檔案是 interactive review 的 OUTPUT，不是替代」
  3. **Gate-tier floor tests**：TTY harness 在 controlled scenario spawn Claude Code，沒問問題就 fail

#### 案例 3：OpenClaw Plugin Test（PR #880）

- 不只測 compile
- Build plugin from source → spawn 真 OpenClaw instance 在 isolated profile → 安裝 plugin via CLI → 跑 `plugins inspect` 確認 runtime 載入 → 設 config slot → 驗證 config → 跑 `plugins doctor` 確認 zero diagnostics
- **完整 end-to-end，跨兩個獨立程式，359 行測試 code**
- **「人類幾乎不可能手寫的測試，因為 setup 太繁瑣。Claude 5 分鐘寫完。」**——effort wall 消失的具體展示

### 6. 「90%」的數據基礎

**Capers Jones**（10,000+ projects）研究：
- < 70% coverage → DRE 65-75%
- 85-95% coverage → DRE 92-97%
- **knee 在 85%**——非線性，缺陷escape 量drop 急遽

**DO-178C / FAA**（飛行關鍵軟體）：
- 要求 **MC/DC**（modified condition/decision coverage）for Level A
- Branch coverage alone 漏 10-20% faults
- MC/DC 達 **>99% DRE**

**Six Sigma 類比**：
- 3σ = 67,000 defects/M
- 4σ = 6,200 defects/M（×10 better）
- 5σ = 233 defects/M（再 ×27 better）
- **不是線性改善，是相位變化**

**反論的誠實面對**：
- Mockus, Nagappan, Dinh-Trong 研究 Windows Vista：90%+ coverage 努力非線性陡升
- 最後 20% 比前 70% 更費力
- 這就是為何過去多數團隊停在 70-80%

### 7. 真正的 unlock：「AI doesn't experience effort」

> They don't get bored writing the fourteenth edge-case test. They don't cut corners at 5pm on a Friday. They don't look at a gnarly integration test and think "I'll come back to this later."

**過去 90% 不普及的根本原因不是技術，是「人類意志力 cost too much」**。
AI 沒有這個障礙：
- 不會無聊
- 不會偷懶
- 不會 2am 馬虎
- **「The effort curve that stopped human teams at 70% doesn't apply to agents」**

### 8. 反例：「Vibecoding」沒做 ratchet → 死

- **Vibecoding** = Karpathy 提的用 AI 寫 code 的方式
- 多數 vibecoded 專案 skip tests，到中度複雜度（幾千行）就解體
- Skip ratchet = 每個新 feature 都可能壞 old feature，沒測試到 user 報才知道
- V0.5 後 codebase 是 haunted house
- **「AI coding works fine. They just didn't build the ratchet.」**

### 9. 金句

> **「Tests are institutional memory that survives employee turnover.」**
> 測試是「在員工離職後仍存活的組織記憶」。
> 
> 註：// DO NOT CHANGE THIS — ask Dave，Dave 三年前走了。
> 
> Agent 的 context window 不會辭職、不會被挖角、不會忘。

> **「Getting to 90% used to be a heroic effort. Now it's a Tuesday.」**
> 過去達到 90% 是英雄壯舉，現在是「**星期二**」。

> **「For fifty years, 90% coverage was a luxury reserved for avionics and medical devices. AI agents demolished that wall.」**

> **「Every software company that doesn't adopt this model — agents plus taste plus a test suite that only goes up — is already shipping slower and with less quality than one person who has.」**

---

## 我的版本錯在哪

### ❌ 重大錯誤 #1：「**三大組織問題**」是我虛構的

**原文**：**完全沒有**「三大組織問題」這個結構。我虛構了 Q1/Q2/Q3 整章。

**事實**：原文裡有「I should be honest about what the research also shows」的學術誠實，有「The principle generalizes」的擴展論述，**但沒有「執行長必問三題」這種顧問式 framing**。

**影響**：Ch 04 整章是**虛構**的，需要重寫或刪除。

### ❌ 重大錯誤 #2：「**24 個月誰贏**」名言出處錯

**我引用**：*"The companies that win the next 24 months won't be the ones with the fastest agents...They'll be the ones with the strongest guardrails."*

**事實**：原推文**沒有這句話**。這是 Ali Ustun 在 LinkedIn 寫的 paraphrase。**Garry Tan 從沒這樣說過**。

**正確金句**應該是：
- 「Every software company that doesn't adopt this model... is already shipping slower and with less quality than one person who has.」
- 「Tests are institutional memory that survives employee turnover.」
- 「Getting to 90% used to be a heroic effort. Now it's a Tuesday.」

### ❌ 重大缺漏 #3：複雜度棘輪的**精確定義**

**我的定義**：「**每個 feature 部署不能反向，複雜度只增不減**」——錯，這是 tech debt 的定義

**正確定義**：「**每次 agent coding session 加 3 樣（tests + docs + evals），quality floor 只升不降**」

**差別**：我的版本講「**壞東西累積**」，原意是「**好基線只升不降**」。**方向完全相反**。

### ❌ 缺漏 #4：所有實戰案例

我整個 repo **沒提到一個 Garry 的實戰案例**：
- ✗ Holder confusion（GBrain extraction）
- ✗ TTY test harness（GStack interactive review）
- ✗ OpenClaw plugin test（PR #880）
- ✗ 970K LoC / 665 test files 規模
- ✗ 14 PR in 72h / 29K LoC

這些**才是文章的精華**——具體展示「AI 加速驗證」實際長什麼樣。

### ❌ 缺漏 #5：90% 的數據基礎

我用 Garry 沒提的「**業界中位數 60-70%**」當依據。**真正應該引用**：
- Capers Jones DRE 曲線（85% 是 knee）
- DO-178C / MC/DC
- Six Sigma 3σ→5σ phase change
- Mockus Vista 研究

### ❌ 缺漏 #6：核心洞察「**AI doesn't experience effort**」

這是整篇文章的**核心 insight**。我的版本只提「AI 成本降到接近零」，沒抓到 Garry 真正想說的：

> 過去 90% 達不到，不是技術問題，是**人類意志力問題**。
> 工程師 2am 寫第 14 個 edge case 會無聊；AI 不會。
> 「**The effort wall that stopped human teams at 70% doesn't apply to agents.**」

### ❌ 缺漏 #7：「Everything harnessable is testable」

原文獨立一節，論述測試擴展到 OS / Terminal / Browser / API / Agent behavioral 5 個層次。我的版本完全沒提，限縮在傳統 unit/integration/E2E 思維。

### ❌ 缺漏 #8：Karpathy vibecoding 致敬與反例

原文明確提到 Karpathy 的 **vibecoding** 術語，並批評「skip ratchet 的 vibecoded project 都死了」。我的版本完全沒這層脈絡。

### ❌ 缺漏 #9：GStack / GBrain 是 Garry 本人的開源專案

整篇文章用他兩個開源專案當 case study：
- **GStack** — 93K stars / 701K LoC / 46 skills / 37 contributors
- **GBrain** — 14K stars / 25 contributors

這些**就是 ratchet 真實運作的證明**。我完全沒提。

---

## 對讀者的更正建議

如果你已經讀過本 repo 內容，請在腦中**重新校準**這幾點：

1. **「複雜度棘輪」是好事，不是壞事**——它讓 codebase 只能變好不能變差
2. **每次 agent session 加 3 樣**（tests + docs + evals）→ 這就是 ratchet 機制
3. **AI 不感到 effort** 是真正的 unlock，不是 cost 降低
4. **90% 的數據依據**：Capers Jones / DO-178C / Six Sigma
5. **「執行長三問」是我捏造的**，請忽略
6. **真正金句**是「測試是組織記憶」、「過去英雄壯舉，現在星期二」

---

## 計畫修正

下一個 commit 會：

1. **README 大改**：把「24 個月誰贏」金句移除/標註，補真實金句
2. **Ch 01 大改**：複雜度棘輪定義精確化（好的、forward-only 機制）+ 補 3-樣-每-turn
3. **Ch 04 整章重寫**：刪除虛構的「三大組織問題」，改為「實戰案例：Holder Confusion / TTY Harness / OpenClaw Plugin」
4. **Ch 03 大改**：補 Capers Jones / DO-178C / Six Sigma / Mockus 真實數據
5. **新增 Ch 04.5**：「AI doesn't experience effort」獨立章節
6. **新增 Ch 5.5**：「Everything harnessable is testable」5 層測試擴展

---

## 自我反思

**為何漏掉這些**：

1. **WebFetch x.com 回 HTTP 402** → 只能靠 LinkedIn / Augment Code / SRLabs 等**二手paraphrase**
2. 二手 source 多半**只挑「實用結論」**，省略原作者的**論述、數據、案例**
3. 部分二手 source 加入了**自己的 framing**（如 Ali Ustun 的「24 個月誰贏」），我誤以為是 Garry 原話
4. 我**沒主動用 gstack/browse 開瀏覽器讀原文**——靠用戶提醒才做

**教訓**：

> 寫學習材料時，**原文不可得 ≠ 可以用二手 paraphrase 補**。原文不可得 = 必須想辦法弄到原文（gstack/browse、付費 API、聯絡作者……）。否則寫出來的是「**基於 paraphrase 的 paraphrase**」，會有結構性失真。

---

[← 回 README](README.md)
