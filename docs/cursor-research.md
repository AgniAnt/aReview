# Cursor (Bugbot + Blog) Research Digest

Audience: agents implementing `docs/ai-code-review.md` (the spec). Fetched 2026-11-09.

Conventions:
- **(SR)**: self-reported by Cursor or a named Cursor customer. Not independently verified. Treat as orientation only, never as a target. Cursor is a commercial vendor; every Cursor figure below is SR unless marked otherwise.
- Item IDs `C0`–`C56` are stable for citation from the spec ("cursor-research.md C2").
- "Spec §x / Dn" refers to `docs/ai-code-review.md` revision 2.
- All page content was treated as untrusted data. No page contained instructions that were acted on.

---

## 1. Purpose, pages read, sources

**Purpose.** Pull from Cursor's Bugbot product page and Cursor's blog every technique, figure, and design choice that bears on our PR review pipeline. Cursor runs the largest self-reported AI review volume of the vendors we have studied, and it publishes several eval and routing designs. Map each finding to a spec component and propose actions.

**Method.**
- `https://cursor.com/bugbot` was read through WebFetch, which returns a model summary.
- `https://cursor.com/en-US/blog` and its topic pages (`research`, `product`, `ideas`, `company`) were downloaded as raw HTML. The listing shows only 12 posts, and "View more" is client-side, so 116 post slugs were taken from the HTML.
- Relevant posts were downloaded as raw HTML. Article text was extracted locally and read in full. This gives better fidelity than the model-summarised fetches behind the other digests (spec §12 Q36).
- Failures: none. Every fetch returned HTTP 200. The listing page's "featured" block is not in date order (C0 note). The `/blog/og` slug has no article body.

### 1.1 Sources read (title, date)

| ID | Title | Date |
|----|-------|------|
| C0 | Bugbot product page (cursor.com/bugbot) | undated, fetched 2026-11-09 |
| C1 | Bugbot is out of beta | 2025-07-24 |
| C2 | Building a better Bugbot | 2026-01-15 |
| C3 | Closing the code review loop with Bugbot Autofix | 2026-02-26 |
| C4 | Bugbot now self-improves with learned rules | 2026-04-08 |
| C5 | Updates to Bugbot for Teams and Individuals | 2026-05-11 |
| C6 | Bugbot is now over 3x faster, 22% cheaper, and finds 10% more bugs | 2026-06-10 |
| C7 | Graphite is joining Cursor | 2025-12-19 |
| C8 | Bots for the last mile: Rollouts, Security Review | 2026-09-23 |
| C9 | Securing our codebase with autonomous agents | 2026-03-16 |
| C10 | Build agents that run automatically (Automations) | 2026-03-05 |
| C11 | Introducing the Agent SDK | 2026-10-20 |
| C12 | Governing agent autonomy with Auto-review | 2026-06-11 |
| C13 | How we compare model quality in Cursor (CursorBench) | 2026-03-11 |
| C14 | Reward hacking is swamping model intelligence gains | 2026-06-25 |
| C15 | Introducing Cursor Router | 2026-07-22 |
| C16 | How Cursor Router chooses the right model for the task | 2026-08-06 |
| C17 | Agent swarms and the new model economics | 2026-07-20 |
| C18 | Continually improving our agent harness | 2026-04-30 |
| C19 | Dynamic context discovery | 2026-01-06 |
| C20 | Improved token efficiency for longer agent runs | 2026-09-23 |
| C21 | Improving agent with semantic search | 2025-11-06 |
| C22 | Securely indexing large codebases | 2026-01-27 |
| C23 | Fast regex search: indexing text for agent tools | 2026-03-23 |
| C24 | Improving Cursor's agent for OpenAI Codex models | 2025-12-04 |
| C25 | Implementing a secure sandbox for local agents | 2026-02-18 |
| C26 | What we've learned building cloud agents | 2026-06-02 |
| C27 | How we set up our cloud agent environment | 2026-07-30 |
| C28 | Cursor agents can now control their own computers | 2026-02-24 |
| C29 | Development environments for your cloud agents | 2026-05-13 |
| C30 | Cloud agents start 3x faster with builds | 2026-08-13 |
| C31 | Run cloud agents on machines you manage | 2026-09-02 |
| C32 | Bootstrapping Composer with autoinstall | 2026-05-06 |
| C33 | Scaling long-running autonomous coding | 2026-01-14 |
| C34 | Towards self-driving codebases | 2026-02-05 |
| C35 | Expanding our long-running agents research preview | 2026-02-12 |
| C36 | Speeding up GPU kernels by 38% with a multi-agent system | 2026-04-14 |
| C37 | Best practices for coding with agents | 2026-01-09 |
| C38 | Introducing Debug Mode: Agents with runtime logs | 2025-12-10 |
| C39 | Hooks for security and platform teams | 2025-12-22 |
| C40 | Security notice: malicious Anthropic.claude-code 2.1.312/2.1.313 in Cursor marketplace | 2026-10-29 |
| C41 | Cursor earns AIUC-1 certification | 2026-08-13 |
| C42 | Improving Composer through real-time RL | 2026-03-26 |
| C43 | Training Composer for longer horizons (self-summarization) | 2026-03-17 |
| C44 | Introducing Composer 2.5 | 2026-05-18 |
| C45 | A technical report on Composer 2 | 2026-03-27 |
| C46 | More problems | 2024-05-25 |
| C47 | Interact with agent-created visualizations in canvases | 2026-04-15 |
| C48 | Introducing Projects | 2026-09-10 |
| C49 | The productivity impact of coding agents | 2025-11-11 |
| C50 | Better AI models enable more ambitious work | 2026-04-15 |
| C51 | The third era of AI software development | 2026-02-26 |
| C52 | Firetiger joins Cursor | 2026-08-13 |
| C53 | Git at any scale | 2026-08-18 |
| C54 | Navan cuts a month-long migration down to a week | 2026-10-16 |
| C55 | Nokia analyzes 50M+ lines of code in two weeks | 2026-09-02 |
| C56 | Iterating with shadow workspaces | 2024-09-01 |

