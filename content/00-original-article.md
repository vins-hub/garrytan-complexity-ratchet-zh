[← 回 README](../README.md)

# 第 00 章：Garry Tan 原文 — The AI Agent Complexity Ratchet（完整對照）

> 來源：[@garrytan/status/2054064931515855118](https://x.com/garrytan/status/2054064931515855118) · 2026-05-12 1:03 PM · 233.8K views
> 形式：X Article（不是短推文，是長篇文章）
> 系列：AI Explainer Series 第 7 篇
> 抓取方式：gstack/browse headless browser 實際載入

---

## 文章標題

> **The AI Agent Complexity Ratchet: Why 90% Test Coverage Is Required**
>
> AI Agent 複雜度棘輪：為什麼 90% 測試覆蓋率是必要的

---

## 完整英文原文（依原文段落順序）

### 開場：作者的個人 credibility

> I've been coding with AI for the past year. Not just prompting — building real software. Two open-source projects: **GStack**, which makes AI coding agents better, and **GBrain**, which turns everything you read and write into a searchable knowledge base your AI can use. Between them, **about 970,000 lines of code and 665 test files**. Pretty much all written by Claude Code and Codex at my direction (15 simultaneous Conductor sessions most of the time).
>
> Last week I merged **fourteen pull requests in 72 hours. Nearly 29,000 lines of new code**. Each release was better tested than the one before it.
>
> That's supposed to be impossible. Speed and quality are supposed to trade off. Ship fast, break things. Move slow, ship right. Pick one.
>
> **You don't have to pick anymore. The unlock is 90% test coverage — and AI agents made it free to get there.** For fifty years, that level of verification cost too much human willpower to sustain. Now the agent writes the tests alongside the code. The result is what I call the complexity ratchet: **a system that can only get better, never worse**.

### 過去：軟體曾經 brittle

> For fifty years, the whole discipline of software engineering was organized around one idea: **prevent errors, because errors are catastrophic**.
>
> You had to get the code right the first time. Miss one edge case and you crash in production. Ship a bad database migration and you lose customer data. Write a function that does something subtle, and when the one person who understands it quits, nobody knows why it works. The whole system depended on humans being careful, and humans are not careful. So we built elaborate processes — code reviews, staging environments, QA teams, release trains — all designed to catch mistakes before they reached users.
>
> It kind of worked. But it was slow. And it meant that **the complexity of any software system had a hard ceiling: the number of things one team could hold in their heads simultaneously**.

### 現在：軟體 squishy

> I don't mean sloppy. I mean **resilient in a way that wasn't possible before**.
>
> When I say "the models are here," I mean that AI coding agents — Claude, GPT, Codex, and the ecosystem growing around them — can now read code, understand context, diagnose errors, and write fixes. Not perfectly. But well enough that **the error model for software has changed**.
>
> The migration breaks? The agent reads the error message, understands the database schema history across 45 versions, writes the fix, writes the test. The file sync hangs on a million symlinks? Agent diagnoses the parser timeout, bounds it at 30 seconds, ships the fix with tests. An extraction pipeline has an attribution bug? A cross-model evaluation catches it, the prompt gets iterated, enforcement gets added at the database layer.
>
> For most code-level errors — logic bugs, parsing failures, broken edge cases — agents can now diagnose and fix them in the next turn. That's genuinely new. **The errors that remain catastrophic are the ones that destroy state**: bad migrations on production data, security holes exploited before detection, privacy leaks that can't be un-leaked. The ratchet helps here too (good tests catch most of these before production) but the real shift is that the vast majority of errors in a codebase are the fixable kind.
>
> **This is a phase change for how software gets built. But it only works if you have the ratchet.**

### 核心概念：The Agent Complexity Ratchet

> A ratchet is a mechanism that **allows motion in one direction only**. A socket wrench turns a bolt forward and prevents it from turning back. That's the metaphor.
>
> In agent-coded software, every coding session with an AI agent adds **three things** to the codebase:
>
> 1. **Tests** that encode what "correct" means — automated checks that run every time someone changes the code, and fail loudly if the change breaks something
> 2. **Documentation** that records why decisions were made — not just what the code does, but the reasoning and tradeoffs behind it
> 3. **Evaluation results** that establish quality thresholds — structured assessments of output quality with scores, so you know if the next version is better or worse
>
> The next time an agent works on the codebase, it loads all three into its context window (the text the AI can see and reason about). **It can't regress below the test suite — the tests would fail. It can't ignore the documentation — it's right there in context. It can't ship quality below the evaluation baseline — the scores are recorded.**
>
> **The quality floor goes up with every turn. Forward-only motion. That's the ratchet.**

### 具體案例 1：Holder Confusion（GBrain）

> I'll make this concrete. GBrain is a knowledge system I'm building — it gives AI agents long-term memory by storing, indexing, and searching through a person's notes, meetings, conversations, and research. Think of it as a second brain that your AI assistant can actually read.
>
> One of its features is **epistemological extraction**: it reads through thousands of pages and extracts who believes what, with what confidence, over time. "Garry thinks Bitcoin will hit $300K (confidence: 0.45)." "Jared thinks this startup has strong retention (confidence: 0.80)." Like that, but across 28,000 pages.
>
> The first extraction run pulled **100,720 claims**. I used a cross-model evaluation to grade the quality — I had GPT-5.5 and Claude both independently score the output. Overall: **6.8 out of 10**.
>
> The biggest problem? Something I call **holder confusion**. Take the claim "AI will replace 80% of software engineers by 2027." Who holds that belief? Is it the person who wrote it? Is it someone they're quoting? Or is it the system's analysis engine, which inferred it from a podcast transcript? **Version 1 got this distinction wrong 35% of the time.** That matters — if you're building a system that tracks what people believe, you need to know WHO believes it.
>
> So the evaluation results got documented. Six specific failure modes got identified. The version 2 prompt addressed all six. **Weight rounding** (the confidence scores) got enforced at the database layer — no more false precision like 0.74 when 0.75 is the honest answer. **Seventeen tests locked in the contract.**
>
> Now no future version of the extraction can ship without those 17 tests passing. Nobody has to remember why weight rounding matters or what holder confusion is. **The tests remember.**
>
> The quality floor went up permanently. That's one turn of the ratchet.

### 反例：Vibecoded 專案的死法

> **"Vibecoding" is Andrej Karpathy's term** for coding with AI by describing what you want in natural language and letting the model generate the code. It's powerful and it's how I build. But from what I've seen across YC applications and open-source repos, **most vibecoded projects that skip tests start falling apart once they reach moderate complexity** — a few thousand lines, a handful of interacting features.
>
> They skip the ratchet. No tests, no docs, no evals. The agent adds complexity but nothing prevents regression. Every new feature has a chance of breaking an old one, and without tests, you don't find out until a user reports it. By version 0.5 the codebase is a haunted house where every change breaks something unexpected. Then the developer writes a blog post about how AI coding doesn't work.
>
> **AI coding works fine. They just didn't build the ratchet.**

### Tests as Institutional Memory（金句段）

> In traditional software companies, institutional memory lives in humans. The senior engineer who knows why that caching layer exists. The architect who remembers the migration that almost destroyed the database. The tech lead who can explain the weird edge case in the billing system.
>
> Humans leave. They retire, they get poached, they burn out. When they leave, the knowledge goes with them. Every software company has had the experience of opening a critical file and finding a comment that says `// DO NOT CHANGE THIS — ask Dave` and Dave left three years ago.
>
> **The agent's context window doesn't quit. It doesn't get poached. It doesn't forget.** When the test suite encodes "weight rounding must use 0.05 increments" and the documentation explains "because cross-modal eval showed false precision degrades trust in the confidence scores," **that knowledge is durable**.
>
> **Tests are institutional memory that survives employee turnover.** For a one-person project, they're even more critical — they're the only institutional memory you have.

### Everything Harnessable Is Testable（測試擴展論）

> The ratchet doesn't just work for traditional code. **It works for anything a computer can observe.**
>
> Think about the layers of a modern system:
>
> - The **OS** gives you process trees, file system state, network sockets, cron schedules
> - The **terminal** gives you every keystroke, every line of output, every interactive prompt
> - The **browser** gives you rendered pages, button states, navigation events
> - **APIs** give you structured responses you can parse and validate
> - And **AI agents** give you observable behavior — what they say, what tools they call, what order they do things in, whether they ask before acting
>
> **All of these are harnessable. And if you can harness it, you can observe it. If you can observe it, you can assert on it. If you can assert on it, you can ratchet it.**

### 具體案例 2：TTY Test Harness（GStack）

> GStack is my open-source coding agent framework — **93,000 GitHub stars, 701,000 lines of code, 46 skills**. One of its core features is interactive plan review: you ask it to review your architecture, and it walks through the plan section by section, asking questions, probing edge cases, challenging your assumptions. Like having an engineering manager who actually reads the code.
>
> The problem: Claude Code would sometimes skip the whole interactive part. It would read the plan file, dump every finding in one shot, and exit — without asking the user a single question. The entire point of the review is the back-and-forth dialogue. **Skipping it defeats the purpose.**
>
> **How do you even test that? You can't unit test "did the AI have a conversation."** No traditional testing framework covers this.
>
> So I used Bun's TTY functionality to build a test harness (**PR #1354**) that literally **spawns Claude Code inside a pseudo-terminal**, feeds it a specific repo scenario, triggers the review skill, and **watches the terminal output in real time**. The test observes whether the agent fires an interactive question before finishing. If it dumps findings and exits without asking anything, the test fails.
>
> **That's not testing code. That's testing whether an AI agent follows a behavioral contract. At the TTY level. By literally watching it work.**
>
> The ratchet response was three layers:
>
> 1. **STOP gates** in the skill instructions — explicit rules: "you MUST ask the user before proceeding to the next section," with anti-rationalization clauses that name the specific failure mode
> 2. **Anti-shortcut clause** — "the plan file is the OUTPUT of the interactive review, not a substitute for it." One sentence that closes the exact loophole the model kept exploiting.
> 3. **Gate-tier floor tests** — the TTY harness tests that spawn Claude Code in controlled scenarios and fail if the agent doesn't ask at least one interactive question

### 具體案例 3：OpenClaw Plugin Test（PR #880）

> Or take **PR #880**, which shipped a new OpenClaw plugin. The test doesn't just check that the code compiles. **It builds the plugin from source, spawns a real OpenClaw instance in an isolated profile, installs the plugin via the CLI, runs `plugins inspect` to verify the runtime loaded it, sets the config slot, validates the config, and runs `plugins doctor` to confirm zero diagnostics.** A full end-to-end round trip across two separate programs. **359 lines of test code.** The kind of test a human would almost never write by hand because the setup is too tedious. **Claude wrote it in about five minutes. That's the effort wall disappearing in real time.**

### 90% 的科學依據

> So what does 90% test coverage actually buy you?
>
> **Capers Jones** studied over 10,000 software projects and measured **defect removal efficiency (DRE)** — the percentage of bugs caught before they reach users. His data from *Applied Software Measurement* shows a nonlinear curve:
>
> - **Below 70% coverage**, DRE sits around 65-75%
> - **At 85-95% coverage**, DRE jumps to **92-97%**
>
> The relationship isn't linear. **There's a knee in the curve around 85%** where defect escapes drop sharply.
>
> The **avionics industry** figured this out decades ago. **DO-178C**, the FAA standard for flight-critical software, requires **modified condition/decision coverage (MC/DC)** for Level A systems — the ones where a bug means a plane crash. Branch coverage alone misses 10-20% of faults. MC/DC, which is stricter than line coverage, **achieves >99% DRE**. They don't mandate this because bureaucrats like paperwork. They mandate it because the data showed that below certain coverage thresholds, critical defects escape at rates incompatible with not killing people.
>
> The **reliability engineering parallel** is clean. Factories use a system called **Six Sigma**. The idea: count how many defects you get per million units produced, then express that as a "sigma level":
>
> - **3-sigma** process: about 67,000 defects per million (pretty bad)
> - **4-sigma**: about 6,200 (ten times better)
> - **5-sigma**: 233 (another 27x better)
>
> **The jump from 4 to 5 sigma is not incremental improvement. It's a phase change.**
>
> Test coverage follows the same curve. **Going from 70% to 90% coverage isn't 30% better. It's an order of magnitude fewer escapes.**

### 誠實面對反論

> Now, I should be honest about what the research also shows. **Mockus, Nagappan, and Dinh-Trong** studied Windows Vista and found that while coverage correlates with fewer post-release defects, the effort to reach 90%+ rises sharply. **The last 20% of coverage takes disproportionately more work than the first 70%.** This has been true for decades. It's why most teams stop at 70-80% and call it good enough.

### 核心 unlock：AI doesn't experience effort

> But something changed: **AI coding agents don't experience effort.**
>
> They don't get bored writing the fourteenth edge-case test. They don't cut corners at 5pm on a Friday. They don't look at a gnarly integration test and think "I'll come back to this later." **The effort curve that stopped human teams at 70% doesn't apply to agents.**
>
> You can ask Claude to write tests for every edge case in a module and it will do it cheerfully, thoroughly, at 2am, without complaining. **The brutal last 20% that made 90% coverage impractical for human teams is exactly the kind of work AI agents are best at.**
>
> **This is the real unlock. It's not that AI lets you write code faster. Plenty of people have noticed that. It's that AI lets you verify at a level that was previously too expensive to sustain. The 90% threshold that the data says is magical? It used to cost too much human willpower to reach. Now it's free.**

### 收尾金句段

> **Getting to 90% used to be a heroic effort. Now it's a Tuesday. That's the game change.**
>
> ...
>
> **The new complexity ceiling**: It used to be bounded by one team's ability to hold the system in their heads. Now it's bounded by one person plus agents who can load the full codebase, schema history, test suite, and documentation into context.
>
> That's a much bigger number. And it keeps growing as context windows get larger and models get better at reasoning about code.
>
> **Every software company that doesn't adopt this model — agents plus taste plus a test suite that only goes up — is already shipping slower and with less quality than one person who has.**
>
> The tools are here. The code is open. **Tests are the ratchet. 90% coverage, every PR, no exceptions.**
>
> **For fifty years, 90% coverage was a luxury reserved for avionics and medical devices** — teams with the budget to throw human hours at the effort wall. AI agents demolished that wall. The coverage threshold that makes software reliable is no longer expensive. It's just a setting. **The question isn't whether you can afford 90%. It's whether you can afford not to.**

### Garry 的開源專案

- **GStack** — AI coding agent framework — **93K stars** — MIT
- **GBrain** — second brain for AI agents — **14K stars** — MIT

---

## 繁中精簡翻譯（核心段落）

### 標題
**「AI Agent 複雜度棘輪：為什麼 90% 測試覆蓋率是必要的」**

### 核心命題
- 過去 50 年，軟體工程的組織原則是「**防錯，因為錯誤是災難**」
- AI agent 時代，軟體變得「**squishy**」（有彈性）——大多數錯誤 agent 下一個 turn 就能修
- 但前提是要有 **ratchet**

### 棘輪定義
每個 agent coding session 加 **3 樣**到 codebase：
1. **Tests**（什麼是「正確」的編碼）
2. **Documentation**（為何這樣決策的理由 + tradeoff）
3. **Evaluation results**（品質基線分數）

**下一輪 agent 工作時，這 3 樣都進 context window** → 不能 regress below test, 不能 ignore docs, 不能 ship 低於 eval baseline。**Quality floor 只升不降。Forward-only motion**。

### 90% 的科學依據

| 來源 | 數據 |
|---|---|
| Capers Jones（10,000+ 專案）| < 70% → DRE 65-75%；85-95% → DRE 92-97%。**Knee 在 85%** |
| DO-178C / FAA | Level A 要 MC/DC，達 >99% DRE |
| Six Sigma | 3σ→4σ→5σ 是相位變化（67K→6.2K→233 defects/M）|
| Mockus Vista | 最後 20% coverage 努力非線性陡升 |

### 真正的 unlock

**AI 不感到 effort**。
工程師 14 個邊角 test 寫一寫就無聊；agent 不會。
過去 90% 達不到不是技術問題，**是人類意志力 cost**。
AI 把這個 wall 拆了。
**Getting to 90% used to be a heroic effort. Now it's a Tuesday.**

### Garry 的口號

> Tests are the ratchet. **90% coverage, every PR, no exceptions.**
> 
> The question isn't whether you can afford 90%. **It's whether you can afford not to.**

---

## 修正聲明

本 repo 第一版（commit 13ac436）有以下虛構或錯誤：

1. **「執行長必問三大組織問題」**（Ch 04）——**原文無此結構**，我自己編的。已在 [SELF-VALIDATION.md](../SELF-VALIDATION.md) 詳細說明。
2. **「24 個月誰贏」名言**——Ali Ustun 的 LinkedIn paraphrase，**不是 Garry 原話**。
3. **「複雜度棘輪」定義**——我寫成「**部署鎖住、技術債只增**」，**原意是「Quality floor 只升不降」**，方向相反。

如果你看到 repo 任何地方引用 Garry 時用引號 `"..."`，**請對照本章原文**。本章英文版才是 source of truth。

---

## 接下來

➡️ [Chapter 01: 複雜度棘輪](01-the-complexity-ratchet.md)（精確定義版，已根據原文修正）
