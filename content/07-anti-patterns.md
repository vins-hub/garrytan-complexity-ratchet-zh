[← 回 README](../README.md) · [← Ch 06](06-ai-as-test-writer.md)

# 第 07 章：反模式 — 假測試陷阱

> **核心句**：80% 覆蓋率比 60% 危險——因為它讓你**以為**有防線。本章列 10 個 anti-patterns，幫你識別「**看起來像測試但實際上是裝飾**」的陷阱。

---

## 反模式 #1：`assert True`

❌ **錯**：
```python
def test_calculate_price():
    result = calculate_price(items=[item1, item2])
    assert True  # 啊？沒測什麼
```

**為什麼壞**：執行了 code，但沒驗證任何行為。Coverage 漂亮，零價值。

**識別信號**：
- `assert True`
- `assert 1 == 1`
- `assert response is not None`（只測沒 crash）

✅ **正解**：
```python
def test_calculate_price():
    result = calculate_price(items=[
        Item(price=100), Item(price=200)
    ])
    assert result == 300
```

---

## 反模式 #2：測試是 Implementation 的鏡像

❌ **錯**：
```python
# implementation
def calculate_tax(price):
    return price * 0.08

# test
def test_calculate_tax():
    assert calculate_tax(100) == 100 * 0.08  # ← 同一個 0.08
```

**為什麼壞**：如果 `0.08` 應該是 `0.10`（程式 bug），測試也是 `0.08`，永遠抓不到。**測試跟著程式一起錯**。

✅ **正解**：用 **explicit expected value**：
```python
def test_calculate_tax():
    # 期望：8% 稅率（從 spec）
    assert calculate_tax(100) == 8.0
    assert calculate_tax(50) == 4.0
    assert calculate_tax(1000) == 80.0
```

如果 spec 改 tax rate，你**只改 test 對應的 expected**，不會自動跟程式錯。

---

## 反模式 #3：Mock 整個世界

❌ **錯**：
```python
def test_user_signup(mocker):
    mocker.patch('db.User.save')        # mock DB
    mocker.patch('mailer.send')          # mock email
    mocker.patch('redis.set')             # mock cache
    mocker.patch('sentry.capture')        # mock error tracker
    mocker.patch('analytics.track')       # mock analytics
    
    user_signup(email='foo@bar.com', password='x')
    
    db.User.save.assert_called_once()    # 只驗證「**call 過了**」
```

**為什麼壞**：你測的不是「**user signup 正確**」，是「**這些 mock 被呼叫了**」。實際 DB schema 改了、real mailer 改了，這測試還是 pass。**Mock pass，prod fail**。

✅ **正解**：用 testcontainers 跑真 DB + 真 service：
```python
def test_user_signup(db_container):  # 真 PostgreSQL container
    user_signup(email='foo@bar.com', password='x')
    
    # 從真 DB 查
    user = db_container.session.query(User).filter_by(
        email='foo@bar.com'
    ).first()
    assert user is not None
    assert user.password_hash != 'x'  # 確實被 hash 了
```

**Mock 是工具不是教義**。Mock 慢的、外部的、隨機的；不要 mock 你的 schema、你的核心邏輯。

---

## 反模式 #4：超大 Test，沒有 isolation

❌ **錯**：
```python
def test_full_workflow():
    # 30 行 setup
    user = create_user(...)
    org = create_org(user)
    project = create_project(org)
    issue = create_issue(project)
    
    # 50 行 action
    add_comment(issue, ...)
    assign_user(issue, ...)
    update_status(issue, ...)
    
    # 20 行 assert
    assert issue.comments[0].text == ...
    assert issue.assignee == ...
    assert issue.status == ...
```

**為什麼壞**：
- 任一行壞，整個 test fail
- 不知道是哪個 action 出問題
- 失敗時 debug 1 小時起跳
- LLM 也理解不了「**這個 test 到底在測什麼**」