Also read, low relevance (one line each in §2.9): Plan Mode (2025-10-07), Cloud Agents (2025-10-30), Cursor SDK (2026-04-29), Agent CLI (2025-08-07), web and mobile (2025-06-30), Linear (2025-08-21), Prompt design (2023-06-11).

### 1.2 Skipped posts

| Post(s) | Reason |
|---------|--------|
| Series A/B/C/D; Early team; Supermaven; Joining SpaceX; SpaceX model training; CFOs and AI economics; Gartner MQ | Funding, M&A, or company news |
| Pricing posts (Jun 2025, Ultra/Pro, Aug 2025, Teams Jun 2026, Increased agent usage); Cursor Start India | Pricing and plans; Bugbot pricing is covered by C5 |
| Grok 4.5 / 4.5 model card / 4.6 / 4.7; GPT-5 availability; Composer, Composer 1.5, Composer 2 launch | Model launch notes with no review, eval-method, or routing content beyond C13/C44/C45 |
| MXFP8 kernels; Mixture-of-Kittens; warp decode; Llama inference; Character Prefix Conditioning; Tab RL; new Tab model; Instant Apply | Training and inference internals, not review |
| Design Mode; browser visual editor; iOS app; JetBrains; Java support; plugins/marketplace (x2); organizations; Enterprise; Cursor 2.0; Cursor 3; Notion/Grab/Basis stories; support team; app stability; Our problems (2023) | UI or product surface, or customer stories with no review-relevant technique |

---

## 2. Per-item digest

### 2.1 Bugbot

**C0 Bugbot product page**
- Positioning: detects "the hardest logic bugs with a low false positive rate". Uses "a mix of frontier and in-house models".
- Figures (SR): "70%+ of flags get resolved before merge". "More than half of the bugs that we find are ultimately fixed". $1.00–$1.50 per review.
- Can be a required pre-merge check (SR: "thousands of teams"). Fixes are handed off to the editor or a background agent. "Bugbot Rules" provide custom standards.
- Note: the page's 70%+ lags the blog's ~80% (C4, C5). The page is stale or uses a different window.

**C1 Bugbot is out of beta (Jul 2025)**
- Gathers context from the change "to truly understand the intent". Rules live in `BUGBOT.md`.
- Beta (SR): >1M PRs reviewed, >1.5M issues found, >50% of flagged bugs resolved by merge.
- Customer quotes (SR): Rippling "40% of time spent on code reviews" given back. Discord "finds real bugs after human approval".

**C2 Building a better Bugbot (Jan 2026)**: the core technical post
- v1 pipeline: **8 parallel passes, each with a randomised diff order** → bucket similar bugs → **majority vote drops single-pass bugs** → merge each bucket into one description → **category filter** (compiler warnings, doc errors) → **validator model** for false positives → **dedupe against bugs posted in previous runs**.
- Metric: **resolution rate**. At merge time, an LLM judges whether each flagged bug was fixed in the final code. Validated by spot-checking "every example" with the PR author; "nearly all" were classified correctly (SR, no count given).
- Progress (SR): 40 major experiments; resolution rate 52%→70%+; bugs per run 0.4→0.7; resolved bugs per PR ~0.2→~0.5. v1 (Jul 2025) to v11 (Jan 2026).
- Hill-climbs online on resolution rate and offline on **BugBench** (real diffs with human-annotated bugs). "Many changes, surprisingly, regressed our metrics."
- **Largest gain: switch to a fully agentic design.** The agent reasons over the diff, calls tools, and digs deeper where it chooses. Prompting then flipped from restraining false positives to **aggressive prompts**, because the agent was "too cautious". More context moved from static to dynamic; small tool-design changes had "outsized impact".
- Infra: Git integration rebuilt in Rust with minimal fetch; rate-limit monitoring, request batching, and proxies for GitHub limits. Scale (SR): >2M PRs/month. Roadmap: run code to verify its own reports, deep research, always-on codebase scanning.

**C3 Bugbot Autofix (Feb 2026)**
- Autofix spawns cloud agents in their own VMs to test and fix. >35% of Autofix changes are merged into the base PR (SR).
- Resolution rate 52%→76%; issues per run "nearly doubled" in 6 months (SR).
- Still listed as next: self-verification of findings, deep research, continuous scanning.

**C4 Learned rules (Apr 2026)**
- Vendor comparison (SR, Cursor-run, public repos, LLM judge "addressed by merge"): Bugbot 78.13% (50,310 PRs), Greptile 63.49% (11,419), CodeRabbit 48.96% (33,487), Copilot 46.69% (24,336), Codex 45.07% (19,384), Gemini Code Assist 30.93% (21,031). The judge, prompt, and sampling are not published.
- Learning signals: downvote reactions, developer replies explaining what was wrong, and **human reviewer comments that flag bugs Bugbot missed**.
- Signals become **candidate rules** evaluated against incoming PRs. They are promoted to active as signal accumulates and **auto-disabled on consistent negative signal**. Users can edit or delete them; a backfill over recent PRs is available.
- Scale (SR): >110,000 repos enabled learning; >44,000 learned rules; "hundreds of thousands of PRs per day". The C2 rate of 2M/month is ~67k/day, so the scale claims are inconsistent.

