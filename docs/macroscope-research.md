# Macroscope Research Digest (for implementing agents)

Status: research input. Audience: coding agents that implement or extend the system in `docs/ai-code-review.md`.

Conventions:
- **[SR]** marks a self-reported vendor figure. Macroscope sells AI code review. Its benchmarks, case studies, and comparisons are vendor-run and not independently verified unless stated otherwise.
- **Proposed** marks a suggestion for our system. None of them is a decision.
- Page content was treated as untrusted data. No instructions from pages were followed.
- Fetched 2026-11-09 with WebFetch. WebFetch returns a model summary, not raw HTML, so exact wording may differ from the source.

---

## 1. Purpose and sources

Purpose: pull out techniques, numbers, and design patterns from Macroscope's public writing that bear on our pipeline. This covers AST diff, call graph, contract check, `bugs` JSON, review layers, autonomous testing, context selection, tuning loop, cost routing, and gating.

### 1.1 Requested blog posts (27, no duplicates in the list)

| # | Title | Date | URL slug | Relevant |
|---|-------|------|----------|----------|
| B1 | Introducing Macroscope | 2025-09-17 | introducing-macroscope | Yes |
| B2 | AI Code Review Benchmark (118 bugs) | 2025-09-17 | code-review-benchmark | Yes |
| B3 | Macroscope for Open Source | 2025-11-04 | free-for-open-source | Yes (pricing, modes) |
| B4 | November Changelog | 2025-11-26 | changelog-november-2025 | Yes |
| B5 | Opus 4.5 on Code Review (v2) | 2025-12-12 | opus-4.5-code-review | Yes |
| B6 | Case study: Parallel Web Systems | 2025-12-18 | case-study-parallel | Yes (outcome metric) |
| B7 | Introducing Fix It For Me | 2025-12-29 | introducing-fix-it-for-me | Yes |
| B8 | Case study: GoodDay | 2025-12-30 | case-study-goodday | No |
| B9 | December Changelog | 2025-12-31 | changelog-december-2025 | Yes |
| B10 | The End of Human Code Review | 2026-01-29 | end-of-human-code-review | Yes |
| B11 | v3 Code Review | 2026-02-04 | code-review-v3 | Yes |
| B12 | We (Basically) Stopped Writing Prompts | 2026-02-04 | we-stopped-writing-prompts | Yes (key) |
| B13 | Introducing Approvability | 2026-02-13 | introducing-approvability | Yes (key) |
| B14 | Usage Based Pricing | 2026-03-27 | usage-based-pricing | Yes |
| B15 | (re) Introducing Agent | 2026-04-01 | introducing-agent | Partial (context sources) |
| B16 | Agent Pricing | 2026-04-15 | agent-pricing-update | Yes (cost model) |
| B17 | Check Run Agents | 2026-05-27 | introducing-check-run-agents | Yes (key) |
| B18 | Beyond Tokenmaxxing | 2026-06-11 | beyond-tokenmaxxing | No |
| B19 | Improved Code Review + Detection Mode | 2026-06-24 | new-code-review-pipeline-detection-mode | Yes (key) |
| B20 | Open Source Models for Check Run Agents | 2026-07-16 | open-source-models-for-check-run-agents | Yes (routing) |
| B21 | Introducing Macroscope CLI | 2026-07-22 | introducing-macroscope-cli | Yes (layer 2) |
| B22 | Expanding Detection Mode | 2026-08-28 | expanding-detection-mode | Yes |
| B23 | Introducing Murmur | 2026-09-03 | introducing-murmur | Partial (sandbox, evidence) |
| B24 | Case study: Pydantic | 2026-09-18 | case-study-pydantic | Yes |
| B25 | MacroscopeBench | 2026-09-22 | macroscopebench | Yes (key) |
| B26 | Case study: Cars24 | 2026-09-29 | case-study-cars24 | No |
| B27 | October Changelog | 2026-10-28 | changelog-october-2026 | Yes (key) |

### 1.2 Extra posts from https://macroscope.com/content

The index lists 92 posts with no dates. Most are SEO comparison pages. Several repeat blog posts above: the Pydantic case study, December changelog, v2/Opus 4.5, the 2025 benchmark, Fix It For Me, Check Run Agents, usage-based pricing, and Murmur FAQ. Those were skipped as duplicates. These 10 were fetched because they add technical detail:

| # | Title | Date | Slug (/content/...) |
|---|-------|------|---------------------|
| C1 | How AI Code Review Catches Bugs Across Files | 2026-05-06 | how-ai-code-review-catches-bugs-across-files |
| C2 | What Is Approvability? | 2026-05-01 | what-is-approvability-auto-approve-safe-pull-requests |
| C3 | Context-Aware Code Review (production context) | 2026-06-18 | context-aware-ai-code-review-production-context |
| C4 | Precision vs Recall: Tune Detection Mode | 2026-07-08 | ai-code-review-precision-vs-recall |
| C5 | How to Control AI Code Review Costs | 2026-08-18 | control-ai-code-review-costs-spend-limits |
| C6 | Why Train a Specialized Model (25% fewer tokens) | 2026-09-22 | why-train-a-specialized-model-for-ai-code-review |
| C7 | AI Code Review Benchmark 2026: Best Models | 2026-09-22 | ai-code-review-benchmark-best-models |
| C8 | Make AI Code Review a Required Check on GitHub | 2026-10-19 | how-to-make-ai-code-review-a-required-check-github |
| C9 | False Positives and Signal-to-Noise | 2026-11-02 | ai-code-review-false-positives-signal-to-noise |
| C10 | Bug Detection by Language | 2026-11-02 | ai-code-review-bug-detection-by-language |

