[← 回 README](../README.md) · [← Ch 08](08-implementation-roadmap.md)

# 第 09 章：跟 12-Factor Agents 的關聯

> **核心句**：Garry Tan 的「90% test coverage」跟 Dex Horthy 的「12-factor agents」乍看談不同問題。實際上它們是「**LLM 時代 production-grade 軟體**」這枚硬幣的兩面——12-factor 是建造側、90% 是驗證側。沒有任一面，AI 應用都上不了 customer-facing production。

---

## 兩條觀念的關係

| 觀念 | 提出者 | 解決問題 |
|---|---|---|
| 12-factor agents | Dex Horthy | **怎麼建造**可控的 LLM 應用 |
| 90% test coverage | Garry Tan | **怎麼驗證** AI 產出的軟體 |

**12-factor 是「**寫對**」**，**90% 是「**確認沒寫錯**」**。一個是 input quality、一個是 output quality。

---

## 為什麼這兩條必須同時存在

只有 12-factor 而沒有 90%：
- Agent 架構漂亮、可控、可暫停
- **但你不知道它在 production 會不會壞**——AI agent 寫的 deterministic 程式碼可能有 bug 你沒抓到
- 結果：可控的 agent 還是會出 incident

只有 90% 而沒有 12-factor：
- Agent 寫的 code 都被測試覆蓋
- **但 agent 本身是黑盒，走火入魔時沒人能介入**
- 結果：production agent 跑 5 分鐘就 spin out

**兩條一起做**，你才得到「**production-grade AI 應用**」。

---

## 對應映射

### Garry Q1 (verification coverage) ↔ 12-factor Factor 9 (Compact Errors)

> Garry Q1: What is our verification coverage on AI outputs?
> 
> Factor 9: 把錯誤壓進 context window，讓 LLM 自癒

兩條都在問「**錯誤怎麼被抓到 + 修復**」。

- **Factor 9** 處理「**運行時錯誤**」：tool call 失敗時，LLM 看到 stack trace 後嘗試自我修復
- **Garry Q1** 處理「**部署前錯誤**」：測試體系抓到 bug，在 PR 階段就 reject

兩個都是「**錯誤捕獲**」，只是在不同 stage。

---

### Garry Q2 (ratchet bite hardest) ↔ 12-factor Factor 10 (Small Focused Agents)

> Garry Q2: Where does the ratchet bite hardest?
> 
> Factor 10: Small, focused agents（3-10 step）

兩條都在處理「**複雜度爆炸**」。

- **Factor 10** 從**結構**上避免複雜度：把 agent 拆小，每個只處理一件事
- **Garry Q2** 從**驗證**上對抗複雜度：找出複雜度最大的區域，集中防線

**結構限制 + 驗證強化**，是對抗複雜度的兩支腳。

---

### Garry Q3 (who owns) ↔ 12-factor Factor 7 (Contact Humans with Tools)

> Garry Q3: Who owns catching regressions?
> 
> Factor 7: 用 tool call 聯絡人類（人類介入機制）

兩條都在處理「**人類在 loop 的角色**」。

- **Factor 7** 設計**運行時**人類介入：agent 跑到關鍵點主動找人類
- **Garry Q3** 設計**組織**人類介入：明確誰負責抓 regression

**人類不是裝飾，是 critical infrastructure**。

---

## 完整 production architecture

把兩條觀念合成完整架構：

```
                        Production-Grade AI Application
        ┌───────────────────────────────────────────────────────────┐
        │                                                            │
        │   ★ BUILDING SIDE (12-factor agents) ★                      │
        │                                                            │
        │   Factor 1: NL → tool call          Factor 8: control flow │
        │   Factor 2: own prompts             Factor 9: compact errors│
        │   Factor 3: own context             Factor 10: small focused│
        │   Factor 4: tools = structured     Factor 11: trigger any  │
        │   Factor 5: unify state             Factor 12: stateless   │
        │   Factor 6: launch/pause/resume                            │
        │   Factor 7: contact humans                                  │
        │                                                            │
        ├───────────────────────────────────────────────────────────┤
        │                                                            │
        │   ★ VERIFICATION SIDE (Garry Tan 90%) ★                     │
        │                                                            │
        │   90% line coverage     · 75% mutation score                 │
        │   Critical path 100%    · Diff coverage gate                 │
        │   3 layers verification · Quality Lead with veto             │
        │   AI as test writer     · Mutation testing CI gate           │
        │                                                            │
        └───────────────────────────────────────────────────────────┘
                                  ↓
                       PRODUCTION-GRADE AI APPLICATION
```

---

## 工程組織的 4 種狀態

根據兩條原則的完成度，組織分 4 種：

```
                        ↑ 12-factor agents adoption
                        │
        Type C          │          Type D ★ (你要去的地方)
        過度工程        │          Production-ready
        架構美觀但      │          架構好 + 驗證強
        測試弱          │          
                        │          
        ━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━→ 90% test coverage
                        │          
        Type A          │          Type B
        Early stage     │          Reactive QA
        簡單 prompt loop│          測試多但 agent 失控
        + 弱測試        │          
                        │          
                        ↓
```

**多數 AI 新創 stuck 在 Type A（沒測試也沒架構）**
**部分大公司在 Type C（架構漂亮但沒測試）**
**少數成熟公司在 Type B（測試多但 agent 鬆散）**
**極少數在 Type D（兩者都做）**

**Garry + Dex 的方法論合起來，就是把你從任何起點推到 Type D 的 playbook**。

---