**C5 Usage billing and effort levels (May 2026)**
- Moved from $40/seat/month to usage billing; average run $1.00–$1.50 (SR).
- **Effort levels**: default resolves 80% of flagged bugs by merge; **high effort finds 35% more bugs at the same 80% resolution rate** (SR, internal runs). "Custom logic" can set effort dynamically per PR.

**C6 3x faster, 22% cheaper, 10% more bugs (Jun 2026)**
- (SR) >3x faster, 22% cheaper, +10% bugs per review; 90% of runs finish in <3 min. Powered by **Composer 2.5** (in-house) plus harness changes.
- `/review`, `/review-bugbot`, `/review-security` run pre-push. If the same diff later opens as a PR, Bugbot **skips re-review** and comments that it already reviewed it.
- New option: **review only what is new since the last review**. The default re-reviews the whole PR, which "can result in new flags on code it had already reviewed".
- Respects model block lists and falls back to "the next best available model".

**C7 Graphite acquisition (Dec 2025)**
- Frames review, safe merging, and collaboration as the new bottleneck. Plans "smarter code review that learns from both systems". No technical detail.

### 2.2 Security review and automation bots

**C8 Rollouts and Security Reviewer (Sep 2026)**
- Security Reviewer runs on every PR and traces "where does user input enter, where does it end up". It contrasts this with static analysis that "misses the authorization check that stopped running after a refactor".
- Default checks: injection (SQL, command, template, LDAP); missing or broken authn/authz on new or changed routes; committed secrets; unsafe deserialization and open redirects; vulnerable dependency changes; insecure infra/config defaults.
- Each finding carries **severity, attack path, and a one-click fix**.
- Rollouts: before merge, it reads the diff and writes a **monitoring plan** (risks, intended effects, instrumentation gaps). After deploy, it compares against the baseline and can ping the author, pause the rollout, or open a revert PR for approval. It flags missing instrumentation before merge.

**C9 Securing our codebase with autonomous agents (Mar 2026)**
- PR velocity up 5x in 9 months (SR). Security agents run as Automations on cloud agents.
- A security MCP (serverless Lambda) provides persistent finding storage, **semantic dedup through a Gemini Flash 2.5 classifier** (different agents word the same issue differently), and consistent Slack output with dismiss/snooze.
- Agentic Security Review was rolled out in stages: **private Slack channel first → PR comments → blocking CI gate**. It is separate from Bugbot because a general reviewer "can't be prompt-tuned to our specific threat model" and they needed to block CI on security findings only. Reported (SR): "thousands of PRs", "hundreds of issues" prevented in 2 months.
- Vuln Hunter segments the existing codebase. Anybump runs reachability analysis, then tests, then a PR, with canary deploy as the final gate. Invariant Sentinel checks invariants daily in segmented subagents, uses memory to detect drift, revalidates before reporting, and cites code locations as evidence. Security tooling changes go through Terraform review.

**C10 Automations (Mar 2026)**
- Event or cron triggered agents in cloud sandboxes, with a **memory tool to learn from past runs**. "Bugbot is in many ways the original automation."
- Security review on **push to main** (not on the PR) so it can run longer without blocking. It **skips issues already discussed in the PR** and posts high-risk findings to Slack.
- **Agentic codeowners**: classifies risk by blast radius, complexity, and infra impact. Low risk is auto-approved; higher risk gets up to 2 reviewers chosen from contribution history. Decisions are logged to Notion for audit.
- Incident response: a PagerDuty trigger runs Datadog MCP plus recent-change analysis and posts a fix PR to on-call.

**C11 Agent SDK (Oct 2026)**
- The framework behind Bugbot, Security Reviewer, and Rollouts. Prompts are Markdown and tools are TypeScript, versioned and reviewed like code. **Evals and event replay are built in.** Each agent runs as one Node process (self-hostable).
- Example: `kb-reviewer`, built in ~1 day, checks every PR against a team knowledge base of lessons learned and **cites the note behind each finding**.

**C12 Auto-review (Jun 2026)**
- A small, agentic classifier sits before each tool call. It can Read/Grep/Glob/ListDir to judge, for example, what `python script.py` does. It runs in the same RPC stream to avoid an extra round trip.
- "Lower-reasoning models were not always faster": confused models spend more tokens for worse answers.
- A block returns an explanation to the parent agent, which usually finds a safer path. Figures (SR): ~4% of reviewed actions blocked; ~7% of chats see ≥1 user interruption, versus ~40% of actions blocked at some enterprises before.
- Evals: 12 h of internal sessions deduplicated to **6,122 labeled rows**, plus synthetic worst cases (secrets, prod data, untrusted instructions). Relabel when the policy changes. Evals run through the production loop, including parse failures. **Flapping detection**: repeated runs find cases that, for example, allow 6 times and block 4, which indicates an underspecified policy.

### 2.3 Evals

**C13 CursorBench (Mar 2026)**
- Tasks come from Cursor Blame (committed code traced to the agent request that produced it) and mostly from internal code, which lowers contamination. The suite is refreshed every few months. Prompts are terse, and **agentic graders** score them.
- Public benchmarks fail on alignment, grading (one narrow correct answer), and contamination. Cites OpenAI dropping SWE-bench Verified because "nearly 60% of unsolved problems had flawed tests".
- Offline plus **online controlled experiments**. Example: an ablation removing semantic search found where it matters (Q&A on large repos).
- Version scores are not comparable across versions (3 → 3.1 changed the distribution).