### 1.3 Not relevant

- **B8 GoodDay case study**: leadership visibility and daily change digests. No review mechanics.
- **B18 Beyond Tokenmaxxing**: productivity metric (pushed vs landed coding time). Not about review quality.
- **B26 Cars24 case study**: landed-code metrics and capitalisation. It only mentions Code Review and Approvability in passing.
- **Remaining /content pages** (comparisons, pricing pages of other vendors, DORA, PR summaries, productivity): marketing or general explainers with no new technique.

### 1.4 Failures and data-quality notes

- No fetch failed.
- The `/content` index shows no dates. Dates for C1–C10 come from each page.
- **Date inconsistencies inside pages**:
  - B3 (dated 2025-11) cites average costs for "Sep 29–Oct 28, 2026".
  - C4 (dated 2026-07) and C5 (dated 2026-08) cite data from later dates.
  - C1 says "last updated Oct 2026".
  - Treat these pages as edited after publication.
- **Leaderboard conflict**: B25 lists Claude Opus 5.5 (max) top at 83.9. C7 and C6 list GPT-6 Astra (max) top at 78.0, with Claude Opus 5 at 69.9. These are different model sets and probably different snapshots of the same benchmark. B25's excerpt showed only 5 of 48 models.
- **Signal/noise definitions**: C7 defines signal/noise as valid Medium+ findings ÷ (invalid + Low). B25's column was not defined in the fetched summary.

---

## 2. Per-post digest (relevant posts)

### B1 Introducing Macroscope (2025-09-17)
- Product is pitched as an "understanding engine": PR review, PR summaries, codebase Q&A, and leadership status.
- Review technique: for each change, it explores the surrounding repo and collects code references. LLMs then judge impact. The vendor credits this context step with less hallucination and noise.
- Internal benchmark [SR]: 100+ runtime bugs, 45 OSS repos, 8 languages. It caught 5% more than the next tool with 75% fewer comments than the next tool. Style issues were excluded.
- The post names no AST or graph. Integrations: Linear, Jira.

### B2 AI Code Review Benchmark, 118 bugs (2025-09-17)
- Dataset: bug-fix commits from 45 repos in 8 languages. An LLM keeps only self-contained runtime bugs and writes a description from the fix. `git blame` finds the introducing commit; an LLM verifies it and humans spot-check a subset.
- Harness: replay each introducing commit as a fork PR. Each tool runs on default settings in its own PR.
- Matching: an LLM matches comments to bug descriptions. Humans verified all matches and spot-checked 5 non-matches per tool.
- Results [SR]:

  | Tool | Detection | Avg runtime comments/PR |
  |------|-----------|------------------------|
  | Macroscope | 57/118 (48.3%) | 2.55 |
  | CodeRabbit | 45.8% | 4.69 (10.84 total) |
  | Cursor Bugbot | 42.4% | 0.91 |
  | Greptile | 17/72 (23.6%) | 1.61 |
  | Graphite Diamond | 18.3% | 0.62 |

- Stated limitations: Macroscope tuned its own pipeline on these results, sample sizes were uneven, false negatives were mostly unchecked, and no tool caught most bugs.
- C10 per-language results [SR]: Macroscope scored 86% in Go and 50% in Python. TypeScript was 27–36% for all tools. Per-language samples have 9–22 bugs, so the percentages are noisy.

### B3 Open Source (2025-11-04)
- Free for non-commercial OSS. OSS defaults to Budget mode, and Ultra is paid only.
- Price [SR]: $0.05/KB of diff, 10 KB minimum. Average billed review was 76¢ on Balanced and 55¢ on Budget.
- Output features: PR summary written into an empty description, bug comment with an applicable diff fix, and follow-up Q&A through a code research agent.

### B4 November Changelog (2025-11-26)
- Manual trigger: comment `@macroscope-app review`.
- Fix rendering: the system auto-picks one of three formats. These are a GitHub suggestion block, a unified diff block, or a text-only explanation.
- **Web research for library docs** (via Parallel search API). Vendor says this "meaningfully reduces false positives about third-party libraries" [SR, no number].

### B5 Opus 4.5 / Code Review v2 (2025-12-12)
- Pipeline moved from mostly GPT-5.1 to Opus 4.5 only.
- Results vs the prior pipeline [SR]: recall +40%, false positives −10%, F1 +25%, latency +40% (262 s average vs 183 s).
- Rationale: 262 s is still within CI check budgets. The model choice weighed cost, latency, and quality together.

### B6 Parallel case study (2025-12-18)
- 4,800+ PRs reviewed and 6,000+ suggestions [SR].
- **Disapproval rate 2.2%** over 30 days, measured as the share of comments with a thumbs-down [SR]. This is a cheap, outcome-based precision proxy.
- Workflow: the author addresses AI comments first, then requests human review. Humans focus on architecture and design.

