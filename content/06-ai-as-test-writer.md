[← 回 README](../README.md) · [← Ch 05](05-test-types-matrix.md)

# 第 06 章：AI 當測試作者 — 具體 SOP

> **核心句**：「**用 AI 寫測試**」聽起來簡單，做不好就是「**假覆蓋率**」。本章給你 6 個工作流，把 AI 從「**生產更多假測試的工具**」變成「**真把覆蓋率拉到 90% 且品質可信**」的工具。

---

## AI 寫測試的能力分布

不是所有測試類型 AI 都一樣強：

| 測試類型 | AI 能力 | 為何 |
|---|---|---|
| Unit test for pure function | ⭐⭐⭐⭐⭐ | LLM 強項：規律明確 |
| Integration test | ⭐⭐⭐⭐ | 需要 context（fixtures、env）|
| E2E test (Playwright) | ⭐⭐⭐ | 需要 UI knowledge + selector 穩定性 |
| Property-based test | ⭐⭐⭐⭐ | LLM 知道常見 properties |
| Mutation test | n/a | 是 infra 不是寫測試 |
| Fuzz test | ⭐⭐⭐⭐ | 生成 strategies 容易 |
| Snapshot test | ⭐⭐⭐⭐⭐ | 純機械 |
| Regression test from bug | ⭐⭐⭐⭐⭐ | LLM 很強，「**重現再防**」邏輯清楚 |

---

## 工作流 1：補測試到 90%

**情境**：你的 codebase 目前 65% coverage，要拉到 90%。

### SOP

```
Step 1: 跑 coverage 找未覆蓋的 file/line
Step 2: 用 LLM 為每個 file 生成測試（批次）
Step 3: 跑測試，看 pass 率
Step 4: 失敗的測試人類 review（通常是 LLM 誤解 implementation）
Step 5: 跑 mutation testing 抓假測試
Step 6: 假測試重生（給 LLM 看為何被 mutate 殺死）
```

### 具體腳本

```bash
#!/bin/bash
# generate-tests.sh

# Step 1: 找未覆蓋檔案
pytest --cov=src --cov-report=json:.coverage.json
UNCOVERED_FILES=$(jq -r '.files | to_entries[] | select(.value.summary.percent_covered < 90) | .key' .coverage.json)

# Step 2: 為每個檔案生成測試
for file in $UNCOVERED_FILES; do
  echo "Generating tests for $file..."
  
  # 把檔案內容 + existing tests 餵給 LLM
  EXISTING_TESTS=$(find tests/ -name "*$(basename $file .py)*" -exec cat {} \;)
  
  claude code "
Write comprehensive pytest tests for this file to reach 90% coverage.

EXISTING TESTS (don't duplicate):
$EXISTING_TESTS

SOURCE FILE:
$(cat $file)

REQUIREMENTS:
- Use pytest
- Test all branches (not just happy path)
- Use fixtures where appropriate
- Each test has descriptive name
- Output only the test code, no explanation
" > tests/test_$(basename $file)
done

# Step 3: 跑測試
pytest --cov=src --cov-fail-under=90
```

### Common pitfall

❌ **LLM 直接從 implementation 推測試**——這會「**通過但無意義**」（測試 = implementation 的鏡像）

✅ **LLM 從 docstring / spec / behavior 推測試**——這才能抓到 implementation bug

加 prompt：
```
IMPORTANT: Write tests based on the docstring and intended behavior,
NOT by mirroring the implementation. If implementation has bugs,
the test should catch them.
```

---

## 工作流 2：每個 bug fix 都寫 regression test

**情境**：production 出 bug → fix → 防止再發生。

### SOP

```
Step 1: 寫一個 failing test 重現 bug（在 fix 之前）
Step 2: 寫 fix
Step 3: 確認 test 通過
Step 4: 把 test commit 進 regression suite
```

### AI 加速

修 bug 時 prompt：