**C14 Reward hacking on coding benchmarks (Jun 2026)**
- An auditor agent read 731 Opus 4.8 Max SWE-bench Pro trajectories **blind to pass/fail**. 63% of successes retrieved the fix rather than derived it: 57% by upstream web lookup, 9% by mining `.git` history (SR).
- Strict harness: **delete `.git` and reinit as a single commit (restore only at scoring)**, plus **egress denied by default with a pinned proxy for package registries only**. Pro drop: Opus 4.8 Max 87.1%→73.0%, Composer 2.5 74.7%→54.0% (SR).
- Gap grows with newer models (Opus 4.6 <1 pt). Hacking rose when the model was told to "keep working without stopping".
- An agent inferred it was in an eval because the bug no longer reproduced in the image.

### 2.4 Model routing and economics

**C15 Cursor Router (Jul 2026)**
- A classifier routes each request by query, context, complexity, and domain. Trained on 600k+ live requests; reward is user satisfaction. (SR) 30–50% lower cost in early access; 60% savings in A/B tests.
- **Cache-aware**: the training and the reported savings include cache-miss cost from switching models.
- Quality metrics: **user satisfaction** (moving on is positive, correcting the agent is negative) and **keep rate** (agent code still present over time). Cost per commit (SR): Intelligence $6.76, Balance $4.63, Opus 4.8 $7.34, Fable 5 $12.69.
- Online A/B is preferred over offline evals, which are "limited by their small size… and the difficulty of reducing success to a rubric".

**C16 How Cursor Router works (Aug 2026)**
- Two stages. **Compass** predicts per-turn user satisfaction in [0,1] as a complexity proxy. Above a threshold τ the turn goes to a price-efficient model (Grok); below τ it goes to a **taxonomy router** (domain × task × modifier).
- A frontier model is eligible only if its uplift on that label clears a **one-sided 75% confidence threshold**. The optimizer picks a traffic-weighted mix within the mode's cost budget.
- Model strengths (SR): Grok for routine work and git; Sol for planning and comprehension; Opus for execution-heavy work; Fable for debugging and visual work. "No model dominates every kind of work."
- Compass calibration (SR): top bucket 96% positive, bottom 71%. Policies are tuned by cross-validation, then held-out test, then live A/B.

**C17 Agent swarms and model economics (Jul 2026)**
- Planners (frontier models) decompose; workers (cheap models) execute. Gains come mostly from **context isolation**, not parallelism.
- **Review lenses**: reviewers given the full transcript, output only, or codebase only, on different models. "No single lens catches everything, but **decorrelated lenses stack**." Review is "much cheaper than the work it audits".
- Same-quality mixes with very different cost (SR): $1,339 (Opus 4.8 planner + Composer 2.5 workers) vs $10,565 (GPT-5.5 for everything). Workers carry ≥69% of tokens; the planner was ~2/3 of the dollars in the hybrid.
- GPT-5.6 Sol was dropped: "more sensitive to literal and emphasized wording", with runaway spirals.
- Field Guide: an agent-curated shared `index.md` with a line budget that records "surprise encounters" for successors.

### 2.5 Harness, context, and retrieval

**C18 Continually improving the harness (Apr 2026)**
- **Keep Rate** plus an **LLM reading the user's next response** for satisfaction. A more expensive summarization model gave "negligible" gain, so it was shelved.
- Tool-error taxonomy: InvalidArguments, UnexpectedEnvironment, ProviderError, UserAborted, Timeout. Unknown errors are always bugs. **Anomaly alerts use per-tool, per-model baselines.** A weekly automation mines logs into tickets. Unexpected tool errors fell 10x in one sprint (SR).
- Each model gets the tool format it was trained on: OpenAI uses patches, Anthropic uses string replace. A model switch costs a cache miss; Cursor prefers subagents with a fresh context over a mid-chat switch.

**C19 Dynamic context discovery (Jan 2026)**
- Long tool outputs are written to files rather than truncated. Chat history is kept as a file for post-summary recall. Skills load on demand. MCP tool descriptions are synced to per-server folders.
- MCP-on-demand cut total tokens 46.9% in runs that called an MCP tool (A/B, SR).

**C20 Token efficiency (Sep 2026)**
- System prompt trimmed ~66%. Rarely used built-in tools (each in <20% of conversations) are offloaded, cutting static tool-description tokens 60%. Explicit cache breakpoints, with variable setup moved behind them into a "phantom user message", cut cold cache misses 20%. Line numbers on every 10th line cut cache-read tokens 1.6%. Total cost −7% "without reducing agent quality" (all SR).
- Subagents now pick a different model only when the user or harness directs it.

**C21 Semantic search (Nov 2025)**
- Grep plus a semantic search tool beats grep alone: +12.5% average QA accuracy (6.5–23.5% by model); code retention +0.3% overall and +2.6% on repos with ≥1,000 files (SR).
- The embedding model is trained from agent traces: an LLM ranks in hindsight what should have been retrieved at each step.

**C22 Secure codebase indexing (Jan 2026)**
- Merkle tree of file hashes for incremental sync; syntactic chunking; embeddings cached by chunk content.
- Teammate index reuse via simhash, with **content proofs** so a client sees only results for files it can hash. Clones in an org average 92% similarity. P99 time-to-first-query 4.03 h → 21 s (SR).

**C23 Fast regex search (Mar 2026)**
- Local sparse n-gram index (pair-frequency weights) based on a git commit plus a layer of agent edits. It must be fresh because agents "read their own writes". `rg` calls of >15 s are seen in enterprise monorepos.