### B7 Fix It For Me (2025-12-29) and B9 December Changelog (2025-12-31)
- Flow: the author replies "fix it for me" on a finding. The bot creates a branch, commits the fix, and opens a PR into the feature branch, which auto-merges once checks pass. On failing checks or annotations, it reads the logs and pushes more commits ("self-heal").
- Guardrail: auto-merge only targets feature branches. PRs into main always need a manual merge.
- GitHub suggestion blocks only work on added lines, so the bot opens its own PR for multi-line or removed-line fixes.
- B9 changes: the check run is ✅ with no issues and **neutral** when correctness issues are found (branch rules on check names had to be updated). Per-developer settings override repo defaults. Elixir and Starlark added.
- No fix success rates are published.

### B10 The End of Human Code Review (2026-01-29, CEO opinion)
- Thesis: humans will review near-zero PRs, and review will become always-on and automatic.
- Architecture claims: the agentic pipeline explores repo code, issue trackers, git history, the web, and related repos. A continuous optimizer picks the model and prompt per language and repo. Sub-agents validate findings with coding agents. An orchestrator checks test sufficiency through sub-agent attestation and assesses blast radius.
- Auto-approve only if all hold: (1) trusted AI review finds no unresolved correctness issue; (2) small blast radius; (3) strong author or agent trust profile; (4) no org policy guardrail is hit (for example, auth handlers always need a human).
- Humans write policy instead of approving PRs. This is opinion with no data.

### B11 v3 Code Review (2026-02-04)
- Results [SR]: precision 75% → 98%; 22% fewer comments; nitpicks −64% in Python and −80% in TypeScript; up to 3.5× more harmful bugs (about +4 per 100 PRs). After launch: thumbs-up +30%, comments per PR −37%, addressed rate +10%.
- Technique: an agentic approach plus **auto-tune**, which jointly picks prompt × model × language (see B12).
- **v2 used native AST parsing for about 12 languages. v3 reviews every file in the PR**, including configs, scripts, and docs.
- Four severity levels by harm if shipped: CRITICAL (unrecoverable data loss or breach), HIGH (crash, hang, degraded security), MEDIUM (core function broken but recoverable), LOW (cosmetic or rare edge case).
- Recall values are not published.

### B12 We (Basically) Stopped Writing Prompts (2026-02-04), the key technique post
- Problem: hand-tuning prompts per language × model did not scale. Each benchmark run took hours on 500+ labelled bugs.
- What failed: A/B tests were too slow and went stale with model updates. **Few-shot examples were harmful across languages**: a Python example helped Python but hurt TypeScript.
- Auto-tune is built on ACE (Agentic Context Engineering, arXiv:2510.04618). It extends ACE with **joint search over model + prompt + params**, run in parallel with hill-climbing. DSPy, by contrast, tunes prompts for a fixed model.
- Strategy "seeds", such as "trace every code path" or "reason like an adversary", are tested across models. Winners are refined.
- **Severity-weighted objective**: each tier is worth 5× the tier below it, so Critical = 125× Low. False positives are penalised. Raw F1 is not the objective.
- Edit "learning rate": low allows wording tweaks only, high allows section rewrites. It decays on success and rises on plateau.
- Anti-overfit: the curator LLM must find patterns across a batch rather than patch single failures. Scoring uses a held-out set, re-split every epoch.
- **Detection vs validation split**: permissive detection, then strict validation that rejects hedged, speculative, or unprovable findings.
  - On the same recall prompt, Opus flagged 199 issues and GPT-5.2 flagged 3,923 [SR].
  - Model-specific reject signals: GPT-5.2 modal words ("could", "potentially", "may") correlate with false positives. Gemini 3 filler ("wait", "however") precedes wrong answers.
  - Different models can serve different subtasks.
- Held-out results [SR]: TypeScript about 3.5× more high-severity bugs and about 50% fewer false positives; Python about 65% fewer nitpicks and 45% fewer false positives; Java about 50% fewer bad or low-value comments.
- Limits: one full optimisation makes thousands of LLM calls, so it runs only when adding a language or upgrading a model. It needs a labelled set, and building that is "the main work".

### B13 Introducing Approvability (2026-02-13) and C2 What Is Approvability (2026-05-01)
- It runs as a check run named `Macroscope - Approvability Check`, next to `Macroscope - Correctness Check`. It is **off by default**.
- Verdicts: Approved; Would Approve (informational); Recommend Approval (passes, but a CODEOWNER must approve); Needs human review.
- Three hurdles must all pass:
  1. **Ownership**: the author owns all changed files per CODEOWNERS. Off by default.
  2. **Eligibility**: a *separate* LLM agent from the correctness agent reads the diff and the repo. It returns `SUGGEST_APPROVABLE` or `SUGGEST_HUMAN_REVIEW`.
  3. **Correctness**: any MEDIUM+ finding blocks approval.
- Eligibility inputs: change type and runtime impact, scope, complexity, schema impact (additive vs breaking), author trust (role, ownership, prior commits in touched paths), open human comments, and `.macroscope/approvability.md` (highest priority).
- Auto-approve: docs, tests, code behind a disabled flag; copy changes and endpoints following an existing pattern; single-function fixes (null check, off-by-one, inverted condition); renames and moves; minor CI tweaks.
- Always escalate: breaking schema or API contract changes; gating logic (flags, permissions, routing); infra (Kubernetes, Terraform, Helm); security, auth, billing, sensitive data regardless of size; CODEOWNERS edits; any change to `approvability.md` itself.
- Rule: "any doubt → defer to a human."
- Guardrails: never approves its own bot's PRs. **Diff-hash caching**: a rebase with an identical diff reuses the verdict. Every decision posts a check run with a verdict comment (audit trail). Approval is separate from merge.
- The compliance caveat is stated in the post: SOC 2 controls often assume human review.

