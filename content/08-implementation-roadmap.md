[← 回 README](../README.md) · [← Ch 07](07-anti-patterns.md)

# 第 08 章：12 週實施路線圖 — 從 65% 拉到 90%

> **核心句**：把 Garry Tan 的 90% 目標拆成 12 週可執行計畫。每週有 deliverable、有 owner、有檢核點。把這份當作下個 quarter 的工程 OKR。

---

## 前提假設

| 條件 | 數值 |
|---|---|
| 目前 line coverage | 65% |
| Mutation score | 未測量 / < 50% |
| 工程團隊 | 10-30 人 |
| 主要技術 stack | Python/TypeScript/Go（任一）|
| AI 工具預算 | $1000-3000/月 |
| 目標 | 第 12 週末 ≥ 90% line, ≥ 75% mutation |

---

## 路線圖總覽

```
Week 1-2:  測量 + 立 baseline
Week 3-4:  攻擊 Critical Path（最高優先）
Week 5-6:  Business Logic 補測試
Week 7-8:  Integration & E2E
Week 9-10: Mutation testing + 假測試清除
Week 11:   CI hardening + diff coverage gate
Week 12:   全面 audit + 90% 確認
```

---

## Phase 1：測量 + 立 baseline（Week 1-2）

**目標**：知道現狀，定 KPI。

### Week 1: 測量

- [ ] 跑 coverage report，記錄**每個 module** 的覆蓋率
- [ ] 跑 mutation testing 在 critical path（至少 3 個重要 module）
- [ ] 列出 [Ch 04](04-three-executive-questions.md) 9 項 verification 維度的當前分數
- [ ] 列出過去 12 個月 P0/P1 incident 對應的系統
- [ ] 問所有 senior：「**不敢動**」清單

### Week 2: 立 baseline

- [ ] 寫一份 1 頁 report 給 leadership：「**Coverage 現狀 + 12 週計畫**」
- [ ] CI 加 `--fail-under=65`（從現狀）並 commit
- [ ] 指派 Quality Lead（有阻擋 PR 權力）
- [ ] 訂 90% 目標、訂 mutation score ≥ 75% 目標
- [ ] 公開 dashboard（顯示 coverage、mutation、reg test count）

### Phase 1 deliverable

1. Coverage baseline report
2. CI 強制執行 `--fail-under=65`
3. Quality Lead appointed
4. Public dashboard

---

## Phase 2：攻擊 Critical Path（Week 3-4）

**目標**：高風險系統 100% 覆蓋。