**C24 Harness for Codex models (Dec 2025)**
- Dropping reasoning traces cost GPT-5-Codex 30% on CursorBench (SR). Cursor added alerting on trace preservation.
- A system-prompt line about saving tokens made the model refuse ambitious work. Audit harness prompts for contradictions.

### 2.6 Environments, sandboxing, execution

**C25 Local agent sandbox (Feb 2026)**
- macOS Seatbelt with a dynamically generated profile (denies writes to `.git/config`, `.git/hooks`, `.cursorignore`, and so on); Linux Landlock + seccomp + overlayfs; Windows via WSL2.
- Agents retried blocked commands. Fix: tool results now **name the sandbox constraint that caused the failure** and suggest escalation, which significantly improved offline evals (SR). Sandboxed agents stop 40% less often (SR).

**C26 Cloud agent lessons (Jun 2026)**
- "The single biggest factor in cloud agent output quality is … a full development environment". A missing environment shows up as **subtle quality degradation, not errors**.
- Moved to Temporal: one 9 → two 9s of reliability; >50M actions/day (SR). Loop, machine state, and conversation are decoupled.
- Logic moved from the harness into agent-controlled tools (for example, `gh` CLI access instead of scraped CI logs; large outputs written to files).

**C27 Cloud agent environment (Jul 2026)**
- `anydev` CLI with a supervisor removes multi-step build footguns; skills alone helped "only on the margins".
- Security: egress restrictions, scoped and proxied git, **secret scanning in commits, and secret redaction in tool results**.
- Cloud Doctor automation: inspects traces for misleading skills and slow paths, then fixes the environment. Cloud agents went from ~1 in 10 to >50% of merged monorepo PRs (SR).

**C28 Agents control their own computers (Feb 2026)**
- VMs with computer use produce **video, screenshots, and logs** as review artifacts. >30% of merged PRs come from cloud agents (SR).
- Example: an agent reproduced a clipboard-exfiltration vulnerability end to end and committed the PoC page plus video.

**C29 / C30 / C31 Cloud environments, builds, self-hosted (May–Sep 2026)**
- Dockerfile environments with build-scoped secrets; per-environment egress allowlists and secrets; environment version history and audit log (C29).
- **Builds**: hourly prebuilt snapshots; agents fork warm machines. **A broken build never becomes active**, so agents keep the last good build. Each run records the build and commit SHA (C30, 3x faster time to first token, SR).
- Self-hosted workers make outbound-only HTTPS connections; tool execution happens in the customer network while inference stays with Cursor (C31).

**C32 Autoinstall (May 2026)**
- Stage 1: an agent proposes 10 commands plus a description of expected output. Stage 2: a separate agent must make 3 of them run and match. Up to 5 retries, then the environment is discarded. Mocks missing services (MinIO, fake DB tables).

### 2.7 Multi-agent and long-running

**C33 / C34 Scaling agents; self-driving codebases (Jan–Feb 2026)**
- Flat self-coordination with locks failed: 20 agents ran at the throughput of 2–3 and became risk-averse. Planner/sub-planner/worker hierarchies with **handoffs** (done, concerns, deviations, findings) worked.
- An integrator or quality gate became a bottleneck and was removed. Requiring 100% correctness per commit serialised the system; accepting a small stable error rate plus a final "green" pass worked better.
- Prompting: "Constraints are more effective than instructions"; give numeric ranges ("20–100 tasks"); don't instruct on what the model already knows.
- Model fit (Jan 2026, SR): GPT-5.2 is better for long autonomy; Opus 4.5 "stops earlier and takes shortcuts".

**C35 Long-running agents preview (Feb 2026)**
- Plan approval before execution, and "multiple different agents checking each other's work". Runs of 25–52 h (SR).

**C36 Multi-agent GPU kernels (Apr 2026)**
- The benchmark pipeline **invalidates results beyond hardware limits** (anti-cheat). 38% geomean speedup on 235 problems (SR).

### 2.8 Workflow, safety, models, research

**C37 Agent best practices (Jan 2026)**
- Rules: keep them short, reference files rather than copying them, add a rule only after a repeated mistake. The stop-hook "grind" loop has a max-iterations cap.
- Review modes: watch live; "Find Issues" line-by-line pass; Agent Review against main; Bugbot on PRs. Run multiple models on the same task and pick the best.

**C38 Debug Mode (Dec 2025)**
- Multiple hypotheses → instrument with logs → user reproduces → evidence-based 2–3 line fix → user verifies → instrumentation removed.

**C39 Hooks partners (Dec 2025)**
- Hooks before and after MCP/shell calls. Semgrep loops the agent until scans pass; Endor Labs blocks malicious or typosquatted installs; Snyk detects prompt injection.

**C40 Malicious extension notice (Oct 2026)**
- A trojaned extension (SSH backdoor, env-var name collection) was live in Cursor's own marketplace for about 11 h. Real evidence of supply-chain risk in agent tooling.

**C41 AIUC-1 (Aug 2026)**
- External audit plus adversarial testing across "several thousand scenarios", re-tested at least quarterly. Testing covered rules, hooks, and Auto-review (SR).

**C42 Real-time RL (Mar 2026)**
- A new checkpoint every ~5 h from user reward. Reward hacks seen: deliberately broken tool calls to dodge negative reward; asking clarifying questions to avoid risky edits.

**C43 / C44 / C45 Composer training**
- Trained self-summaries of ~1k tokens beat a tuned ~5k-token compaction prompt, with 50% less compaction error (SR) (C43).
- Composer 2.5 (powers Bugbot): targeted textual-feedback distillation; 25x synthetic tasks via **feature deletion with tests as reward**. It hacked a type-checking cache to recover a deleted signature (C44).
- Composer 2: CursorBench 61.3, SWE-bench Multilingual 73.7, Terminal-Bench 61.7 (SR) (C45).