### B14 Usage Based Pricing (2026-03-27) and B16 Agent Pricing (2026-04-15)
- Rationale [SR]: per seat, commits rose 1.8×, reviews 1.9×, and review size 1.7× in 3 months. Agents broke the seat model.
- Prices [SR]: review $0.05/KB of diff with a 10 KB floor ($0.50); Status $0.05/commit; Agent $0.01/credit, where credits = raw LLM cost × 1.05 / $0.01, with 1,000 free credits per month.
- Spend controls: $10 per-review and $50 per-PR caps by default, a monthly limit, exclusions by repo, author, or file type (lockfiles, generated code), and a manual-only option.
- Comparison cited: Claude Code Review at $15–25 per review [SR, third-party claim].

### B15 (re) Introducing Agent (2026-04-01), partial
- Context and tool sources: Sentry, Amplitude, read-only GitHub API, GCP logging, Linear, Jira, LaunchDarkly, PostHog, BigQuery, plus any MCP server with per-tool access control.
- Event triggers: new commit, new PR, schedule, and webhook.
- Webhook destinations must pass an allowlist at both config time and run time. This is a pattern for safe outbound posting.

### B17 Check Run Agents (2026-05-27)
- Format: one markdown file per agent in `.macroscope/check-run-agents/` on the default branch. Each agent posts its own GitHub check run.
- Frontmatter fields:
  - `model`: Opus/Sonnet/GPT family. Default `claude-opus-4-6`.
  - `reasoning`: off, low, medium, or high.
  - `effort`: how far the agent explores beyond the diff.
  - `tools`: defaults are `browse_code`, `git_tools`, `github_api_read_only`, and `modify_pr`, plus about 12 opt-in tools.
  - `include` / `exclude` globs. The agent does not run if no file matches.
  - `conclusion`: `neutral` (default) or `failure`, which blocks merge.
- `input` modes: `incremental` (default, files changed since the last review), `full_diff`, **`code_object`** (up to 20 parallel agents, one per function or class), `pr_metadata` (no diff).
- Use cases: style per language (six internal agents), architecture boundaries, ticket-requirement traceability, analytics event removal, audit logging and auth middleware on new endpoints.
- Cost: raw LLM cost + 5%. Usage is exported per agent and per PR as CSV.
- Vendor recommendation: start with one narrow agent set to `neutral`. Promote it to `failure` once its output is reliable. Write instructions as specific, runbook-style guidance.

### B19 Improved Code Review + Detection Mode (2026-06-24)
- New pipeline vs the previous one [SR]: Critical +71%, High +27%, Medium +50%, valid findings +31% overall, with a "modest" rise in false positives and latency.
- Two modes:
  - **Prefer Coverage**: 85% precision [SR]. For teams whose coding agents triage comments, since an agent filters false positives cheaply.
  - **Prefer Precision**: 97% precision [SR]. For humans who read comments in GitHub.
- Set per repo or per developer; the developer setting wins. Recall per mode is not published.

### B20 Open Source Models for Check Run Agents (2026-07-16)
- Adds kimi-k2-7-code, glm-5-2, minimax-m3, and qwen-3-7-plus.
- Rationale: suites of many agents per PR make inference cost material. Pick the model per agent.
- No benchmark numbers are given. C5 adds that GPT-5.6 Terra and Luna prices were cut in July 2026.

### B21 Macroscope CLI (2026-07-22)
- `macroscope codereview` gives a local review before push, with a terminal UI that streams issues.
- Each review runs in an **isolated git worktree** snapshot by default. `--in-place` reviews the working directory instead.
- Agent plugins for Claude Code, Codex, Cursor, and OpenCode. The agent validates findings and then fixes them.
- **Autoloop**: review → validate → fix, up to **5 cycles**.
- Billed in agent credits. C5 adds per-user and workspace CLI spend limits, which default to unlimited.

### B22 Expanding Detection Mode (2026-08-28) and C4 Precision vs Recall (2026-07-08)
- Four modes [SR prices from C4]:

  | Mode | Price per KB | Use |
  |------|-------------|-----|
  | Budget | $0.025 | Low-risk bumps and config |
  | Balanced (default) | $0.05 | Agents triage comments |
  | Precise | $0.06 | Adds an **agentic validation** pass before posting |
  | Ultra | $0.20 | Many more parallel agents and stronger models, for payments, auth, migrations |

- Mode selection, most specific wins: repo setting < author setting < PR label (`macroscope-<mode>`, sticky for later pushes) < comment ("do an Ultra review").
- Every mode starts from an agentic review with full-repo context. No per-mode precision or recall figures are given.