✅ **正解**：每個 test 一個 action：
```python
def test_add_comment(issue_factory):
    issue = issue_factory()
    add_comment(issue, 'hello')
    assert issue.comments[0].text == 'hello'

def test_assign_user(issue_factory, user_factory):
    issue = issue_factory()
    user = user_factory()
    assign_user(issue, user)
    assert issue.assignee == user

# ... 各自獨立
```

**Test 一行 setup、一行 action、一行 assert** 是聖經。

---

## 反模式 #5：Skipping Tests 卻不刪

❌ **錯**：
```python
@pytest.mark.skip(reason="flaky, fix later")
def test_complex_scenario():
    ...

@pytest.mark.xfail
def test_another_thing():
    ...
```

**為什麼壞**：
- Skip 的測試**從不被刪除也不被修**，永遠在 codebase 裡裝樣子
- Coverage 報告仍可能算入（看工具）
- 新工程師看到 skip 不知道是不是還有意義
- 跟 `assert True` 一樣等於沒測

✅ **正解**：
- 一週內修不好 → **刪掉**
- 真的需要保留 → 加 **expiry 與 owner**：

```python
@pytest.mark.skip(
    reason="DB locked race condition - tracked in JIRA-123",
    # SKIP_EXPIRES: 2026-06-01
    # OWNER: jane@example.com
)
def test_complex_scenario():
    ...
```

跟 CI 整合：超過 expiry 還 skip → CI fail，強迫處理。

---

## 反模式 #6：用 try/except 包整個 Test Body

❌ **錯**：
```python
def test_thing():
    try:
        result = my_function()
        assert result == expected
    except Exception:
        pass  # 包住所有錯誤
```

**為什麼壞**：你的 test 永遠 pass，不管 `my_function` 是不是真壞了。

✅ **正解**：只 catch **預期** 的 exception，並斷言它**確實**發生：
```python
def test_thing_raises_on_invalid():
    with pytest.raises(ValueError, match="invalid input"):
        my_function(invalid_arg)
```

---

## 反模式 #7：依賴 Test 執行順序

❌ **錯**：
```python
def test_1_create_user():
    global user_id
    user_id = create_user('foo')
    assert user_id

def test_2_login_user():
    response = login(user_id)  # ← 依賴 test_1 跑過
    assert response.status == 200
```

**為什麼壞**：
- 換順序就壞
- 平行跑就壞
- 沒辦法 isolate debug 單一 test

✅ **正解**：每個 test 自己 setup（用 fixture 共享）：
```python
@pytest.fixture
def user():
    return create_user('foo')

def test_login(user):
    response = login(user.id)
    assert response.status == 200
```

---

## 反模式 #8：Flaky test 容忍症

❌ **錯**：CI 有 5% test flaky。團隊「**習以為常**」，看到 fail 就 retry，過了就 merge。

**為什麼壞**：
- 真 bug 跟 flaky 混在一起，分不清
- 信任度崩盤——CI fail 大家先 retry
- 慢慢更多 test 變 flaky，但沒人警告
- 直到某天 prod 壞了，回查發現原來那個 flaky test 真的在抓 bug

✅ **正解**：**Zero tolerance**

```bash
# 每個 PR 跑 3 次，3 次都 pass 才算 pass
pytest --reruns=0 --maxfail=1  # 嚴格模式

# 偵測 flaky
pytest --count=10  # 跑 10 次，有任何不一致就 fail
```

Flaky test 出現：
1. 立即 isolate（mark + 報警，**不直接 disable**）
2. 24 小時內 root cause
3. 修不好就刪

---

## 反模式 #9：以 LoC 為 Test KPI

❌ **錯**：「**這個 sprint 我們寫了 500 行 test**」當成功標記。

**為什麼壞**：
- 鼓勵寫**多**而非**好**
- AI 一秒鐘可以寫 5000 行 garbage test
- LoC 跟「**抓到的 bug 數**」沒關係

✅ **正解**：用這些 KPI：
- **Mutation score**（測試品質）
- **Regression test count**（每修一個 bug 加一個）
- **Critical path coverage**（高風險區 100%）
- **CI confidence**（綠燈才能上 prod 的信任度）

