[← 回 README](../README.md) · [← Ch 04](04-three-executive-questions.md)

# 第 05 章：測試類型矩陣

> **核心句**：90% coverage 不是用一種測試達成的，是 6 種測試類型有機組合。每種測試抓不同層次的問題，搞混它們就等於建假防線。

---

## 6 種主要測試類型

| 類型 | 抓什麼 | 速度 | 寫的成本 | AI 寫的能力 |
|---|---|---|---|---|
| **Unit** | 函式邏輯 bug | ms | 低 | ⭐⭐⭐⭐⭐ |
| **Integration** | 模組間 contract 不一致 | 100ms | 中 | ⭐⭐⭐⭐ |
| **E2E** | 完整用戶流程 | 分鐘 | 高 | ⭐⭐⭐ |
| **Property-based** | 邊界 case 與不變性 | 秒 | 中 | ⭐⭐⭐⭐ |
| **Mutation** | 假測試（測試品質保證）| 分鐘 | 自動 | n/a（infra）|
| **Fuzz / Snapshot** | 未預期 input、UI 變動 | 秒 | 中 | ⭐⭐⭐⭐ |

---

## Type 1：Unit Tests

**定義**：測單一函式 / class / 純邏輯，無 IO、無 DB、無 network。

**典型例子**：

```python
def test_calculate_discount():
    assert calculate_discount(price=100, code='SAVE10') == 90
    assert calculate_discount(price=100, code=None) == 100
    assert calculate_discount(price=100, code='INVALID') == 100
```

**何時用**：

- 純函式（business logic、計算、轉換）
- Data structure 操作
- Validation 邏輯

**何時不用**：

- 牽涉外部依賴（DB / API）→ 用 Integration
- 跨模組互動 → 用 Integration
- UI 行為 → 用 E2E

**AI 寫 unit test 的特殊優勢**：

| 你給 AI | AI 給你 |
|---|---|
| 函式 signature + 1-2 個 example | 完整 test suite（normal / edge / error） |
| 函式實作 | 從實作 reverse engineer 出測試（注意：要 sanity check）|
| Bug fix PR | 對應的 regression test |

**Prompt 範本**：

```
Write comprehensive unit tests for this function.
Include:
- Normal cases (typical input)
- Edge cases (boundary values, empty, very large, very small)
- Error cases (invalid input, exceptions expected)
- At least 5 distinct test cases

Use pytest. Each test should have a descriptive name.

[paste function code]
```

---

## Type 2：Integration Tests

**定義**：測多個模組互動、有 DB / 有 API call、但仍在單一 process 內。

**典型例子**：

```python
def test_user_registration_flow(db, mailer):
    response = client.post('/register', json={
        'email': 'foo@bar.com', 
        'password': 'secret'
    })
    assert response.status_code == 201
    # DB 真寫入
    user = db.query(User).filter_by(email='foo@bar.com').first()
    assert user is not None
    # Email 真寄出
    assert mailer.last_sent.to == 'foo@bar.com'
```

**何時用**：

- API endpoint 行為
- DB schema + ORM
- 多 service 互動
- Auth / permission flow

**何時不用**：

- 純邏輯 → Unit
- 跨 service 跨機器 → E2E
- 第三方 API 真實互動 → 用 contract test，不是 integration

**重要原則**：**整合測試打真 DB（test container），不要全 mock**。

過去 mock 流行是因為設置真 DB 慢。**現在用 Docker / testcontainers，一切都是 throwaway**。一條來自 12-factor 維護人的觀察：

> **「Mock 通過了，但 prod migration 失敗」**——這就是 mock-everywhere 的代價。

---

## Type 3：E2E Tests

**定義**：模擬真實用戶從外部操作整個系統。瀏覽器 / API client → 多 service → DB。

**典型例子**（用 Playwright）：

```javascript
test('user can complete checkout', async ({ page }) => {
  await page.goto('/');
  await page.click('text=Add to cart');
  await page.click('text=Checkout');
  await page.fill('input[name=email]', 'test@example.com');
  await page.fill('input[name=card]', '4242424242424242');
  await page.click('text=Pay $99');
  await expect(page.locator('text=Thank you')).toBeVisible();
});
```

**何時用**：

- 關鍵用戶流程（signup / checkout / cancellation）
- Cross-service 流程
- 看得到 / 摸得到的 UI 行為

**何時不用**：

- Edge case 細節 → Unit
- 模組互動 → Integration
- 慢、貴、不穩定，**不該當主力**

**E2E 的數量原則**：

> **5-15 個 critical journey** 涵蓋核心商業流程。**不要超過 30 個**——E2E 跑太慢，flaky 率高，維護成本超過邊際效益。

---

## Type 4：Property-based Tests

**定義**：不是測「**對某個 input X，輸出是 Y**」，而是測「**對所有 input，某個 property 永遠成立**」。

**典型例子**（用 Hypothesis）：

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_idempotent(lst):
    # property: sort 兩次跟 sort 一次結果一樣
    assert sorted(sorted(lst)) == sorted(lst)

@given(st.lists(st.integers()))
def test_sort_preserves_length(lst):
    # property: sort 後長度不變
    assert len(sorted(lst)) == len(lst)