### B23 Murmur (2026-09-03), partial
- Each coding agent gets its own cloud VM with the full stack running. Macroscope's own VM runs a single-node Kubernetes cluster with dozens of microservices.
- Agents run tests and drive the UI, then **attach screenshots, recordings, and other evidence** to PRs.
- PRs are self-driving: agents answer review comments, fix CI, rebase, and resolve conflicts.
- Orchestration (the "Director") runs locally, while execution runs in the cloud.
- Murmur produced about 90% of Macroscope's shipped code over 2 months [SR].

### B24 Pydantic case study (2026-09-18)
- PR volume went from 100–150 to about 500 per week. About 91% of PRs are auto-approved by Approvability, and the rest escalate [SR].
- **Mining a year of human PR comments**: the most frequent feedback from senior engineers was turned into Check Run Agents covering style, security, performance, and safety.
- It worked across repos without per-repo tuning [SR]. The approval criteria and thresholds are not disclosed.

### B25 MacroscopeBench (2026-09-22) and C7 Best Models (2026-09-22)
- Full dataset: 12,000+ validated bugs, 1,500+ public repos, 14 languages. Each bug is anchored to its introducing and fixing commits.
- Index subset: 195 commits. 144 contain at least one known bug and **51 are control commits with no known bug**, which makes precision measurable.
- Protocol: check out the commit and review the diff against its parent. The reviewer is not told whether a bug exists or where.
- Judging: a **recall judge** (LLM with codebase and doc search tools; each bug credited at most once per run) and a separate **precision judge** (only objective correctness issues introduced or exposed by the commit count; style and optimisation are invalid).
- Score: harmonic mean of recall and precision per run, averaged over **3 runs**. Variance between runs is "several percentage points".
- Cost is the inference cost per task with caching, excluding judging. Latency is the median wall-clock time.
- Severity labels are fixed at ingestion and the mix is tuned to separate models. Results are reported per severity bucket.
- Leaderboard samples [SR]:
  - B25: Opus 5.5 max scored 83.9 (R 83.8%, P 84.0%, $11.58, 10 min). GPT-6.1 Terra high scored 82.3 (P 91.2%, $0.57, 2.5 min).
  - C7: GPT-6 Astra max scored 78.0 (P 91.7%). DeepSeek V4.1 Flash had R 83.1% and P 63.5% at $0.38–0.75. Opus 5 ranged from 59.4 (low effort) to 69.9 (high effort).
  - Costs span about 20×. **Cost does not track quality.**
- Limits: one shared prompt and harness (no per-model tuning), LLM judges with no human adjudication, public repos only, and ground truth covers only bugs someone later found and fixed.
- Vendor recommendation: validate on about 20 of your own bug-containing merged PRs plus about 10 clean PRs, and score by severity.

### B27 October Changelog (2026-10-28)
- Approvability modes: Permissive (default) relies on verifiable evidence (Correctness, Check Run Agents, CI, review comments, CODEOWNERS). Strict asks for a human when risk or uncertainty remains.
- Dashboard: shows each verdict with its reason, exportable to CSV. **Preview Mode** shows what would be approved without submitting approvals.
- Run modes: always, manual only, or never. **Policy Repo**: `.macroscope/` files are inherited across repos, and a repo-local file overrides.
- **Correctness: removed the AST-walking context step.** Result: 14% faster and +5% recall with no precision change on MacroscopeBench [SR]. C3 dates this to 2026-10-16: "multi-step agentic approach … replaced fixed AST walkers".
- Other: `onlyApprove` skips PRs touching paths outside the listed patterns; auto rerun when the last blocking thread is resolved; Check Run Agents inherit the Correctness ignore list; `ignoreTests: false` lets one agent review tests.

### C1 Cross-File Bugs (2026-05-06)
- Method: find every reference to a changed function, read each call site, and check how the return value is used (comparisons, arithmetic, serialisation). Then compare the old and new contracts and **report only callers that break**, with file, line, and reason.
- Bug classes: caller/callee contract drift; type ripple (a renamed field breaks a serialiser); a new enum value falling through to a default in another file; cross-file concurrency invariants; config coupling.
- Worked example: a return type changes from cents (a number) to an object. Three files outside the diff break, one of them silently (a `> 0` check stops firing). The diff and its tests look fine.
- Platform: GitHub only. GitLab and Bitbucket are on the roadmap.

### C3 Context-Aware Review (2026-06-18)
- Three context layers: codebase (callers, types, migrations), production (Sentry and Datadog error volume and latency), and intent (linked Jira or Linear ticket compared with what the diff does).
- **Rank findings by blast radius**, not line count.
- The correctness agent uses MCP only if `tools: [mcp]` is set in `.macroscope/correctness/correctness.md`.
- Automated runs have no requesting user, so they can only use unrestricted workspace connectors.

### C5 Control Costs (2026-08-18)
- Per-run budget `maxBudgetPerRun` is checked *between turns*, so one turn can overshoot. A budget-stopped run concludes **neutral**, not failure.
- `maxRuns` caps runs per PR. It is essential for agents that write code, because their commits re-trigger the agent and can loop.
- The monthly limit is not reserved atomically, so concurrent runs can overshoot. Commits pushed after the limit is hit are skipped, not queued.
- Other clamps: `waitsFor` keeps up to 10 entries, and `waitsForTimeout` is 1–60 min.
- Vendor recommendation: tier models by consequence, with cheap models for mechanical checks and strong models for correctness.