LoC 不要當 KPI。

---

## 反模式 #10：「我們有 manual QA 所以不需要這麼多自動測試」

❌ **錯**：「**Manual QA team 跑 regression suite，自動測試 60% 就夠**」

**為什麼壞**：
- Manual QA 一週只能跑 1-2 個完整 regression cycle
- AI agent 一週可以塞 50 個 PR
- 數學上 manual QA **永遠追不上**

✅ **正解**：把 manual QA 角色升級：

| Manual QA 從做 | 升級成做 |
|---|---|
| 跑 regression（重複勞動）| 設計新測試（高槓桿）|
| 手動驗證 happy path | 寫 exploratory testing tool |
| 寫 bug report | 寫 root cause analysis + 防 regression test |
| Test 執行 | Test architecture |

**Manual QA 沒有消失，是被「升維」**——做機器做不了的事。

---

## 反模式對照速查

| # | 反模式 | 識別信號 | 解法 |
|---|---|---|---|
| 1 | assert True | 沒有 assertion | 用 explicit value |
| 2 | Implementation 鏡像 | Test 跟 code 用同 constant | 用 spec-derived value |
| 3 | Mock 整個世界 | 一堆 patch | 用 testcontainers |
| 4 | 巨型 test | 100+ 行 | 拆成單一 action |
| 5 | 不刪的 skip | `@pytest.mark.skip` 沒 expiry | 加 expiry + owner |
| 6 | try/except 包 test | bare except | 用 `pytest.raises` |
| 7 | 依賴順序 | global state | 用 fixture |
| 8 | Flaky 容忍 | retry 文化 | Zero tolerance |
| 9 | LoC 當 KPI | 周報講行數 | 用 mutation score |
| 10 | Manual QA 替代 | 60% 覆蓋率自滿 | 升級 QA 角色 |

---

## 自我檢查腳本

把這個跑你 codebase 一遍：

```bash
#!/bin/bash
# detect-anti-patterns.sh

echo "=== 反模式 #1: assert True / 平凡 assertion ==="
grep -rn 'assert True\|assert 1 == 1\|assert None is None' tests/ || echo "✓ none"

echo "=== 反模式 #5: skip 沒 expiry ==="
grep -rn 'pytest.mark.skip\|@skip' tests/ | grep -v 'SKIP_EXPIRES' || echo "✓ all good"

echo "=== 反模式 #6: bare except ==="
grep -rn 'except:\|except Exception:\s*pass' tests/ || echo "✓ none"

echo "=== 反模式 #7: global state in tests ==="
grep -rn '^global ' tests/ || echo "✓ none"

echo "=== Test 行數分布 (找太大的 test) ==="
awk '/^def test_/{n=NR; name=$0} /^def [^t]/{if(n)print NR-n, name; n=0}' tests/*.py | sort -n -r | head -10

echo "=== 跑 mutation testing (基礎品質指標) ==="
mutmut run 2>&1 | tail -3
```

---

## 本章小結

| 反模式 | 危害程度 | 修復難度 |
|---|---|---|
| #1 assert True | 高 | 低 |
| #2 Impl 鏡像 | **致命**（看不到 bug） | 中 |
| #3 Mock 全世界 | **致命**（mock pass prod fail） | 中 |
| #4 巨型 test | 中（debug 痛苦） | 低 |
| #5 不刪 skip | 中（累積技術債） | 低 |
| #6 try/except 包 test | 高 | 低 |
| #7 順序依賴 | 中 | 低 |
| #8 Flaky 容忍 | **致命**（信任度崩盤） | 高 |
| #9 LoC KPI | 中 | 低 |
| #10 Manual QA 替代 | 中 | 中 |

修這 10 個之前，講 90% 覆蓋率都是空話。

---

## 接下來

➡️ [Chapter 08: 12 週實施路線圖](08-implementation-roadmap.md)