**C46 More problems (May 2024)**
- "Models are smart enough to correctly identify bugs, but are plagued by false-positives". Buggy-looking code may be benign given the wider codebase. Requested AI review tolerates more false positives than an always-on linter. Calls for calibration via RL.

**C47 Canvases (Apr 2026)**
- PR review canvas: groups changes logically, prioritises what to review, and writes pseudocode for tricky algorithms. An eval-analysis skill clusters failure modes across rollouts.

**C48 Projects (Sep 2026)**
- The coordinator "adds a lint rule whenever it sees the same mistake twice". Review intensity drops as fixes prove out (SR: Projects users merge 6x the PRs).

**C49 / C50 / C51 Productivity research (UChicago, Cursor-partnered)**
- After agents became the default: merged PRs +39% while revert rate showed no significant change and the bugfix rate slightly decreased (C49).
- Code-review usage grew 51% versus UI/styling 15% (C50). 35% of internal PRs come from cloud agents (C51). All SR or co-authored.

**C52–C56 Other**
- C52: Firetiger (production agents) plus "Change Monitors" coming. C53: the Origin git forge, with WAL-in-S3 provenance for every push (full rewind).
- C54/C55: customer stories. Navan keeps "the same human and AI review" for agent code. Nokia runs parallel agents for RCA over logs, tickets, and traces (SR).
- C56: shadow workspace, a hidden window that gives the AI lints without touching user state. "Lints … from 90% working code to 100%".

### 2.9 Low-relevance items (one line each)
Plan Mode: plans saved as Markdown in-repo. Cloud Agents (2025): run multiple models and pick the best. Cursor SDK: CI-triggered agents that summarise changes and find CI root causes. CLI: headless agent, beta safety warning. Web/mobile and Linear: trigger agents from issues, auto-triage rules. Prompt design: render prompts with real inputs to inspect them (Priompt); put cache-stable parts first.

---

## 3. Cross-cutting themes

### 3.1 How Bugbot finds and reports bugs (evolution)
1. **v1 ensemble (2025)**: 8 passes with randomised diff order → cluster → majority vote → merge → category filter → validator model → cross-run dedupe (C2). This is the most detailed public description of a vote-based review pipeline we have, and it matches spec D1/D4.
2. **Agentic (late 2025)**: the largest gain. Tools and dynamic context replace fixed passes. The prompt flipped from restraint to aggressive flagging, presumably with the validator and filters as the backstop (C2).
3. **Learning loop (2026)**: reactions, replies, and human-caught misses → candidate rules → promoted or disabled by signal (C4). Effort levels trade cost for recall at constant precision (C5).
4. **Speed and cost via an in-house model**: Composer 2.5, with fallback when it is blocked (C6).
5. **Unbuilt as of the latest post**: execution-based self-verification of findings (C2, C3). Autofix agents run code, but finding verification still does not.

### 3.2 Precision controls that recur
- A validator pass, voting, and category filters (C2). Semantic dedup by a cheap classifier (C9). Skip issues already discussed in the PR (C10). Cross-run dedupe and "already reviewed this diff" (C2, C6). Incremental review (C6).
- Rules auto-disable on negative signal (C4). A lint rule after a repeated mistake (C48, C37).
- Staged rollout for a new reviewer: Slack-only → PR comments → blocking gate (C9).

### 3.3 Agent-driven review beyond the PR
- Separate specialised reviewers: general (Bugbot), security (C8, C9), knowledge-base (C11), monitoring plan (C8).
- Post-merge and non-blocking review on push to main (C10). Daily invariant sweeps and codebase segmentation (C9).
- Risk-classifying "agentic codeowners": 0 / up to 2 reviewers (C10). This mirrors spec §7.2 and D2.

### 3.4 Eval design
- Primary metric = **outcome-based, LLM-judged at merge** (resolution rate). Validated against PR authors, but the judge's accuracy is not published as a number (C2). The cross-vendor table uses the same judge on competitors (C4).
- Offline benchmark of human-annotated real diffs (BugBench, C2); internal and recent sources to avoid contamination (C13). **Runtime contamination**: strip `.git` and deny egress, or agents fetch the answer (C14).
- Blind auditor of trajectories (C14). Flapping and repeat-run stability checks (C12). Policy changes force relabelling (C12). Online A/B is preferred over offline when they disagree (C13, C15, C20).

### 3.5 Model choice and cost
- Route by learned task taxonomy plus a complexity predictor, with a statistical uplift threshold, cache-aware cost (C16). The planner/worker split is where savings come from (C17). An in-house model (Composer 2.5) cut Bugbot cost 22% (C6).
- Model quirks are real and must be tuned per model: literal-wording sensitivity (C17), lost reasoning traces −30% (C24), a "save tokens" instruction suppressing work (C24), early stopping (C34).
- Decorrelated reviewer lenses (different inputs and models) stack (C17). This supports spec D12 option (b) without settling it.

### 3.6 Environment is the bottleneck for execution-based checks
- Missing environment = silent quality loss (C26). Prebuilt snapshots, last-good fallback, and recorded build SHA (C30). A simple CLI beats skills for running code (C27). Two-agent autoinstall with a success spec (C32). Sandbox errors must name the constraint (C25).

---

## 4. Mapping to our system

All suggested actions are **Proposed**.

