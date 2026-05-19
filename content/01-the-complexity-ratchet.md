[← 回 README](../README.md)

# 第 01 章：複雜度棘輪 — 為何 AI 時代是 Inflection Point

> **核心句**：每個 feature 部署，都把 codebase 的複雜度往上推一格、且**鎖住不能往下**——這就是 ratchet（棘輪）。AI agent 把這個棘輪轉動速度提高 10 倍，**人類驗證能力不變**，缺口就是當前最致命的工程風險。

---

## 什麼是 ratchet（棘輪）

棘輪是個機械零件：可以往一個方向轉，但有齒輪鎖住，**不能反向**。

軟體裡的「**複雜度棘輪**」意思：

| 動作 | ratchet 方向 | 為何不能反向 |
|---|---|---|
| 部署一個新 feature | 複雜度 +1 | 用戶開始依賴，移除 = breaking change |
| 加一個 API endpoint | 依賴關係 +1 | 下游服務開始呼叫，下架要協調 |
| 加一張 database 表 | schema 複雜度 +1 | 開始有資料、開始有 query、開始有 ORM mapping |
| 加一個 config flag | 配置矩陣 ×2 | 兩個值都得測試 |
| 加一個 conditional branch | 程式路徑 ×2 | 兩條路徑都得驗證 |

**這些「+1」單個看都不大，但累積起來就是指數爆炸**。一個 100K 行的 codebase 可能有 2^30 條可能的 program path——人類**根本不可能**全部驗證。

---

## AI Agent 加速棘輪

過去：

```
Human dev 一週寫 1 個 feature → 每週棘輪 +1
Human reviewer 一週審 1-2 個 feature → 跟得上
```

AI agent 時代：

```
AI agent 一天寫 5 個 feature → 每天棘輪 +5
Human reviewer 一週還是審 1-2 個 → 跟不上
```

**Gap = (5×5 - 2) = 23 個 feature 的「未驗證債」每週累積。**

3 個月後，你 codebase 有 ~300 個未充分驗證的 feature 在 production。**這就是 ratchet bite hardest 的時刻**。

---

## 為何不能「先快後審」

❌ **錯誤直覺**：「先讓 AI 衝產量，之後再補 review。」

問題：
1. **棘輪鎖住**：上線後用戶開始用，補 review 發現問題 → fix 變成 breaking change → coordination cost 暴增
2. **複合債務**：未驗證 feature 之間互相依賴，越補越亂
3. **Onboarding 災難**：新工程師看不懂 codebase
4. **Incident 雪崩**：到某個臨界點，每天都有事故

✅ **正解**：**驗證能力先於產出能力**。先把驗證 baseline 拉高，**再**讓 AI agent 全速產出。

---

## 棘輪在哪些地方咬最深？

Garry 的觀察（從 YC portfolio 中常見的失敗模式）：

### 1. **業務邏輯層（business logic）**
- AI 改了 pricing 計算，沒人發現
- AI 改了 permission check，安全洞
- AI 改了狀態機 transition，產生不可達狀態

### 2. **資料 schema 變遷**
- AI 加 column 但忘記 migration
- AI 改 column type 但下游 service 沒同步
- AI 加 index 但忘了 production-scale 影響

### 3. **整合層（integrations）**
- AI 改 API contract 但 client 沒同步
- AI 加 webhook 但沒處理 retry/idempotency
- AI 加 timeout 但 ripple effect 到其他 service

### 4. **配置與 feature flag**
- AI 加 feature flag 但忘了清除舊路徑
- AI 改預設值但沒測試非預設值情境
- AI 加 env var 但 docs / staging / prod 不同步

### 5. **「無聲失敗」場景**
- AI 加 error handling 但 silent swallow exception
- AI 加 retry logic 但無限循環
- AI 加 cache 但 invalidation 邏輯錯

**這 5 個區域**就是「**ratchet bite hardest**」的具體位置。**你公司部署第一個 AI agent，應該優先針對其中之一加強驗證——不是隨機選用例**。

---

## Inflection Point 的判斷指標

什麼時候你公司「**進入 ratchet 困境**」？看這幾個信號：

| 信號 | 意義 |
|---|---|
| Bug count 連續 3 個月上升 | 棘輪轉得比 fix 快 |
| Incident 平均解時間（MTTR）變長 | codebase 已經沒人完全理解 |
| 新人 onboarding > 3 個月才能 first deploy | 複雜度超過合理範圍 |
| PR review 平均 > 48 小時 | reviewer 跟不上 |
| 「不敢動的 module」list 變長 | technical debt 累積 |
| Senior engineer 開始 burnout | 驗證壓力都壓在他們身上 |

**3 個以上信號出現 → 立即啟動 ratchet defense**（[Ch 08 路線圖](08-implementation-roadmap.md)）。

---

## 反直覺洞察：限速 AI agent 是錯的

很多 leader 看到 ratchet 問題，本能反應是「**那就限制 AI 寫程式速度**」。

**錯**。理由：

1. **競爭對手不會限速**——你限了你輸
2. **AI 寫程式的價值就在速度**——限速等於放棄價值
3. **限速不解決問題**——只是把棘輪轉慢、最終還是會咬

**正確反應**：**升級驗證能力**。讓驗證速度也 10x。

**怎麼讓驗證 10x**？答案是 [Ch 06](06-ai-as-test-writer.md)：**用 AI 寫測試**。

> 棘輪轉得多快，驗證就要多快。
> AI 加速生產 → AI 加速驗證。
> 唯一不可加速的是「**人類做最終 judgment**」，所以人類的 judgment 要被用在最高槓桿的地方。

---

## 棘輪 vs Technical Debt 的差別

| 維度 | Technical Debt | Complexity Ratchet |
|---|---|---|
| 來源 | 人類 shortcuts | AI 過量產出 |
| 速度 | 線性累積 | 指數累積 |
| 可逆性 | 可重構還清 | 鎖住、不可逆 |
| 主要表徵 | 程式碼變醜 | 行為變不可預測 |
| 處理方式 | sprint 預算還債 | 驗證能力升級 |

Tech debt 是「**程式品質下降**」；ratchet 是「**驗證能力被超車**」。兩者都該管，但 ratchet 更危險，**因為它在 production 才現形**。

---

## 本章小結

| 觀念 | 為何重要 |
|---|---|
| 棘輪是「上得去下不來」的單向結構 | 解釋為何複雜度只增不減 |
| AI agent 加速棘輪 10x | 看清為何過去的 testing playbook 不夠 |
| 5 個 bite hardest 區域 | 知道把第一道防線蓋在哪 |
| 6 個 inflection point 信號 | 自我診斷 |
| 限速 AI 是錯的 | 正確的反應是升級驗證能力 |
| ratchet ≠ tech debt | 兩個概念分開管 |

---

## 接下來

➡️ [Chapter 02: 驗證瓶頸 — 從 production 搬到 verification](02-verification-bottleneck.md)