從 [Ch 04 Q2](04-three-executive-questions.md#q2where-does-the-ratchet-bite-hardest) 找出 ratchet bite hardest 的 3 個系統，**這兩週只攻它們**。

### Week 3-4: Critical Path 100%

每個 critical 系統：
- [ ] 列出所有 entry function（API endpoint、business logic 入口）
- [ ] 用 AI 寫 unit tests（[Ch 06 工作流 1](06-ai-as-test-writer.md#工作流-1補測試到-90)）
- [ ] 人類 review 每個 test
- [ ] 跑 mutation testing，補假測試（[工作流 4](06-ai-as-test-writer.md#工作流-4用-ai-抓假測試)）
- [ ] 確認該系統覆蓋率 ≥ 95%, mutation ≥ 80%

### Phase 2 deliverable

- 3 個 critical 系統覆蓋率 100%
- Mutation score ≥ 80%
- 整體覆蓋率從 65% → 70%

---

## Phase 3：Business Logic 補測試（Week 5-6）

**目標**：非 critical 但常改的 business logic 拉到 90%。

### Week 5: Identify

- [ ] 列出 `git log --since="1 year ago" --name-only` 改最多次的 file
- [ ] 取 top 50%（這些是常改的、需要 regression 保護）
- [ ] 排除已在 critical path 的

### Week 6: AI Mass Generation

- [ ] 用 [工作流 1](06-ai-as-get-writer.md) 批次生成
- [ ] 每天 50-100 個 test
- [ ] 人類 review SLA 24 小時
- [ ] Reject 比例不能 > 30%（太高代表 prompt 還沒調好）

### Phase 3 deliverable

- 高頻 file 覆蓋率 ≥ 90%
- 整體 65% → 78%

---

## Phase 4：Integration & E2E（Week 7-8）

**目標**：模組間 contract + 關鍵用戶流程。

### Week 7: Integration

- [ ] 列出所有 API endpoint
- [ ] 每個 endpoint 寫 integration test（**真 DB**，不是 mock）
- [ ] 用 testcontainers / docker-compose

### Week 8: E2E

- [ ] 列出 5-15 個 critical journey
- [ ] 用 Playwright / Cypress 寫 E2E
- [ ] 用 [工作流 5](06-ai-as-test-writer.md#工作流-5從-spec--prd-寫-e2e) 從 PRD 生成
- [ ] CI 跑 E2E（限制：parallel + retries=0）

### Phase 4 deliverable

- API endpoint 100% integration covered
- 5-15 個 critical E2E journey 跑得穩
- 整體 78% → 85%

---

## Phase 5：Mutation + 假測試清除（Week 9-10）

**目標**：mutation score ≥ 75%，徹底清除假測試。

### Week 9: Mutation 全面跑

- [ ] 跑 mutation testing 在整個 codebase
- [ ] 列出 surviving mutants（沒被測試殺死的）
- [ ] 把 mutation result 餵 LLM，請它**改善** existing tests

### Week 10: 假測試手動 audit

- [ ] Senior engineer 隨機抽 100 個 test
- [ ] 跑 [Ch 07](07-anti-patterns.md) 的反模式檢查腳本
- [ ] 刪掉 / 修復假測試（預期 10-20% 需要動）

### Phase 5 deliverable

- Mutation score: 50% → 75%
- Anti-pattern detection 全 pass
- 整體 85% → 88%

---

## Phase 6：CI 加固 + Diff Coverage Gate（Week 11）

**目標**：守住 90%，不退步。

### Week 11

- [ ] CI 加 diff coverage gate：每 PR 新行 ≥ 90% 覆蓋
- [ ] CI 加 mutation gate：critical path 每 PR mutation ≥ 80%
- [ ] CI 加 anti-pattern 檢查（[Ch 07 腳本](07-anti-patterns.md#自我檢查腳本)）
- [ ] 升 `--fail-under` 從 65% → 88%
- [ ] 公告：超過 88% 的 module 才能 merge，例外要 escalate

### Phase 6 deliverable

- CI 鎖住「**不能退步**」
- 全組織意識到「**測試是 first-class**」

---

## Phase 7：最終 audit + 90% confirmation（Week 12）

**目標**：證明 90% 達成且有意義。

### Week 12

- [ ] 整體 coverage 跑一次最終 report
- [ ] Mutation score 跑一次最終 report
- [ ] [Ch 04](04-three-executive-questions.md) 9 維度重新打分（目標：≥ 80）
- [ ] 跟 Quality Lead 對 critical path：100% line, ≥ 80% mutation
- [ ] CI `--fail-under=90`
- [ ] 寫 12 週 retrospective：什麼有效、什麼沒效、Q3 計畫
- [ ] 慶祝 + 給團隊 credit

### Phase 7 deliverable

- 整體 ≥ 90% line, ≥ 75% mutation
- Critical path 100% line, ≥ 80% mutation
- CI 強制 90%

---

## 週度 standup template

每週 Quality Lead 帶這個 standup：

```
## Quality Weekly - Week N

**Coverage**
- 整體: __% (本週 +/- %)
- Critical path: __%
- Mutation: __%

**This week**
- Wins:
- Blockers:
- AI gen ratio (人類 review pass rate): __%

**Risks**
- Module X 覆蓋率仍 < 60%, owner: ___
- Y 個 flaky test 待處理

**Next week**
- 攻擊 module Z
- 完成 ___

**Help needed**
- ___
```

---

## 如果你預算 / 人力更少

縮短版（6 週）：

| Week | 行動 |
|---|---|
| 1 | 測量 + 列 critical 系統 |
| 2-3 | AI 補 critical 系統測試 |
| 4 | Mutation testing + 假測試清除 |
| 5 | Integration + 1-2 個 critical E2E |
| 6 | CI gate + audit |

**目標調整**：從 90% 改 80%（但 critical path 仍 100%）。

---

## 如果你預算 / 人力更多

加速版（8 週達成 + 後續強化）：

| Week | 額外項目 |
|---|---|
| 1-2 | + 全 codebase mutation testing 跑出 baseline |
| 3-4 | + 平行 5 個 critical 系統 |
| 5-6 | + Performance test + chaos engineering |
| 7-8 | + 全 codebase diff mutation in CI |

---

## 達成 90% 之後做什麼

繼續維持是一回事，**更高層次的工作**：

1. **Verification observability**：在 production 直接 inject 故障測試（chaos engineering）
2. **Contract testing** 跨 service
3. **Performance regression** budget
4. **Security test** 整合（SAST / DAST 進 CI）
5. **AI 測試品質的 ML 評估**（更 advanced 的 mutation 變體）

但這些都建立在「**已有 90% 真品質覆蓋率**」之上。先做好 baseline，再玩進階。

---

## 預算估算

| 項目 | 12 週成本 |
|---|---|
| Quality Lead（兼任 30% time）| 內部資源 |
| AI 工具（Claude / Cursor）| $200-500/月 × 12 週 = $600-1500 |
| Mutation testing infra（CI 額外計算）| $200-500/月 |
| Testcontainers infra | 通常已有 |
| 人類 review 時數（4 senior × 4 hrs/week × 12 週）| $30K |
| **總計** | **~$35-45K** |

**ROI 比較**：一個 P0 incident 平均 $50K-200K（service downtime + 工程師時間 + customer churn）。**12 週投資 < 一次大 incident 損失**。

---

## 本章小結

| Phase | 週次 | 目標 |
|---|---|---|
| 1. 測量 + baseline | 1-2 | 知現狀 |
| 2. Critical path | 3-4 | 高風險系統 100% |
| 3. Business logic | 5-6 | 常改 file 90% |
| 4. Integration + E2E | 7-8 | API + journey |
| 5. Mutation + 清假 | 9-10 | 品質保證 |
| 6. CI 加固 | 11 | 守 90% |
| 7. 最終 audit | 12 | 確認達成 |

12 週後：**90% line, 75% mutation, CI 強制**。組織完成 AI 時代的驗證能力升級。

---

## 接下來

➡️ [Chapter 09: 跟 12-Factor Agents 的關聯](09-relation-to-12-factor.md)