### C6 Specialized Model (2026-09-22)
- A Fireworks fine-tune cut tokens by 25% "without degradation" in early runs [SR]. The base model and method are not disclosed.
- Review split into 5 subtasks:
  1. Detection: the best training target, since fix commits give ground truth.
  2. Severity grading.
  3. **Dedup and suppression**: the vendor says this drives most of the gap between 6:1 and 1.2:1 signal/noise.
  4. Comment writing: route to a general model.
  5. Fix generation: route to a coding model.
- Vendor recommendation: do not train first. Exhaust prompts, harness and tools, routing, and effort tuning first. Train only for a narrow, high-volume task with ground truth, a trusted benchmark, and fast rollback.

### C8 Required Check (2026-10-19)
- GitHub required checks: success, skipped, and neutral pass; failure or a missing result blocks. A required check that never reports stays "Expected" forever.
- Blocking Check Run Agents need `conclusion: failure` **and** `input: full_diff`, because `incremental` cannot fail.
- Scoped agents need `requiredStatusCheck: true` to report `skipped` outside their scope.
- **Gate tampering**: agent definitions are read from the PR head, so a PR author can weaken a gate. Fixes: add `.macroscope/check-run-agents/**` and `.macroscope/ignore.md` to `neverApprove`, and require CODEOWNERS review for `.macroscope/`.
- Rollout: keep agents advisory for 1–2 weeks, then block only for security, auth, billing, migrations, and infra. Style checks stay advisory.

### C9 False Positives / Signal-to-Noise (2026-11-02)
- Comments per bug detected [SR]: Macroscope 5.3, CodeRabbit 23.7, Bugbot 2.1.
- Martian third-party precision leaderboard (checked 2026-11-05 per the page): Macroscope 82.8%, Greptile 82.5%, CodeRabbit 71.7%, Copilot 65.8%. This is third-party, but still quoted through the vendor.
- Team metric: effective precision = comments that led to a code change ÷ total comments, measured weekly.
- Stated false-positive causes: missing business context, over-applied patterns, weak understanding of cross-system interactions, and style framed as a bug.

---

## 3. Cross-cutting themes

### 3.1 Review scope
- Every surface is scoped by **path globs** plus **trigger rules**: Check Run Agents use include/exclude, Approvability uses `onlyApprove`/`neverApprove`, and an ignore list skips generated code, vendored code, and lockfiles. Skipping by author (bots), label, and target branch exists as well.
- Correctness review is incremental by default: each push reviews the files changed since the last review. Full-diff review is required for blocking checks.
- Context reaches well beyond the diff: callers, git history, web docs, tickets, and production telemetry. **The trend is away from fixed AST walkers toward agent-chosen exploration**, a change Macroscope reports improved both speed and recall [SR] (B11 → B27).
- `code_object` input fans out one agent per function or class, up to 20. This is the closest analogue to our per-symbol change analysis.

### 3.2 Detection vs validation
- The consistent pattern is high-recall detection followed by a separate strict validation pass (B12, B22 Precise, B10 sub-agent validation).
- Validation heuristics include rejecting hedged language and unprovable claims. Some reject signals are model-specific.
- Where comments go sets the precision target. If a coding agent consumes them, use the coverage mode at about 85% precision. If humans read them, use the precision mode at about 97% (B19 [SR]).

### 3.3 Benchmark design
- Use real bugs only: fix commit → `git blame` → introducing commit, then replay the introducing commit as a PR.
- **Control commits with no bugs** make precision measurable.
- Use LLM judges with tool access, and keep the recall judge separate from the precision judge. Credit each bug once.
- Run 3 times and average. Variance is several points.
- Severity weighting: a 5× ladder in tuning (B12), and per-bucket reporting in the benchmark.
- Known weaknesses: the vendor tunes on its own benchmark, the judges are LLMs only, and only fixed bugs are labelled.

### 3.4 Prompt and model tuning
- Joint search over model × prompt × params beats prompt-only tuning (B12).
- Few-shot examples can hurt across languages.
- Re-run tuning on model upgrades, because calibration patterns shift.
- Routing: different models per subtask and per custom check. Open-source models are offered for cost (B20).

### 3.5 Approval gating
- Approval runs as a separate agent from correctness, behind a correctness gate (MEDIUM+ blocks) and an optional ownership gate.
- Policy lives as a markdown file in the repo and outranks defaults. Changes to the policy or gate files always escalate.
- Ship with a shadow mode first ("Would Approve" / Preview), then turn approval on. Default the conclusion to neutral.
- Cache verdicts by diff hash. Every verdict is a check run with a written reason, which gives an audit trail.
- Hard-escalate areas match our §7.3 almost exactly: auth, billing, security, schema/API contracts, infrastructure, and permissions/gating logic.

### 3.6 Pricing and usage limits
- Usage-based pricing: per KB of diff with a floor, and per agent credit at cost + 5%.
- Cost controls: caps per review ($10), per PR ($50), and per month; per-run dollar budget and maximum runs per PR; spend limits for the local CLI.
- Overshoot is known and tolerated: caps are checked between turns and not reserved atomically.
- A run stopped by the budget concludes neutral, so it never blocks the merge.