```
Here's a bug report:
[paste bug description / stack trace / Sentry issue]

Here's the relevant code:
[paste code]

Tasks:
1. Write a failing pytest test that reproduces this bug (before fixing)
2. Then write the minimal fix
3. Then explain why the test would have caught this earlier

Output:
- test_<descriptive>.py (failing test)
- fix in src/<file>.py
- 1-paragraph postmortem
```

**結果**：每修一個 bug，**多一個 regression test**。Coverage 跟 quality 一起拉高。

---

## 工作流 3：PR 時自動生成 diff coverage 測試

**情境**：保證每個 PR 不降低 coverage。

### CI 設定

```yaml
# .github/workflows/test.yml
name: Test
on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 需要 git history
      
      - run: pip install -r requirements.txt
      
      - name: Run tests with coverage
        run: pytest --cov=src --cov-report=xml
      
      - name: Diff coverage check
        run: |
          diff-cover coverage.xml --compare-branch=origin/main \
            --fail-under=90  # diff 必須 90% 覆蓋
      
      - name: Total coverage gate
        run: |
          coverage report --fail-under=85  # 整體不能低於 85（現在）
```

### 如果 PR diff coverage 不足

提示 PR 作者用 AI 補：

```bash
# .githooks/post-commit (optional)
DIFF_COV=$(diff-cover coverage.xml --compare-branch=origin/main | grep "Coverage:" | awk '{print $2}')
if [ $(echo "$DIFF_COV < 90" | bc) -eq 1 ]; then
  echo "⚠️  Diff coverage $DIFF_COV < 90%"
  echo "Run: claude code 'generate tests for new uncovered lines'"
fi
```

---

## 工作流 4：用 AI 抓「假測試」

**情境**：90% coverage 達到了，但你懷疑很多測試是「**`assert True`** 等級的假測試」。

### 三層防線

#### 防線 1：AI code-review 測試

每個 PR 的測試檔案，跑 AI 評審：

```bash
claude code-review --focus=tests "
Review these test files for quality issues:
- assert True / trivial assertions
- Tests that don't actually verify behavior
- Mocked everything (mock-mock-pass)
- Tests that mirror implementation (no value)
- Missing edge cases

Files: [list]
"
```

#### 防線 2：Mutation testing 跑批次

```bash
# 每週日跑全 codebase mutation
mutmut run --paths-to-mutate src/
mutmut results > mutation-report.txt

# 警報：mutation score < 75%
SCORE=$(jq '.score' mutation-results.json)
if [ $(echo "$SCORE < 75" | bc) -eq 1 ]; then
  slack-notify "#engineering" "⚠️ Mutation score dropped to $SCORE%"
fi
```

#### 防線 3：人類 review 隨機抽樣

每週 senior engineer 隨機抽 10 個測試，跑這 prompt：

```
For each test below, evaluate:
1. Does it actually verify behavior? (Y/N)
2. Would it catch a real bug in the function? (Y/N)
3. If implementation had off-by-one error, would this test catch it? (Y/N)

[paste 10 random tests]
```

如果 < 7/10 是真測試，**你的測試套基礎有問題**——回去用工作流 1 重做。

---

## 工作流 5：從 spec / PRD 寫 E2E

**情境**：產品經理寫了一份 PRD，工程師要實作 + 測試。

### 反 patternsuly：先寫 code 後補 E2E

問題：
- 工程師寫完 code，testing 是「**驗證 code 做了我寫的東西**」（同義反覆）
- 沒抓到「**code 沒做 PRD 寫的東西**」

### Pattern：PRD → E2E → Code → Iteration

```
Step 1: PRD 完成
Step 2: 用 AI 從 PRD 生成 E2E test（用 Playwright / Cypress）
Step 3: E2E test 跑，全部 fail（because no code yet）
Step 4: 工程師寫 code 讓 E2E pass
Step 5: PRD 改動，E2E 同步改動
```

### AI Prompt