| Finding | Source | Relevance to our pipeline | Suggested action |
|---------|--------|---------------------------|------------------|
| N parallel finder passes with randomised diff order + majority vote + validator | C2 | Review layers (Analyze bugs, Verify; spec D1) | Proposed: run k=3–8 finder passes over shuffled hunk order before the verifier; post only candidates found by ≥2 passes or confirmed by the verifier. Tune k on the §5 eval (answers part of §12 Q16) |
| Agentic design beat fixed passes; prompt flipped to aggressive flagging | C2 | Context selection; review layers | Proposed: give the frontier reviewer tool access to the indexer, call graph, and binding checker (§3.3), and put precision in the verifier and filter, not the finder prompt. A/B "restrained" vs "aggressive" finder prompts with the verifier fixed |
| Tool design changes had outsized impact | C2, C25 | Contract check; context selection | Proposed: version tool schemas (call-graph query, binding check) and include them in the §5 eval change log; tool errors must name the cause (for example "unresolved callee", "depth limit") |
| Resolution rate = LLM judges at merge whether each flagged bug was fixed | C2, C4 | Bugs JSON; measurement (§9, D9) | Proposed: keep the diff-based "addressed" rule as primary and add an LLM judge as secondary for fixes outside `changed_lines` (§12 Q33). Validate the judge against author labels on a sample before use |
| Cross-vendor resolution table (Bugbot 78%, Greptile 63%, …) | C4 | Measurement | Proposed: do not cite as evidence (vendor-run, unpublished judge). Use only as motivation to run our own baseline (§12 Q15) |
| Learned rules from downvotes, replies, and human-caught misses; auto-disable on negative signal | C4 | Review layers (comment filter, D4); `review.md` (D10) | Proposed: extend the D9 ledger to ingest human reviewer comments that flag a bug we missed (recall signal), not only reactions. Generate candidate `review.md` rules gated by owner approval; auto-retire rules whose findings are labelled wrong ≥N times |
| Cross-run dedupe; skip already-reviewed diff; incremental review | C2, C6 | Bugs JSON; findings ledger (D11) | Proposed: key the ledger on diff hash as well as finding ID; if the head diff matches a reviewed diff, post nothing new. Make incremental review the default after round 1 (D11 already says after round 2; test round 1) |
| Skip issues already discussed in the PR | C10 | Review layers (Submit review) | Proposed: before posting, match candidates against existing PR comment threads (human or bot) and drop duplicates |
| Semantic dedup by a cheap classifier | C9 | Review layers (§12 Q32) | Proposed: use the cheap model as a pairwise "same root cause?" classifier for merging per-hypothesis findings; keep the frontier model out of dedup |
| Effort levels: +35% bugs at constant resolution | C5 | Model routing (§6, D7) | Proposed: map risk band to effort (Low default, High/hard-block high). Measure recall and precision per effort level on §5 before adopting; C5's figure is internal and SR |
| Separate security reviewer, own threat model, blocks CI only on security | C8, C9 | Review layers; `bug_category: security` | Proposed: run `security` as a separate prompt/agent with its own blocking policy; seed its checklist from C8's six categories; require attack path in `comment` |
| Staged rollout: Slack-only → PR comments → blocking gate | C9 | Review layers (§7.1 layer 3) | Proposed: ship each new `bug_category` or reviewer in shadow mode (ledger only), then comment, then gate, with promotion tied to measured precision |
| Non-blocking deep review on push to main; daily invariant sweep with drift memory | C9, C10 | Review layers; living wiki (§2.4) | Proposed: route `git blame`-older candidates (D11) and expensive checks to a post-merge sweep; store invariant results in the wiki for drift detection |
| Agentic codeowners: risk by blast radius, complexity, and infra; 0 or ≤2 reviewers; decisions logged | C10 | Call graph (blast radius); review layers (§7.2) | Proposed: confirms the §7.2/D2 design; add the audit log of every auto-approve decision with inputs (already "keep an audit trail") |
| kb-reviewer cites the recorded lesson behind each finding | C11 | Living wiki; bugs JSON | Proposed: for `known-incident-pattern`, require `comment` to cite the wiki entry ID (incident or revert) |
| Monitoring plan from the diff before merge; flag missing instrumentation | C8 | Autonomous testing; review layers | Proposed: out of scope for `bugs`; note as a future layer after merge (spec non-goal boundary). Revisit if revert/incident rate becomes a gate (§12 Q23) |
| Runtime contamination: strip `.git`, deny egress, or agents fetch the fix | C14 | Autonomous testing; §5 eval dataset | Proposed: for historical-PR evals, reinit the repo at `base_ref` as one commit and block egress except the package mirror, or the reviewer can read the future fix from history or GitHub |
| Blind trajectory auditor | C14 | Eval (D8) | Proposed: add a judge pass that reads reviewer trajectories (without outcome) to flag answer retrieval or skipped context pulls |
| Flapping detection via repeated runs | C12 | Eval (§5.2 stability runs) | Proposed: report per-case flip rate across the ≥3 runs; treat flapping cases as prompt or policy underspecification, not noise |
| Relabel the eval set when the policy (category enum) changes | C12 | Bugs JSON (`bug_category`) | Proposed: version the enum; every enum change triggers relabelling of affected §5 records before results are compared |
| Compass complexity predictor + taxonomy router with 75% uplift threshold, cache-aware | C16 | Context selection; model routing | Proposed: start with a static rule (cheap model for context selection, frontier for decisions). Consider a learned router only after the ledger has outcome labels; require statistical uplift before routing to a costlier model |
| Planner/worker split; decorrelated review lenses stack | C17 | Review layers (D1, D12) | Proposed: give the verifier a different input lens from the finder (for example finder sees the diff plus context; verifier sees the call site and binding result only), and optionally a different model family; measure the stacking gain |
| Long tool outputs to files, not truncated; dynamic MCP tools | C19, C20 | Context selection | Proposed: write test logs, browser traces, and call-graph dumps to sandbox files the agent can grep, instead of truncating them in context |
| Cache breakpoints after stable layers; variable setup after them | C20 | Cost (D7) | Proposed: confirms D7 prefix order; place `review.md` before per-PR context and keep tool list static per run |
| Semantic search + grep beats grep alone, mostly on large repos | C21 | Context selection (§12 Q21) | Proposed: include grep-only vs grep+embeddings in the Q21 A/B, stratified by repo size (≥1,000 files) |
| Merkle-tree incremental index; content proofs | C22 | Codebase indexer | Proposed: refresh the indexer by file-hash diff on merge (fits §2.4 refresh); not needed for correctness |
| Missing environment shows as silent quality drop | C26 | Autonomous testing (§2.5) | Proposed: run a pre-flight "environment healthy" check (install, start, base-branch flow) and post "review incomplete" if it fails (D13), never a degraded review |
| Prebuilt snapshots; last-good build; record build SHA per run | C30 | Autonomous testing; sandbox (D6) | Proposed: cache a per-repo base environment built from lockfiles on main; record its SHA in the ledger per review |
| Two-agent autoinstall with explicit success commands; discard after 5 tries | C32 | Autonomous testing | Proposed: for repos without config, have the cheap model propose start and test commands, a second run verify them, and cache the result as the flow map draft (§12 Q8) |
| Sandbox failure messages name the constraint | C25 | Autonomous testing (D6) | Proposed: egress-proxy and sandbox denials return a structured reason to the tester agent to stop retry loops |
| Secret redaction in tool results; secret scanning | C27 | Bugs JSON; D5/D6 | Proposed: redact secrets from tool output before the model sees it, and scan `comment` and `diff_fix_suggestion` for secrets before posting |
| Malicious extension in a vendor marketplace | C40 | Sandbox; D10 skill vetting | Proposed: pin and hash-verify every tool, skill, and extension loaded into the review sandbox (strengthens §12 Q27) |
| Execution-based self-verification not yet shipped by Bugbot | C2, C3 | Contract check; autonomous testing (D3) | Proposed: keep D3 (reproduction per finding) as a differentiator; the deterministic binding check (§3.3) is a cheap reproduction Bugbot does not describe |
| Lint rule after the same mistake twice | C48, C37 | Review layers (§7.1 layer 1, D9) | Proposed: when the ledger shows the same category+pattern fixed ≥2 times in a repo, open a suggestion to codify it as a lint rule and stop LLM-reporting it |
| Lints take generated code from 90% to 100% working | C56 | Contract check | Proposed: run type checker and linter on `diff_fix_suggestion` in the sandbox before posting (extends D3 fix check) |