### 3.7 Act vs comment
- Default: comment, with a suggested fix in one of three rendering formats.
- Act only on request ("fix it for me") and only into a feature branch. Auto-merge happens only after checks pass, and main always needs a human.
- Autoloop (CLI) and Murmur close the loop: review → validate → fix, with a cycle cap (5) and `maxRuns` to stop self-triggering loops.
- Blocking is opt-in (`conclusion: failure`). The vendor advises advisory-first.

---

## 4. Mapping to our system

All suggested actions are **Proposed**.

| Finding | Source | Relevance to our pipeline | Suggested action |
|---------|--------|---------------------------|------------------|
| Removing the fixed AST-walking context step made review 14% faster with +5% recall [SR] | B27, C3 | AST diff, call graph, context selection | Proposed: keep §3 AST and call graph as the *deterministic candidate generator* for contract checks. Do not use them as the only context path; let the cheap model also explore beyond the graph. A/B test both on our dataset |
| v2 used AST for about 12 languages, v3 reviews every file [SR] | B11 | AST diff, open question §10.1 | Proposed: for non-Python files, fall back to a plain diff review by the frontier model rather than skipping them |
| Report only callers that break, with file, line, and reason | C1 | Call graph, contract check, bugs JSON | Proposed: §3.3 emits one candidate per *failing* call site only. The `comment` names the external caller file and line |
| Cross-file bug classes: return-type ripple, enum fall-through, config coupling | C1 | Contract check | Proposed: extend §3.3 beyond parameters to return-shape changes and use sites (comparisons, arithmetic). Add a category such as `return-contract-change` to the tuning labels |
| `code_object` input: one agent per function or class, up to 20 in parallel | B17 | AST diff, review layers, cost | Proposed: fan out frontier verification per `changed_symbol` from §3.1, with a configurable cap (default 20) |
| Separate high-recall detection from strict validation; reject hedged findings | B12, B22 | Bugs JSON, frontier review | Proposed: a two-stage Analyze bugs step (detect, then validate). The validator drops findings with modal hedging or no concrete failing path |
| Severity ladder Critical/High/Medium/Low, defined by harm | B11, B25 | Bugs JSON | Proposed: §4.2 keys are fixed, so carry severity in a sidecar record keyed by finding rather than a new key, or add it via a spec change. Use it for ranking and gating |
| Severity-weighted tuning objective (5× per tier) with false-positive penalty | B12 | Prompt-tuning loop §5 | Proposed: replace the raw recall/precision report in §5.2 with a severity-weighted score, and keep per-category metrics |
| Joint model × prompt search, held-out set re-split per epoch, batch-level edits | B12 | Tuning loop, model routing | Proposed: §5 loop searches over (model, prompt) pairs, scores on a held-out split, and the curator edits only from patterns across a batch |
| Few-shot examples hurt other languages | B12 | Tuning loop | Proposed: keep few-shot examples per language or omit them. Test removing them as a tuning arm |
| Dataset from fix commits → `git blame` → introducing commit, replayed as PRs | B2, B25 | Tuning dataset §5.1 | Proposed: build §5.1 records this way from our repos. Add revert/incident-derived bugs as a second source |
| Control commits with no known bugs | B25 | Tuning dataset, false positives | Proposed: add clean PRs at roughly 1 clean per 3 buggy (B25 uses 51/195) so precision is measurable |
| Separate recall and precision LLM judges with tool access; credit each bug once | B25 | Tuning loop matching §5.2 | Proposed: keep §5.2's file/line/category rule as the primary match, plus an LLM judge for near-misses. Have humans adjudicate a sample |
| 3 runs per config; several points of variance | B25, C6 | Run-to-run stability §9 | Proposed: run every tuning config 3× and report mean and spread. Feed this into the stability metric |
| Validate on ~20 buggy plus ~10 clean own PRs | C7 | Tuning dataset, model choice §10.5 | Proposed: use this as the minimum smoke set before choosing models |
| Cost spans 20× and does not track quality; effort level swings the score by ~10 points | C7, B25 | Cost routing §6 | Proposed: benchmark (model, effort) pairs, not models alone. Record cost per review in §9 |
| Dedup and suppression drive most of the signal/noise gap [SR] | C6 | Post-processing §9 | Proposed: build the §9 post-processing layer (dedup against prior findings and same-root-cause merge) before any model training |
| Don't train a model first | C6 | Cost routing | Proposed: no fine-tuning until prompt, routing, and effort tuning plateau on our benchmark |
| Prefer Coverage (~85% P) for agent consumers, Prefer Precision (~97% P) for humans [SR] | B19 | Review layers §7.1 | Proposed: a config flag for the comment audience. Layer 2 (local agent) uses coverage; layer 3 PR comments use precision |
| Four detection modes priced $0.025–0.20/KB; mode set per repo, author, label, or comment | B22, C4 | Cost routing, risk scoring §7.2 | Proposed: map the §7.2 risk band to review depth. Low uses a cheap pass; High or hard-block areas use a deeper multi-agent pass |
| Web research for library docs reduces third-party false positives | B4, B6 | Context selection | Proposed: add a docs-lookup tool (MCP) for third-party APIs touched by the diff |
| Production and intent context (Sentry, tickets); rank by blast radius | C3 | Context selection, MCP, risk §7.2 | Proposed: the cheap model pulls the linked ticket and error data for changed paths. Use call-graph in-degree as the blast-radius ranking key |
| Approvability: separate eligibility agent + correctness gate (MEDIUM+) + ownership gate | C2, B13 | Review layers §7.2–7.3 | Proposed: implement auto-approval as a separate agent from the bug reviewer. Block whenever `bugs` is non-empty at MEDIUM+ |
| Escalate list (schema/API, gating logic, infra, auth, billing, policy file) | C2 | Hard blocks §7.3 | Proposed: add "gating logic (flags, permission checks, routing)", "infra (Terraform/Helm/K8s)", and "review-policy files" to §7.3 |
| Policy file can be tampered with in the PR under review | C8, C2 | Review layers, security | Proposed: load review config (`review.md`, flow globs, risk weights) from the base branch, never the PR head. Escalate any PR that edits it |
| Shadow mode ("Would Approve"/Preview) before real approvals | B27, C2 | Risk scoring rollout | Proposed: run §7.2 scoring in shadow mode, logging verdicts with no approvals, for an initial period. Compare against human decisions |
| Diff-hash verdict caching on rebase | C2 | Run-to-run stability, cost | Proposed: cache review output by (base tree, diff hash) so a rebase with no change does not re-review |
| Verdict written as a check run with a reason; CSV dashboard | B27, C2 | Audit trail §7.2 | Proposed: persist every score and decision with its reason as JSON next to the `bugs` output |
| Neutral by default; blocking only after 1–2 weeks of reliable output and only for high-risk paths | B17, C8 | Review layers | Proposed: our check posts `neutral` at first. Promote to `failure` only for hard-block areas |
| Required checks: missing result blocks forever; scoped checks must report `skipped` | C8 | PR commenter, CI integration | Proposed: always post a check result, using `skipped` when out of scope and `neutral` on budget stop or error |
| Budget per run checked between turns; budget stop → neutral; `maxRuns` per PR | C5 | Cost, review agent | Proposed: add per-run dollar and turn caps and per-PR run caps. Stopping on budget never fails the check |
| Mine a year of senior human review comments into custom checks | B24 | Review layers, living wiki | Proposed: mine past PR comments to seed `review.md` rules and tuning labels |
| Markdown-defined custom checks with model, tools, globs, input mode | B17 | Review layers, model routing | Proposed: support per-rule markdown checks with model choice. Use the cheap model for mechanical rules and the frontier model for correctness |
| Fix It For Me: fix PR into the feature branch only, self-heal on CI failure; never main | B7, B9 | Act vs comment, autonomous testing | Proposed: v1 only comments with `diff_fix_suggestion`. A later opt-in fixer must target the feature branch only, with a capped retry loop |
| Autoloop up to 5 cycles; worktree snapshot isolation | B21 | Review layer 2, sandbox | Proposed: local pre-push review runs in a worktree snapshot with a cap of 5 fix cycles |
| VM sandboxes run the full stack and attach screenshots or recordings as evidence | B23 | Autonomous testing §2.5, evidence §8 | Proposed: confirms the §2.5 design. Attach browser-agent media to the review as §8 evidence |
| Effective precision = comments leading to a change ÷ total; thumbs-down rate (2.2%) | C9, B6 | Measurement §9 | Proposed: add a thumbs-down rate next to the addressed rate. Both are cheap outcome proxies |
| Opus-only v2: +40% recall but +40% latency (262 s) [SR] | B5 | Cost and latency | Proposed: set a latency budget for Submit review (for example under 5 min) and record latency per run in §9 |