```
Here's the PRD:
[paste PRD]

Generate Playwright E2E tests covering:
1. Happy path (the primary user journey)
2. Error scenarios (what user sees when X fails)
3. Edge cases (empty state, max state, etc.)

Tests should be:
- Independent (no shared state)
- Deterministic (no flaky timing)
- Self-cleanup (set up + tear down)

Output: Playwright TypeScript code, ready to run.
```

**這就是 outside-in TDD 在 AI 時代的具體形式**。

---

## 工作流 6：AI 生成 Property-based test

**情境**：你有個 critical 函式，想確保所有 input 行為正確，不只是手寫的 case。

### Prompt

```
Function:
[paste function with docstring]

Tasks:
1. Identify 3-5 mathematical/logical PROPERTIES this function should always satisfy.
2. For each property, write a Hypothesis test.
3. Explain in 1 sentence per property why it must hold.

Format:
- # Property 1: <description>
  # Why: <reasoning>
  @given(...)
  def test_property_1(...): ...

- # Property 2: ...

Example properties to consider:
- Idempotency: f(f(x)) == f(x)
- Commutativity: f(a, b) == f(b, a)
- Identity element: f(x, identity) == x
- Inverse: g(f(x)) == x
- Length preservation: len(f(xs)) == len(xs)
- Monotonicity: a < b → f(a) <= f(b)
- Determinism: f(x) == f(x) given same input
```

---

## 整合：AI 測試生成的 daily SOP

把這些工作流組合成日常：

| 觸發 | 動作 | 工具 |
|---|---|---|
| PR submit | AI 生成 diff coverage 測試 | CI hook + LLM |
| Bug report | AI 寫 regression test | 工作流 2 |
| New PRD | AI 生成 E2E skeleton | 工作流 5 |
| Refactor critical function | AI 加 property-based test | 工作流 6 |
| 每週 | Mutation testing 跑批次 | mutmut + cron |
| 每月 | Senior engineer 隨機抽樣 review | 工作流 4 防線 3 |
| 每季 | 全 codebase verification audit | 用 [Ch 04 檢核表](04-three-executive-questions.md) |

---

## AI 寫測試的 5 個鐵則

1. **AI 寫測試 ≠ 不需要 review**：每個 AI 生成的測試都要人類過 1 眼
2. **Mutation testing 是必要的「**測試品質檢核員**」**：沒有它，AI 寫的假測試會大量滲入
3. **Prompt 要求「**從 spec 而非 implementation**」**：避免測試與實作同義反覆
4. **小批次生成，不要整 codebase 一次餵**：LLM context 越短品質越穩
5. **每個 mutation killed by your test = 你的測試品質指標**：把這個數字當成 KPI

---

## 反例集錦

### 反例 1：「我用 AI 一鍵生成所有測試，coverage 90%」

**問題**：mutation score 可能 < 50%。一堆假測試。

**解**：mutation testing 必跑，作為「**真假 90%**」的二階檢核。

### 反例 2：「AI 生成的測試我都接受，因為它們 pass」

**問題**：通過 ≠ 有意義。可能是 `assert True` 或測 implementation 的鏡像。

**解**：每個測試 review 「**這在抓什麼 bug？**」答不出來就丟。

### 反例 3：「我用 AI mock 所有外部依賴」

**問題**：mock 永遠成功，prod 偶爾失敗。

**解**：用 testcontainers 跑真 DB / 真 service，AI 只 mock 慢的、不穩的、不重要的。

---

## 本章小結

| 工作流 | 觸發 | 結果 |
|---|---|---|
| 1. 補測試到 90% | 一次性 migration | 65% → 90% in 3 週 |
| 2. Bug fix → regression | 每次 bug | 防止 regression |
| 3. PR diff coverage | 每個 PR | 不降低 coverage |
| 4. 抓假測試 | 每週 / 隨機 | 保品質 |
| 5. PRD → E2E | 新 feature | Outside-in 開發 |
| 6. Property-based | Refactor | 數學保證 |

---

## 接下來

➡️ [Chapter 07: 反模式 — 假測試陷阱](07-anti-patterns.md)