## 整合範例：12-factor agent 的 verification 體系

把兩條觀念套到一個具體 deploybot 案例：

### 12-factor 設計

```python
# 標準 12-factor agent 架構
class DeployAgent:
    def handle_next_step(self, thread: Thread):
        while True:
            next_step = self.llm.determine_next_step(thread_to_prompt(thread))
            
            if next_step.intent == 'deploy_backend_to_prod':
                # high-stakes: 需要人類審批
                self.request_human_approval(next_step)
                self.save_thread(thread)
                break
            elif next_step.intent == 'list_git_tags':
                tags = fetch_git_tags()
                thread.events.append({'type': 'list_git_tags_result', 'data': tags})
                continue
            # ... 其他 case
```

### Garry 90% 驗證體系

**對應的測試 suite**：

```python
# Unit tests
def test_handle_next_step_deploy_intent():
    """高 stakes deploy intent 必須 break loop 等人類"""
    agent = DeployAgent(mock_llm_returning('deploy_backend_to_prod'))
    thread = Thread()
    agent.handle_next_step(thread)
    assert mock_request_human_approval.called
    assert thread.is_saved

def test_handle_next_step_list_tags_intent():
    """List tags 同步執行不 break"""
    agent = DeployAgent(mock_llm_returning('list_git_tags'))
    thread = Thread()
    # 應該 continue loop 不 break
    ...

# Integration tests
def test_full_deploy_flow_with_real_db(db_container):
    """完整 deploy flow：trigger → LLM decisions → human approval → execute"""
    ...

# E2E tests
def test_slack_triggered_deploy_e2e(playwright):
    """從 Slack message 觸發到 deploy 完成的完整 journey"""
    ...

# Property-based tests
@given(thread_strategy)
def test_thread_serialization_round_trip(thread):
    """Property: thread serialize + deserialize = identity (Factor 5/12)"""
    serialized = json.dumps(thread.to_dict())
    restored = Thread.from_dict(json.loads(serialized))
    assert restored == thread

# Mutation testing
# 跑 mutmut，確認 critical control flow 的 mutation 都被殺死
```

**這套測試對應 12-factor 的每個 factor**：
- Factor 8 (control flow) → unit tests for switch cases
- Factor 5 (state) → property-based serialization round-trip
- Factor 7 (human) → integration test 確認 approval 路徑
- Factor 6 (pause/resume) → integration test 確認 save/load thread
- Factor 11 (trigger anywhere) → E2E test 從 Slack 觸發

---

## 1+1 > 2 的綜效

| 場景 | 只有 12-factor | 只有 90% | 兩者皆有 |
|---|---|---|---|
| Agent 走火入魔 | Factor 10 防範 + Factor 9 自癒 | 測試抓不到 runtime 問題 | ★ 結構限制 + 運行時自癒 + 測試保底 |
| Bug 進 production | 架構好但仍可能漏 | 測試抓到 | ★ 雙重防線 |
| Human-in-loop 失效 | Factor 7 結構保證 | 沒這概念 | ★ 結構 + integration test 驗證 |
| Refactor 風險 | Factor 12 純函數性幫助 | 測試保底 | ★ 結構 + 測試 |
| Cross-team handoff | 架構文件化 | Test 是文件 | ★ 雙重文件 |

---

## 給工程 leadership 的綜合建議

1. **不要選邊站**：兩條一起做。如果預算只夠做一條，**先做 90%（短期 ROI 更高），同時規劃 12-factor 的 phase-in**。

2. **Headcount 配置**：
   - 50% 開發團隊（寫 feature with AI）
   - 30% 平台 / quality team（負責 12-factor architecture + 90% gating）
   - 20% reliability / SRE（production verification）

3. **季度 OKR 配對**：
   - Q1: 12-factor adoption（agent code 重構）+ Coverage 65% → 75%
   - Q2: Factor 5/6/7 完成 + Coverage 75% → 85%
   - Q3: Factor 9/10 完成 + Coverage 85% → 90% + Mutation 75%
   - Q4: Audit + 升級到 Type D

4. **預算 ratio**：
   - AI tool 預算（包含寫測試 + 寫 feature）：$2-5K/工程師/年
   - Verification infra：$1-3K/工程師/年
   - 加起來 < 5% 工程薪資總和

---

## 終極論點

**Garry Tan 跟 Dex Horthy 是同一場戰役的兩個將軍**：

- Dex 守住「**LLM 應用怎麼建**」
- Garry 守住「**LLM 應用怎麼驗**」

兩條都贏，你的組織就贏。**任何只贏一條的組織，最終會被同時做到兩條的對手取代**。

> *"The companies that win the next 24 months won't be the ones with the fastest agents...They'll be the ones with the strongest guardrails."*
> — Garry Tan

把 12-factor 當「建造 guardrail 的圖紙」，90% 當「guardrail 的承重測試」。**兩個都做完，你才真正擁有 guardrail**。

---

## 結尾

恭喜你讀完整份 Garry Tan 複雜度棘輪 + 90% 測試覆蓋率精解。

### 一句話帶走

> **AI agent 寫程式比人類快 10 倍。如果驗證能力沒同步 ×10，你就被複雜度棘輪追上了。90% 測試覆蓋率（含 mutation ≥ 75%）是 AI 時代的新基線。**

### 該做的第一步

1. 跑 `coverage report`，看你現在數字
2. 看 [Ch 08 路線圖](08-implementation-roadmap.md) 的 Week 1
3. 設一個 12 週後的 90% 目標
4. 開始

---

[← 回 README](../README.md)