---

## 5. Open questions and gaps

1. **AST vs agentic context.** Macroscope dropped AST walkers and reports better results [SR]. Our §3 relies on Python `ast`. Do the deterministic contract checks still add recall on our dataset beyond what an agent finds by exploring? Needs an ablation.
2. **Severity in the output contract.** §4 forbids new keys. Decide whether severity becomes a key (spec change) or a sidecar record.
3. **Validation-stage design.** No post discloses the validator's prompt, model, or evidence threshold. Only heuristics are known (rejecting hedged language).
4. **Approvability thresholds.** Pydantic's 91% auto-approve rate [SR] has no published false-approve rate. No post reports how many auto-approved PRs later caused incidents.
5. **Recall per detection mode** is never published. Only precision (85% / 97%) is.
6. **Benchmark conflicts.** B25 and C7 show different top models and scores for the same benchmark, and B25's signal/noise column is undefined. Do not cite either as ground truth.
7. **Vendor tuned on its own benchmark** (B2 says so explicitly). The only third-party precision figure is from the Martian leaderboard, quoted second-hand.
8. **Several pages have dates that conflict with their content**, which shows they were edited after publication. Treat figures as of fetch date 2026-11-09.
9. **Fix success rate** for Fix It For Me and Autoloop is unpublished.
10. **Dedup and suppression method** is called the main signal/noise lever (C6) but never described.
11. **Gate tampering.** Macroscope reads policy from the PR head and patches this with `neverApprove`. Our system should decide base-branch-only config loading up front.
12. **Living-wiki analogue.** Macroscope has no explicit incident or revert memory. Its closest equivalents are git history search and mined human comments. Our wiki is unvalidated against any vendor data.
13. **Cost per review baseline.** Macroscope's average is 55–76¢ [SR]. We have no estimate of our own cost per review yet (§9).
14. Specialized model (C6): base model, data, and method are undisclosed, so the 25% token-reduction claim cannot be reproduced.