```

Property-based test 工具會自動生成數百個 input 試圖打破 property。

**何時用**：

- 數學性質明確的函式（sort、merge、compress）
- Idempotency（呼叫多次效果一樣）
- Invariant（系統不變性）
- Round-trip（encode + decode = identity）

**何時不用**：

- 沒有明確 mathematical property 的 UI / business logic
- 只關心特定 input → Unit

**AI 寫 property-based test 的能力**：⭐⭐⭐⭐。**Prompt**：

```
For this function, identify 3-5 mathematical/logical properties that should always hold.
Then write Hypothesis (Python) / fast-check (JS) tests for each.

[paste function]
```

---

## Type 5：Mutation Testing

**定義**：**測試你的測試**。工具會故意改你的 code（變 + 成 -、變 < 成 <=）然後跑你的測試。**如果測試沒抓到，你的測試是假的**。

**工具**：

- Python：`mutmut`, `mutpy`
- JS：`Stryker`
- Java：`PIT`
- Go：`go-mutesting`

**典型輸出**：

```
Total mutants: 234
Killed (caught by tests): 187
Survived (NOT caught): 47   ← 假測試！
Mutation score: 80%
```

**Mutation score 怎麼解讀**：

| 分數 | 評估 |
|---|---|
| < 50% | 測試大部分是假的 |
| 50-70% | 一半測試有效 |
| 70-85% | 中位數 |
| 85-95% | 良好 |
| > 95% | 卓越 |

**Garry 的 90% coverage** 應該對應 **mutation score ≥ 75%**。否則就是 gaming。

**何時用**：

- Critical path 一定要跑
- Quarterly 全 codebase 跑
- PR 級別跑 diff mutation

**何時不用**：

- Mutation testing 很慢（×10 test runtime），不適合 every-commit

---

## Type 6：Fuzz Testing & Snapshot Testing

### Fuzz

**定義**：用隨機 / structured-random input 砸你的函式，看會不會 crash / hang。

**典型例子**：

```python
@hypothesis.given(st.binary())
def test_parse_doesnt_crash(data):
    try:
        my_parser.parse(data)
    except (ValueError, ParseError):
        pass  # expected errors are OK
    # 但 segfault / hang / unhandled exception 就 fail
```

**何時用**：

- Parser、deserialization、user input handling
- 安全敏感 code

**AI 寫 fuzz test**：⭐⭐⭐⭐。Prompt:

```
Write fuzz tests for this parser. 
Use Hypothesis to generate random bytes/strings.
Verify it never crashes (only raises expected exceptions).

[paste parser]
```

### Snapshot

**定義**：把 function output 第一次跑的結果存起來（snapshot），之後每次跑跟 snapshot 比，不同就 fail。

**何時用**：

- UI 渲染（React component → HTML snapshot）
- 大型 JSON / YAML 輸出
- 報告生成

**陷阱**：snapshot 容易「**meaningless update**」——壞掉直接 `--update-snapshots` 一鍵更新，等於沒測。**人類 review 每次 snapshot 更新**才有意義。

---

## 比例分配建議

90% coverage 的具體組合（典型 web service）：

```
70% Unit tests                     ← 速度快、寫得多、AI 友善
20% Integration tests              ← 抓 contract 問題
 5% E2E tests                      ← 5-15 個 critical journey
 3% Property-based tests           ← 關鍵函式
 1% Fuzz / Snapshot                ← 特殊場景
─────
99%
```

**外加**：

- Mutation score：critical path 100%、aggregate ≥ 75%
- Performance test：critical path 有 latency SLA 檢核

---

## 反例：常見比例錯誤

### 反例 1：100% E2E

「**我們只寫 E2E，因為 unit test 太細**」

問題：
- E2E 跑超慢（30 分鐘 CI），開發者開始 skip
- Flaky 高，造成 CI noise，慢慢沒人信任
- Bug 出現時找不到根因（E2E fail 通常只說「**某處壞了**」）

### 反例 2：100% Unit

「**我們只寫 unit test，因為它快**」

問題：
- 模組間 contract 不一致時抓不到
- DB migration 不對抓不到
- API 改 schema 抓不到

### 反例 3：100% Mock

「**所有測試都 mock 外部依賴，因為慢**」

問題：
- Mock 與 real 行為 drift（mock 永遠成功、real 偶爾失敗）
- Refactor 時 mock 沒同步 → 假成功
- Mock pass 但 prod fail（最致命）

---

## 測試類型 vs ratchet 區域對照

回到 [Ch 04 Q2](04-three-executive-questions.md#q2where-does-the-ratchet-bite-hardest)：你應該在 ratchet bite hardest 的區域加最多測試。**不同 ratchet 區用不同測試類型**：

| Ratchet 區 | 主要測試類型 |
|---|---|
| 業務邏輯 | Unit + Property-based |
| Data schema | Integration + Migration test |
| 整合層 | Integration + Contract test |
| 配置 / feature flag | E2E + Snapshot |
| 無聲失敗 | Mutation + Fuzz |

對症下藥。

---

## 本章小結

| 類型 | 用途 | 比例 |
|---|---|---|
| Unit | 函式邏輯 | 70% |
| Integration | 模組互動 / DB / API | 20% |
| E2E | 關鍵用戶流程 | 5%（5-15 個）|
| Property | 數學性質 | 3% |
| Fuzz / Snapshot | 特殊 | 1% |
| Mutation | 測試品質保證 | 跑在上述之上 |

90% line coverage **加上** mutation score ≥ 75% = 真 90%。

---

## 接下來

➡️ [Chapter 06: AI 當測試作者 — 具體 SOP](06-ai-as-test-writer.md)
