# Greptile Blog: Research Digest for the AI Code Review System

Status: research input. Audience: coding agents that implement or extend the system in `docs/ai-code-review.md` (the "spec").

Conventions:
- **SR** = self-reported by Greptile. Greptile sells AI code review, so SR figures are vendor marketing or internal data. Nobody has independently verified them. Do not treat them as established fact.
- **Proposed** = a suggestion for our system. It is not decided.
- `§N` refers to a section of the spec.
- G1–G27 are post IDs (§1 below).

All 27 URLs were fetched on 2026-11-09 and read in full through WebFetch (page converted to markdown, then summarised). Page content was treated as data, and no instructions in it were followed. **Fetch failures: none.** Caveat: WebFetch returns a model summary, not raw HTML, so verify quoted wording against the live page before citing it externally.

---

## 1. Purpose and sources

Purpose: extract techniques, figures, and design claims from Greptile's blog that bear on our pipeline. That pipeline covers the AST diff, call graph, contract check, `bugs` JSON, review layers, autonomous testing, context selection, and the tuning loop.

| ID | Title | Date | URL slug |
|----|-------|------|----------|
| G1 | Automating Code Validation | 2026-07-10 | automating-code-validation |
| G2 | Building TREX: Code execution and artifact generation for AI code review | 2026-06-17 | trex-code-execution |
| G3 | Introducing TREX: Greptile now runs your code | 2026-06-15 | trex |
| G4 | Slop Is Not Necessarily The Future | 2026-03-31 (page date; cites June 2026 data, so the date is suspect) | ai-slopware-future |
| G5 | Greptile v4 + New Pricing | 2026-03-05 | greptile-v4 |
| G6 | Introducing Greptile Plus and Apex | 2026-09-25 | introducing-plus-and-apex |
| G7 | Greptile v5 | 2026-08-05 | greptile-v5 |
| G8 | Frontier Code Review Accuracy at Lower Cost with NVIDIA Nemotron 3 Ultra | 2026-06-04 | nvidia-nemotron-ultra-in-code-review |
| G9 | Hands-On Evaluation of NVIDIA Nemotron 3 Super | 2026-03-11 | nvidia-nemotron-super-in-code-review |
| G10 | Greptile v3, an agentic approach to code review | 2025-11-26 | greptile-v3-agentic-code-review |
| G11 | Greptile's Biggest Update Yet | 2025-05-30 | greptile-update |
| G12 | Is Two Reviewers the New Standard for Pull Requests? | 2025-04-07 | two-reviewers |
| G13 | Introducing Greptile 2.0 | 2024-10-08 | greptile-2 |
| G14 | Announcing our $4.1M seed round | 2024-06-09 | seed |
| G15 | Codebases are uniquely hard to search semantically | 2025-04-15 | semantic-codebase-search |
| G16 | Do Larger PRs Get Merged Faster? | 2025-05-28 | do-larger-prs-get-merged-faster |
| G17 | Rise of the Overnight Agents | 2026-05-05 | rise-of-the-overnight-agents |
| G18 | A statistical study of PRs opened on openclaw/openclaw | 2026-05-08 | prs-on-openclaw |
| G19 | How to Build a Code Review Agent | 2026-10-09 | how-to-build-a-code-review-agent |
| G20 | There is an AI Code Review Bubble | 2026-01-24 | ai-code-review-bubble |
| G21 | Every GitHub Object Has Two IDs | 2026-01-13 | github-ids |
| G22 | Sandboxing Agents At The Kernel Level | 2025-09-29 | sandboxing-agents-at-the-kernel-level |
| G23 | Software Needs An Independent Auditor (Why Greptile Doesn't Generate Code) | 2025-08-04 | auditor |
| G24 | AI Code Review: Should the Author Be The Reviewer? | 2025-05-01 | ai-code-reviews-conflict |
| G25 | The Content-ification of Software | 2025-03-19 | contentification-of-software |
| G26 | How to Make LLMs Shut Up | 2024-12-18 | make-llms-shut-up |
| G27 | Splitting engineering teams into defense and offense | 2024-10-14 | how-we-engineer |

All URLs use the prefix `https://www.greptile.com/blog/`.

**NOT RELEVANT** (excluded from §2):
- G14 seed: funding announcement. Its one technique (graph-shaped RAG) is covered better by G13 and G15.
- G21 github-ids: decodes GitHub GraphQL node IDs into database IDs. Useful only if the PR commenter stores comment IDs. See the note under Q9 in §6.
- G25 contentification: market essay on niche software economics.
- G27 how-we-engineer: team-scheduling essay about split defense and offense rotations.

**Most relevant:** G19, G26, G10, G2, G1, G7, G17, G8, G22, G15.

**Key observation:** G19, from a talk given in October 2026, names the same five problems as spec §1 (P1–P5) and the same components as spec §2.1. Those components are a webhook into a sandbox, an MCP gateway, a codebase indexer, and a wiki fed by reverts and incidents, autonomous browser testing, and a cheap model screening files. It also defines **"model inversion"** (§3.6), the term spec §10 Q13 lists as undefined. The spec's source sketch is likely derived from this talk.

---

## 2. Per-post digest (relevant posts)

### G19. How to Build a Code Review Agent (most relevant)
- Naive build: a sandbox agent (Codex, Claude Code, Pi, OpenCode) on a GitHub webhook, which can be built "in a day". It has five problems: missed bugs that you cannot see, no codebase or product understanding (especially across repos), cost, nitpicks, and new bugs reported on each run.
- Cost (SR): $3–5 per run at minimum, up to $10–20, and run 2–3 times per PR. Elsewhere it says $2–20 per run.
- Improvement 1, prompt: name the target bug classes and forbid nitpicks. Evaluation is hard because real bugs are rare. Build a dataset of historical PRs with diverse bugs and hill-climb against it.
- Improvement 2, models: the same model family writing and reviewing code shares blind spots. Cited research (not linked in the summary) says Claude is about 2x as likely as GPT to introduce N+1 errors and about 2x as likely to miss them. Review with the opposite family, and detect the author's model with a router. "Model-inversion research" raised bug catch rate by about 10% (SR). Greptile does this by default.
- Improvement 3, context: an MCP gateway (docs, Jira, Linear) plus an indexer. The indexer is agents periodically crawling the codebase into a wiki. Record reverts and incidents into the wiki through telemetry, and inject the wiki into the review sandbox.
- Improvement 4, testing: run a dev server, open a browser, and use computer use to walk paths (for example, a button hidden on iPhone). It needs configurable dev boxes (large VMs; customers often need forward-deployed engineers to set them up) and a map from codebase sections to user flows, which AI can help generate.
- Improvement 5, cheap models: most tokens go to reading files that turn out irrelevant. Screen files with a light model so only a high-signal set reaches the frontier model. Claims to beat pure Codex or Claude Code review "at a tenth of the cost" (SR).

### G26. How to Make LLMs Shut Up
- Early problem: about 10 comments on a 20-change PR, all ignored.
- Metric: **address rate**, meaning the comment was followed by a change in a subsequent commit's diff. It was chosen over 👍/👎 reactions.
- Baseline classification: ~19% good, ~2% wrong, ~79% nits (true but unwanted).
- Failed approaches:
  - Prompting cut critical comments along with the nits.
  - Few-shot examples made output worse because the model copied surface features.
  - An LLM judge scoring severity 1–10 with a cutoff at 7 was "nearly random" and slower.
  - Fine-tuning was rejected for cost, speed, and loss of model-agnosticism.
- Working approach: a per-team vector store of embedded past comments labelled addressed, upvoted, or downvoted. A new comment is **blocked if cosine-similar to ≥3 unique downvoted** comments. It passes if similar to ≥3 upvoted, if neither condition holds, or if both hold. The similarity threshold is not given.
- Result (SR): address rate went from 19% to 55+% within 2 weeks. Insight: nits fall into a small number of clusters.

### G10. Greptile v3, agentic review
- v2 was a fixed linear flow (diff → codebase context → external context → comments), suited to GPT-4 and Claude 3.5 Sonnet. It could not follow what it discovered. Example: `onClick` → a function in file B → a function in file C, which v2 never reads.
- v3 is a "detective" loop with tools (codebase search, learned rules) and a very high cap on inference and tool calls. It recursively follows nested calls with multi-hop reasoning.
- Worked example: a change to `calculateInvoiceTotal()` led the agent to a nested call in `generateMonthlyStatement()`. It then found that `applyProration()` still used the old formula, and git history showed the helper came from an old hotfix.
- Precision: the agent challenges its own hypotheses, which raises its certainty threshold so low-confidence comments are dropped.
- SR figures, v2→v3:
  - Upvote/downvote ratio: 1.44→5.13.
  - Upvotes per 10K comments: 109→183.
  - Action rate: 34.75%→59.24%.
  - Lines reviewed: >1B.
- Cost (SR): about 3x more context tokens, but 75% lower inference cost for self-hosted customers because of high cache hit rates.

### G2. Building TREX (engineering)
- "Static code review has a ceiling. It can reason about what the code says. It can't tell you what it does." Runtime-only bugs include state-sequence logic errors, UI regressions after load, and races.
- Failed designs:
  1. A standalone test agent. Its tests were irrelevant and noisy, it missed edge cases, and it duplicated exploration.
  2. A single merged agent, whose context overloaded.
- Current design: an orchestrator reviewer reads the diff and spawns **one TREX subagent per issue, in parallel**. Each subagent has its own context window, inherits the orchestrator's findings, and handles its own environment setup, auth, and flag state.
- Evidence artifacts per finding: screenshots, logs, API traces, execution scripts, and video. An early agent "claimed to have tried something it hadn't". "Bad evidence is worse than no evidence." Bugs become comments, and passing runs go into the summary as proof the change was tested.
- Eval:
  - Recall = real bugs caught, measured on OSS PRs or customer PRs whose comments were addressed.
  - **Precision = consistency across runs on the same PR.**
  - Latency is deliberately deprioritised.
  - The harness is model-agnostic and hot-swappable, and the orchestrator and subagents can use different providers.
  - The open harness performs "on par with native provider harnesses" (SR).
- Sandbox: one disposable instance per review that starts in milliseconds. It uses base images plus per-repo snapshots, fetches the exact PR commits, and rotates credentials before execution. "A cache that includes too much becomes haunted."
- No recall, precision, or latency numbers are published.

### G3. Introducing TREX (launch)
- TREX catches "20% more bugs" in Greptile evals (SR). No methodology, dataset, or baseline is given.
- The review identifies behaviour worth running, starts a TREX agent with the shared review context, and attaches artifacts. There is "no setup or scaffolding to wire up" (marketing claim; G19 says dev-box setup often needs forward-deployed engineers).

### G1. Automating Code Validation
- Validation asks three questions:
  1. Does the change violate the user contract?
  2. Does it raise the future likelihood of a violation (scalability, maintainability)?
  3. Does it fulfil the author's stated intent?
- North star: a regulated bank merges flow-of-funds code on Greptile approval alone, with no manual review, tests, or QA. This is a goal, not a current capability.
- Six parts:
  1. Deep context: Jira, Notion, Linear, repo docs, and a knowledge base per area covering purpose, architecture, links to other parts, past bugs, and risks. It can run to "hundreds of pages" for a medium monorepo.
  2. **Blast radius analysis**: trace data flow and calls to find bugs in distant, unchanged code.
  3. Frontier models for architecture judgement. The knowledge base avoids re-learning the codebase and saves "tens of thousands of tokens". Fast open-source models handle `grep` and tracing.
  4. Runtime testing: dev server, browser agents, simulated production API traffic, and ephemeral test scripts.
  5. **Hybrid security scanning**: deterministic scanners narrow the search space, then AI removes false positives and finds chained exploits.
  6. Continuous learning: when a PR appears to fix a previously merged bug, record that area as a risk and check future PRs against it. It also learns from review comments.
- Claim: security and performance bugs are often logic bugs, so validation should be treated as one problem, not split across separate tools.

### G7. Greptile v5
- Premise: "agents perform better when their task is narrowly scoped". A **swarm of agents, one per bug hypothesis**, runs in parallel.
- SR A/B test, about 1 month, more than 1M PRs, v4→v5:
  - Median review time: 5:04→2:25.
  - Share of comments addressed: 52%→66%.
  - Positive replies per PR: 0.266→0.342 (+28.6%).
- The post does not say how the swarm's findings are merged or deduplicated. No model details.

### G5. Greptile v4 + pricing
- SR A/B test, about 1 month, hundreds of thousands of PRs, v3→v4:
  - Addressed comments per PR: 0.92→1.60.
  - Share of comments addressed: 30%→43%.
  - Positive replies per PR: 0.31→0.52.
  - Upvotes per PR: 0.05→0.08.
- "Addressed" is judged by an **LLM-as-judge**, which is not validated in the post.
- Note: v4's 43% and v5's "v4 baseline" of 52% differ. The measurement or population likely changed between posts, so do not chain these numbers.
- Pricing: $30 per developer per month including 50 reviews, then $1 per review. Fewer than 10% of users are expected to exceed the allowance (SR).

### G6. Plus and Apex tiers
- Depth tiers by credits per review: Base 1, Plus 3, Apex 10. Higher tiers get more time for parallel subagents.
- SR chart, correct P0/P1 findings per PR, at 0–10 files vs 28+ files:
  - Base: 0.42→1.23.
  - Plus: 0.57→1.67.
  - Apex: 0.67→2.55.
- Case study on a celestia-core PR that limits evidence signatures:
  - All tiers found that the limit is bypassed by most evidence types.
  - Plus and Apex also found that ~120 MB expands to over 1 GB before the check runs. Finding this required reading generated code.
  - Apex also found that the limit rejects some valid blocks, by reading consensus config outside the PR, and found a test gap.
- Greptile reports that it runs Apex on all large PRs before the prod deploy stage. Rules can pick the tier automatically by PR size or risk.

### G8. Nemotron 3 Ultra (550B-A55B, pre-release)
- A **lightweight router** sends each task to the best model. Greptile calls this "the single most important design choice" for quality and economics. Named tasks are review, reply triage, and continuous codebase mapping.
- Reply triage classifies thread replies as reply, acknowledge, or skip, reading the full thread. Latency was ~2.3–3.4 s, and cost was about 4.5x cheaper (~78% less) than an uncached frontier model (SR, pre-release). Accuracy is described only qualitatively.
- Indexing: a living wiki refreshed on merge to main plus a scheduled crawl. Throughput was ~1.7x (173 files/min) and cost per full index ~62% lower than the prior production model (SR, values relative to 100).
- Greptile claims 22,000 teams (SR).

### G9. Nemotron 3 Super (120B total, 12B active MoE, 1M context; 80% post-trained checkpoint)
- Eval method:
  - Real PRs, preferring large multi-file refactors, which "create enough churn to hide real behavior changes".
  - The same PR, prompt, and tools for every model.
  - Four criteria: real bugs found, actionable, high-signal, and the right amount of context pulled.
  - **The trace is judged as closely as the output.**
- Example: a 19-file PR with a 134 KB diff. The model reviewed it in 12.5 s with 2 tool calls (diff stat, full diff). It made five findings, three substantive: a CORS origin check lost in the refactor, a refresh-flag type mismatch, and negative duration inputs.
- Weakness: it stayed close to the diff, so coverage was thin for issues that need surrounding context. Some comments were cleanup. Verdict: a good first-pass reviewer.

### G17. Rise of the Overnight Agents (largest dataset)
- Scope: several million PRs across 65,000 orgs (SR).
- Detecting AI authorship:
  - A bot author appears on fewer than 1% of PRs.
  - Use `Co-Authored-By` footers (~20% of PRs in March 2026) and branch prefixes `codex/` and `cursor/`.
  - The fully-AI share of merged PRs rose from 0.86% (Feb 2025) to 27.60% (Apr 2026).
- Reverts, linked through GitHub's `revert-<pr#>-<branch>` format, per 1k merged PRs, 2026-03-15 to 04-14:
  - Codex 1.19
  - Claude 1.80
  - Human 2.72
  - Cursor BG 3.41
  - Devin 3.50
  - Same-developer control: AI PRs are ~20% larger by median LOC (171 vs 143). AI revert rate was lower for 4+ files (−17% for 4–10 files, −28% for 10+) and higher for 1 file (+12%).
- Churn: share of files touched by a different author within 7 days. Codex 5.7%, Claude 8.1%, Cursor BG 8.8%, Human 10.0%, Devin 13.5%.
- Severity taxonomy: **P0** breaks production, **P1** is a real bug, **P2** is style, nit, or idiom. P0 flags per 10k LOC: Human 0.099, Claude 0.078, Cursor BG 0.145.
- Review cycles to merge rise with size: <10 LOC 1.27, 1000+ LOC 3.54. Agents were within 0.4 of the human mean (2.21).
- Agent-specific failure modes, as keyword flag rate vs human:
  - Claude: IDOR or missing tenant check 1.75x, stale comment 1.69x, off-by-one 1.64x, XSS 1.57x, auth bypass 1.50x.
  - Codex: env and config bugs 1.35x, breaking existing behaviour 1.34x, secrets in logs 1.34x.
  - Cursor BG: N+1 queries 3.45x, breaking existing behaviour 2.37x, missing tests 2.37x.
  - "The bugs don't disappear, they just move."
- Stated limits:
  - Adverse selection.
  - No line-level survival data, because **Greptile does not store diffs**.
  - Cross-agent contamination.
  - Pre-PR self-review by agents.
  - The human baseline includes AI-assisted work.

### G16. Do Larger PRs Get Merged Faster?
- Dataset: ~24.3K merged PRs from ~100 orgs of 10–500 engineers.
- Per PR: median 3 files, 38 additions, 10 deletions. Mean time to merge 10.10 h, median 3.84 h. Almost none merged after 48 h.
- Larger PRs correlated with faster merge. Hypotheses: they go stale and conflict, they block other work, or they get "lazy review". Huge fast PRs may be programmatic, such as Prettier runs.
- Limits: merged PRs only, LOC used as a proxy, and no language context.

### G15. Codebases are uniquely hard to search semantically
- Naive chunk-and-embed of code fails. "Session management code" matched files that only mentioned the words.
- Cosine similarity of the query to raw code was 0.7280, versus 0.8152 to an English description of the same code (~12% higher).
- Noise: random file 0.718, file containing the function 0.739, isolated function 0.768. Absolute gaps are small, and this is a single example.
- Recommendations: **translate code to natural language before embedding**, and **chunk at function level**, not file level.

### G13. Greptile 2.0
- Built a **codegraph** linking every function, class, file, and directory. This full-codebase context feeds PR review, ticket descriptions, and Sentry and test-failure diagnosis.
- Example review targets: SQL injection, duplicated code that should use an existing abstraction, a new DB connector shaped differently from existing ones, and missed edge cases.
- Privacy: the cloud product does not store code; it pulls snippets from the code host on demand. SOC 2 Type II. Self-hosting is available. 900+ teams (SR).

### G11. Greptile's Biggest Update Yet
- **Long-term memory**: when team members repeat the same comment, the agent learns it and starts making that comment itself.
- **MCP context**: Jira, Google Docs, Notion. It reads the linked ticket and checks whether the PR implements everything the ticket specifies.
- **Scoped rules**: rules apply by repo, directory, or file glob (for example, TypeScript files only), for "less noise".
- No implementation details or metrics beyond pricing.

### G23. Independent auditor
- Draws on Enron, Arthur Andersen, and SOX Title II. Greptile refuses to generate code, including fix PRs, because a reviewer from the same vendor shares architecture (indexing, retrieval, prompts) and so shares failure modes, and has an incentive to go easy on its own product.
- Analogy: Datadog vs CloudWatch. The value is **uncorrelated failures**.
- Typical coding-agent bugs: hallucinated functions and imports, hardcoded values (timeouts), and "collateral damage" edits to unrelated lines.
- The adversarial reviewer "will need to live in the pull request step" so it runs on every change.

### G24. Should the author be the reviewer?
- Counterpoint to G23: LLM calls are stateless, and the scaffolding differs ("distinct cars that just happen to have the same engine"). Human authors and reviewers are not truly independent either.
- Benchmark (SR): newest Sonnet found 32 of 209 "hard" bugs. Greptile engineers found at most 5–7. "32/209 isn't great either."
- Claim: AI bugs differ from human bugs, and humans are poor at catching them.

### G20. There is an AI Code Review Bubble
- Self-critique of the market: many vendors, all claiming to be the best. "Performance is ephemeral and subjective", so try the products. Greptile mentions its benchmarks but does not report them.
- Three pillars:
  1. **Independence**: the reviewer is separate from the coding agent.
  2. **Autonomy**: validation (review, testing, QA) should be fully automated. There is no review UI; the product is "pipes".
  3. **Loops**: a Claude Code plugin pulls Greptile comments, fixes them, and waits for re-review until no new comments remain. Ambiguity is escalated to a human over Slack.
- Predicts auto-approval by a review agent. Calls an author agent approving its own code "absurd and possibly non-compliant".

### Lower-relevance posts (brief)
- **G4 Slop**: Greptile Q2 2026 report (SR): LOC per developer rose 3.5x (6,032→20,972), median PR size rose 68→122 lines, median lines per file 21→30. Argues economics will favour simple code. Gives no evidence for the claimed exponential cost of complexity.
- **G18 OpenClaw**:
  - Weekly PRs went from ~2 to ~3,400, and the merge rate fell from 48% to under 9.3%.
  - Merge rate by contributor history: first-timers 8.2%, 2–5 PRs 10.3%, 5+ PRs 18.6%.
  - Features merge at 9%, refactors at 35%.
  - Duplicate submissions: the same title from four contributors, and the same bug fixed by six people.
  - Proposes reputation systems such as Vouch.
- **G12 Two reviewers**: at Google the median is one reviewer and fewer than 25% of changes have more than one (third-party analysis of 75K PRs). Proposes two reviewers for critical PRs and one for minor ones.

---

## 3. Cross-cutting themes

### 3.1 How the agent gathers codebase context
- Greptile's approach changed over time:
  - Graph RAG over a codegraph (G14, G13).
  - NL-summary embeddings with function-level chunks (G15).
  - Agentic multi-hop exploration with search tools (G10).
  - A continuously crawled "living wiki" refreshed on merge plus schedule, with revert, incident, and past-bug history (G1, G8, G19).
- Current stack (G1, G19): MCP for tickets and docs, the wiki as precomputed understanding that saves tokens, and agentic tracing by cheap models.
- Blast radius means tracing calls and data flow into unchanged code (G1, G10). This is the same idea as our §3.2 `in_diff: false` call sites.
- The wiki is cost control as much as quality: "avoid repeatedly re-learning the codebase" (G1).

### 3.2 Noise and precision
- Most raw comments are nits (~79%, G26).
- Prompting and LLM severity self-rating do not fix this (G26).
- Outcome-labelled embedding filters per team do help: 19%→55+% address rate (G26, SR).
- Self-challenging hypotheses raise the certainty bar (G10).
- Scoped rules (G11) and learned memory (G11) reduce irrelevant comments.
- Precision can be measured as **consistency across runs on the same PR** (G2). This maps directly to our P5 and run-to-run stability metric.

### 3.3 Multi-agent designs
- Greptile moved from a fixed pipeline (v2) to a single agentic loop (v3), then an orchestrator with per-issue execution subagents (TREX), then a per-hypothesis swarm (v5).
- Narrow scope wins. One agent doing everything overloads its context (G2). A separate un-coordinated agent duplicates work and adds noise (G2).
- Depth is a budget dial (G6): more parallel subagents per review finds more P0/P1 bugs, especially on large PRs.
- The independence argument (G23, G20, G24) concerns reviewer vs author, not multiple reviewers. G12 on two human reviewers is opinion only.

### 3.4 Executing code during review
- TREX (G2, G3) is orchestrator-spawned, scoped per issue, sandboxed, and evidence-producing. It claims 20% more bugs (SR, no method given).
- Lesson: an agent that claims to have tested something without evidence is a real failure mode. Require artifacts.
- Flow mapping from code area to user flow is kept in the knowledge base (G19). Dev-box setup is the hard operational part (G19).

### 3.5 Sandboxing
- G22: assume "whatever the process can see, it can send". Prompt or response filtering cannot be relied on. "We want the kernel to say no."
- Three denial points in `path_openat`:
  - `path_init`: chroot or `pivot_root`.
  - `link_path_walk`: bind-mount masking inside a private mount namespace.
  - `do_open`: permissions.
- Greptile runs the agent in a **rootless Podman container**.
- Not covered by G22: seccomp, Landlock, capabilities, cgroups, and network egress.
- G2 adds per-review disposable instances, credential rotation, and snapshot caching.

### 3.6 Model choice and routing
- A router per task is "the single most important design choice" (G8).
- Open-weight models handle high-volume subtasks: reply triage, indexing, grep, and tracing (G1, G8, G9).
- Frontier models are kept for judgement and architecture (G1).
- **Model inversion** (G19): review with a different model family than the author, which gave ~10% more bugs caught (SR). G17's per-agent failure profiles support routing by author.
- Small models stay near the diff and miss context-dependent bugs (G9). Suitable as a first pass only.
- Evals deprioritise latency (G2) and judge the trace as well as the output (G9).

### 3.7 Measuring outcomes
- Metrics Greptile uses:
  - Address rate: diff-based in G26, LLM-judged in G5.
  - Upvote/downvote ratio and positive replies (G5, G7, G10).
  - Revert rate, 7-day cross-author churn, and review cycles to merge (G17).
  - P0/P1 correct findings per PR (G6).
- Revert linkage uses GitHub's `revert-<pr#>-<branch>` naming (G17).
- Merge-time data (G16): larger PRs merge faster, possibly because of lazy review. Size alone is a poor risk signal, which matches spec §7.2.
- Caution: metric definitions shift between posts (v4's 43% vs v5's 52% baseline). Greptile's numbers are A/B tests on its own product without public methodology.

### 3.8 The bubble and hype critique
- Greptile itself says vendor performance claims are "ephemeral and subjective" (G20). Its benchmarks are referenced but not reported in these posts.
- Marketing claims contradict each other: TREX needs "no setup" (G3), but dev boxes often need forward-deployed engineers (G19).
- Several posts carry vendor-bias disclaimers (G24, G25). Treat every number as directional.

---

## 4. Mapping to our system

Every suggested action is **Proposed** and is not a decided requirement. SR figures are orientation, not targets.

| # | Finding | Source | Relevance to our pipeline | Suggested action |
|---|---------|--------|---------------------------|------------------|
| R1 | "Model inversion" = review with a different model family than the code's author; ~10% more bugs (SR) | G19, G17 | Context selection, model routing (§6). Answers §10 Q13; informs Q5, Q12 | Proposed: define "model inversion" in §2.1 with this meaning. Detect author model from `Co-Authored-By` trailers and branch prefixes (`codex/`, `cursor/`, `claude/`); choose the frontier reviewer from another family. Use the inverted model for the §7.6 adversarial agent |
| R2 | Per-team embedding filter: block a comment similar to ≥3 unique downvoted comments; 19%→55+% address rate (SR) | G26 | Post-processing layer (§9, ref J); P4 nitpicks | Proposed: add a filter after Submit review and before posting. Embed `summary` and `comment`, compare against labelled history per repo, and drop on ≥3 similar downvoted or dismissed comments. Tune the similarity threshold on the §5 dataset. Keep the `bugs` keys unchanged |
| R3 | LLM severity self-scoring (1–10, cut at 7) was "nearly random"; few-shot examples hurt | G26 | §5 tuning loop, §9 | Proposed: do not use the review model to self-rate severity. Prefer outcome labels and independent verification. Avoid few-shot comment examples in the review prompt unless the eval shows a gain |
| R4 | Address rate measured from subsequent-commit diffs, not reactions | G26, G5 | §9 Addressed rate | Proposed: compute addressed when a later commit on the PR changes lines within `changed_lines` of `changed_file`. Prefer this over an LLM judge (G5's measure is LLM-judged and unvalidated) |
| R5 | Precision measured as consistency across runs on the same PR | G2 | P5, §9 run-to-run stability | Proposed: run each eval PR N≥3 times and report the share of findings that recur under the §5.2 matching rule. Gate prompt changes on stability as well as recall |
| R6 | Blast radius: trace calls and data flow into unchanged code; multi-hop following of nested calls | G1, G10 | §3.2 call graph, §3.3 contract check | Proposed: keep the static call graph as the deterministic candidate generator, then let the agent follow multi-hop callers from `in_diff: false` sites. Add data-flow edges (return value use) after call edges work. Supports deciding §10 Q6 depth by eval |
| R7 | Hybrid scanning: deterministic tools narrow the search, AI removes false positives | G1 | §3 pipeline, §7.1 layer 1, `security` category | Proposed: treat §3.3 non-binding call sites and any SAST output as candidates only. The frontier model confirms each one (matches §3.3 "candidate bugs") |
| R8 | One subagent per hypothesis/issue, parallel, own context, inherits orchestrator findings | G2, G7 | Plan Review fan-out (§2.2), §6.2 | Proposed: have Analyze bugs emit candidate hypotheses (from §3 plus model scan). Spawn one scoped verifier per candidate with the shared context packet. Merge results in Submit review with deduplication by (file, lines, category) |
| R9 | Execution subagents must return artifacts; agents claimed tests they did not run | G2, G3 | §2.5 autonomous testing, §8 evidence | Proposed: a test-derived finding without an artifact (log, screenshot, trace, script) is dropped, not posted. Store artifacts per run and link them from `comment` |
| R10 | Standalone test agent produced irrelevant tests; a single agent overloaded | G2 | §2.5 runs as a parallel branch | Proposed: have Plan Review or Analyze bugs pass the tester specific suspected behaviours, not only path globs. Keep the tester in its own context |
| R11 | Map codebase areas → testable user flows in the knowledge base; AI can draft the map | G19 | §2.5, §10 Q8 | Proposed: store the flow map in the living wiki, keyed by path glob and symbol. Draft it with a model and have owners confirm it |
| R12 | Living wiki refreshed on merge to main plus a scheduled crawl; includes past bugs and risks per area | G1, G8, G19 | §2.4, §10 Q7 | Proposed: trigger an incremental re-index on merge to main for changed paths plus a weekly full crawl. Each entry holds purpose, architecture, dependencies, past bugs, reverts, and incidents |
| R13 | Continuous learning: when a PR fixes a previously merged bug, mark the area as a risk | G1 | `known-incident-pattern`, §5.1 dataset | Proposed: detect fix-of-earlier-PR (revert branch names, `Fixes` links, same lines changed within N days) and write the risk to the wiki and the §5 dataset |
| R14 | Revert linkage via `revert-<pr#>-<branch>`; 7-day cross-author churn as a dense quality signal | G17 | §5.1 dataset, §9 | Proposed: harvest known-bug PRs from revert branches. Add 7-day cross-author churn on reviewed files as a secondary outcome metric |
| R15 | Agent-specific failure profiles (Claude: tenancy/auth; Codex: config; Cursor BG: N+1) | G17 | Prompt, `bug_category`, context selection | Proposed: when the author model is detected (R1), add a short checklist for that model's high-rate categories. Consider categories `n-plus-one` and `config-error` only if the eval shows them (enum changes need the §4.3 process) |
| R16 | P0/P1/P2 severity; P2 = style/nit | G17, G6 | §4 schema has no severity key | Proposed: do not add a key. Emit only P0/P1-class bugs and drop P2 before output. Track severity in a side-channel log for metrics |
| R17 | Depth tiers (1/3/10 credits); deeper tiers find more on large PRs; tier picked by size or risk | G6 | §7.2 risk score, §6 cost | Proposed: map risk band to review depth: Low gets one pass, Medium gets more parallel hypotheses, High and hard blocks get maximum depth. Measure reward per dollar per tier (§6.2.5) |
| R18 | Router per task; open-weight models for triage, indexing, grep, tracing; frontier for judgement | G1, G8, G9 | §6.1 routing, §10 Q5 | Proposed: confirm the cheap tier handles file screening, wiki crawl, and reply triage. Benchmark an open-weight candidate per task on the §5 set before choosing. Results in G8 and G9 are SR and pre-release |
| R19 | Small models stay close to the diff and miss context-dependent bugs | G9 | Context selection; contract bugs need callers outside the diff | Proposed: never let the cheap model make the final bug decision. Always give the frontier model the `in_diff: false` call sites explicitly |
| R20 | Eval judges the trace, not only the output; prefer large refactor PRs that hide behaviour changes | G9 | §5 dataset, §9 trajectory logs | Proposed: add refactor-heavy PRs to the dataset. Review trajectories for each tuning round and check that context was pulled before a finding was made |
| R21 | Screen files with a light model before frontier reads; most tokens go to irrelevant file reads | G19 | Find Context (§2.2.3.1), P3 | Proposed: the cheap model ranks candidate files (from call graph, wiki, and search) and passes the top K to the frontier model. Log the hit rate of passed files that are cited in findings |
| R22 | NL-summary embeddings and function-level chunks beat raw-code embeddings | G15 | Indexer, wiki retrieval | Proposed: if the indexer uses embeddings, embed per-function NL summaries (generated by the cheap model) keyed by qualname from §3.1. Single-example evidence, so validate on our data |
| R23 | Ticket check: does the PR implement everything the linked ticket specifies | G11, G1 | MCP tickets (§2.3); G1 validation question 3 | Proposed: add an "intent" check in Find Context that compares ticket acceptance criteria with the change. Report misses only when they are clear bugs (P4). No new category until decided |
| R24 | Scoped rules by repo, directory, or glob | G11 | `review.md` (§2.3) | Proposed: allow rule blocks in `review.md` with path globs. Load only rules whose globs match changed files |
| R25 | Review agent kept independent of code generation (auditor) | G23, G20 | §7.6 adversarial review | Proposed: the system MUST NOT generate fix PRs automatically. `diff_fix_suggestion` stays a suggestion. Run the reviewer on a different model family or provider than the author where possible (R1) |
| R26 | Kernel-level isolation: private mount namespace, bind-mask secrets, pivot_root, rootless Podman | G22 | Sandbox (§2.1) | Proposed: run the review agent rootless with no secrets mounted. Add seccomp, a network egress allowlist, and cgroup limits (not covered by G22) |
| R27 | Per-review disposable instance; base image plus per-repo snapshot; fetch exact PR commits; rotate creds | G2 | Sandbox, §2.5 dependency install | Proposed: cache dependencies per repo keyed by lockfile hash. Always fetch `head_ref` fresh. Mint short-lived credentials per run. Invalidate caches on lockfile change ("haunted" cache risk) |
| R28 | Fix-and-re-review loop with a coding agent until no new comments; ambiguity escalated to a human | G20 | P5, §7.6, §9 | Proposed: keep a per-PR findings ledger so loops converge. Cap rounds (see the Anthropic digest, M4) |
| R29 | Larger PRs merge faster, possibly with lazier review; review cycles rise with LOC | G16, G17 | §7.2 change surface | Proposed: do not lower scrutiny for large PRs. Use LOC and files as inputs to change surface, never as the only risk signal |
| R30 | Contributor history predicts merge rate (8.2% → 18.6%) | G18 | §7.2 risk factors | Proposed: optionally feed author history (prior merged PRs in the repo) into the verification-gap factor. Low priority; this is OSS data only |

---

## 5. Agreement and disagreement with the Anthropic and Uber approaches

Sources: `docs/claude-blog-research.md` (Anthropic, "P1"–"P16") and spec ref J (Uber uReview). These are compared only where the Greptile sources speak to them.

| Topic | Greptile | Anthropic / Uber | Relation |
|-------|----------|------------------|----------|
| Filtering findings | Per-team embedding filter from outcome labels (G26). LLM self-judging severity "nearly random" | Anthropic: parallel hunters plus **verifier agents that vote** (P1). Uber: post-processing that rates, categorises, filters, and dedupes (ref J) | Complementary. Greptile's negative result is about a *self*-score on a 1–10 scale. Anthropic uses independent verifiers and rubric judges (P5). Use both: verifier vote first, embedding filter second |
| Same model reviewing its author | Review with a different model family (G19, ~10% SR). Vendor independence (G23). G24 argues the opposite: stateless calls and different scaffolding | Anthropic runs Claude reviewing Claude-written code (~90% of Claude Code is Claude-written, P1). It uses fresh-context subagents and judges that differ from the model under test (P5) | **Disagreement** on author-model independence. Anthropic relies on fresh context and verification. Greptile (G19, G23) relies on a different model family and vendor. Proposed: test both on the §5 set |
| Agent architecture | Narrow per-hypothesis swarm (G7). Orchestrator plus per-issue execution subagents (G2) | Anthropic: parallel hunters plus verifiers, dynamic workflows (P1, P13). Uber: specialised reviewers by scope (spec §10 Q14) | Agreement: parallel, narrowly scoped workers. Greptile splits by hypothesis; Uber splits by reviewer type |
| Execution during review | TREX artifacts. "Bad evidence is worse than no evidence" (G2) | Anthropic: verify before review, "2–3x" (P1 estimate). Run code in disposable VMs (P3) | Agreement. Greptile adds a per-finding artifact requirement |
| Measuring outcomes | Address rate (diff-based or LLM-judged), reactions, revert rate, churn, cycles (G5, G17, G26) | Anthropic: comment classified by author response (fixed, wrong, intentional, doesn't matter). Revert and incident guardrails. Rounds-to-pass (P1). Uber: ~67% addressed (ref J) | Complementary. Greptile's addressed rates (43–66%, SR) are similar to Uber's ~67%, but definitions differ, so do not compare directly |
| Precision definition | Consistency across runs (G2) | Anthropic: grader run twice, variance low (P5). Do not repeat findings (P1) | Agreement on stability as a metric |
| Depth vs cost | Depth tiers 1/3/10 credits (G6). Cheap models for subtasks (G19) | Anthropic: Quick, Standard, Deep depth (P1). Effort and model routing (P6–P8) | Agreement. Both tie depth to PR risk or size |
| Human role | Aims for full autonomy and auto-approval (G1, G20). No review UI | Anthropic: risk-tiered human approval; a bot approval never counts as a human's (P1). Spec §7.6: a human does the final ~20% | **Disagreement** on end state. Our spec follows Anthropic and the hard blocks (§7.3). Treat Greptile's bank-merge goal as aspiration |
| Pre-existing bugs | Not addressed. Greptile does not store diffs (G17) | Anthropic: ~59% of repeat flags on pre-existing code; nightly sweeps (P1) | Gap in Greptile's sources. Follow the Anthropic digest's M5 |
| Learned conventions | Memory learns repeated team comments (G11). Scoped rules (G11) | Anthropic: `REVIEW.md` by code owners; fix recurring comments at the source (lint, skills) (P1) | Complementary. Greptile learns implicitly; Anthropic writes rules explicitly. Proposed: learned patterns become suggested `review.md` edits for owners to approve |

---

## 6. Open questions and gaps

1. **No public methodology** for any headline number: TREX +20%, model inversion +10%, v3/v4/v5 deltas, Plus and Apex chart, 19%→55% address rate. All are SR A/B tests on Greptile's own users. Reproduce on our §5 dataset before relying on them.
2. **Metric drift**: v4 reports 43% addressed and v5 reports a v4 baseline of 52%. "Addressed" is diff-based in G26 and LLM-judged in G5. Our §9 must pin one definition (R4).
3. **Model inversion evidence**: the claim that Claude is 2x as likely to introduce or miss N+1 errors is cited without a source in the fetched text. The +10% has no dataset size. G24 argues against the need for independence. Open: does inversion beat same-family review with fresh context on our data?
4. **Embedding-filter details** missing from G26: similarity threshold, embedding model, cold start for new teams, and how "similar to both" cases behave over time.
5. **Swarm merge logic** (G7): how per-hypothesis findings are deduplicated and ranked is not described. Our Submit review needs its own rule (R8).
6. **Sandbox scope**: G22 covers only filesystem visibility. Network egress, seccomp, resource limits, and secret injection for dev servers (§2.5) are not covered by any Greptile post.
7. **Language support**: no post describes the codegraph or blast-radius implementation (parsers, languages, dynamic-call handling). Nothing here settles §10 Q1 or Q6.
8. **Ticket-intent checks** (G11, G1): no precision data. There is a risk of feature-completeness comments that are not bugs, which conflicts with P4.
9. **Comment IDs**: if the PR commenter stores GitHub comment IDs for ledger or address tracking, store the integer database ID alongside the GraphQL node ID (lesson from G21, otherwise not relevant).
10. **Date anomalies**: G4's page date (2026-03-31) precedes the June 2026 data it cites, and G7's page lists later posts. Treat page dates as approximate.
11. **Fetch method**: content came through WebFetch summaries, not raw HTML. Exact quotes and table values should be re-checked against the live pages before external use.