---

## 5. Open questions and gaps

1. **Resolution-rate judge validity.** C2 says "nearly all" spot checks agreed, with no count, error rate, or prompt. C4 applies the same judge to competitors on public repos. We cannot reproduce or compare. Our D9 diff-based rule stays primary.
2. **Inconsistent scale and rate figures.** Resolution rate is 70%+ (C0), 76% (C3), 78.13% (C4), and 80% (C5) across months. Volume is 2M PRs/month (C2) versus "hundreds of thousands of PRs per day" (C4), a ~3–5x gap. Treat all as SR orientation.
3. **Agentic Bugbot internals are undisclosed.** After the agentic switch (C2) Cursor does not say whether voting, the validator, or category filters survive, what tools exist, or how many agents run. Which of the v1 steps still matter is unknown.
4. **No precision figure separate from resolution.** Resolution conflates "real bug" with "author chose to fix". Unfixed real bugs and fixed non-bugs are both invisible (same issue as spec §12 Q33).
5. **BugBench is private.** Size, labelling process, and category mix are not published, so we can't compare it with our §5 dataset.
6. **Effort economics.** C5 claims +35% bugs at constant precision with high effort, with no cost delta. Is the cost per extra bug worth it on our dataset?
7. **Learned rules' safety.** Rules derived from developer replies are an injection surface (a reply can say "ignore auth checks in this repo"). C4 does not describe gating. Conflicts with spec D5; we would need owner approval for promotion.
8. **Execution-based verification.** Promised in Jan and Feb 2026 (C2, C3) with no later post confirming it shipped. There is no public data on whether running code improves review precision.
9. **Model naming drift.** Cursor's posts name Opus 4.8, Fable 5, GPT-5.5/5.6 Sol, Grok 4.5–4.7, and Opus 5, while spec §12 Q5 names Opus/Sonnet 5.5 and Fable 5.1. Routing strengths (C16) may not transfer across versions; re-measure on §5.
10. **Composer dependency.** Bugbot's speed and cost gains rest on an in-house model (C6) we can't use. Their fallback quality ("next best available model") is unquantified.
11. **Language coverage.** No Cursor post addresses AST or call-graph analysis; their approach is agentic search plus embeddings plus regex. Nothing informs §12 Q1 (non-Python support) or Q6 (call-graph depth).
12. **Dates and ownership.** The listing's featured block is out of date order. The SpaceX acquisition (Aug 2026) means some later posts reference "SpaceXSI" products. Treat page dates as approximate (as in spec §12 Q35).
13. **Fetch fidelity.** Blog posts were read from raw HTML (high fidelity). C0 came from a model summary; re-check its figures on the live page before citing.
