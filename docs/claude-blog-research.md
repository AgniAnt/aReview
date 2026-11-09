# Claude Engineering Blog: Research Digest for the AI Code Review System

Status: research input. Audience: agents that implement or extend the system in `docs/ai-code-review.md` (the "spec").

Conventions:
- Figures are as stated in each post. Anthropic figures are **self-reported**. No post provides independent verification.
- **Proposed** marks a suggestion for this project. It is not a decided requirement. Confirm it before you build it.
- § references point to sections of the spec.

---

## 1. Purpose and sources

Purpose: pull findings from 16 Claude engineering blog posts that apply to our review pipeline, and map each one to a spec section and a proposed action.

Fetch status: all 16 URLs were fetched on 2026-11-09 and none failed. The fetch tool returns a model-generated reading of each page, not raw HTML. Before you rely on any single figure, re-check it against the live page.

| # | Title | Author(s) | Date | URL |
|---|-------|-----------|------|-----|
| 1 | How the Claude Code team does code review | Addy Osmani, Morgan Westlee Lunt, Qing Wang | 2026-10-15 | https://claude.dev/blog/solving-code-review-at-anthropic/ |
| 2 | Building effective agent automations | Lance Martin, CJ Avilla | 2026-10-08 | https://claude.dev/blog/building-effective-agent-automations/ |
| 3 | Claude Code in the cloud: a field guide to cloud sessions | Addy Osmani | 2026-10-06 | https://claude.dev/blog/claude-code-in-the-cloud/ |
| 4 | Getting started with Claude Code mods | Addy Osmani | 2026-10-01 | https://claude.dev/blog/getting-started-with-claude-code-mods/ |
| 5 | Automating eval design and hillclimbing with Claude | Lance Martin | 2026-09-28 | https://claude.dev/blog/automating-eval-design-and-hillclimbing/ |
| 6 | Building with Claude Sonnet 5.5 | Addy Osmani | 2026-09-28 | https://claude.dev/blog/building-with-claude-sonnet-5-5/ |
| 7 | Using Claude Code: Spending your effort | Thariq Shihipar | 2026-09-25 | https://claude.dev/blog/spending-your-effort/ |
| 8 | What a task costs on Opus 5.5 | Addy Osmani | 2026-09-25 | https://claude.dev/blog/what-a-task-costs-on-opus-5-5/ |
| 9 | How we made claude.ai 3x faster in two weeks | Raymond Wang, Sam Attard, Issac G. | 2026-09-23 | https://claude.dev/blog/how-we-made-claude-ai-faster/ |
| 10 | Getting the most out of Opus 5.5 in Claude and Claude Code | Addy Osmani | 2026-09-22 | https://claude.dev/blog/getting-the-most-out-of-opus-5-5/ |
| 11 | The new rules of context engineering for Claude 5 generation models | Thariq Shihipar | 2026-07-24 | https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/ |
| 12 | Lessons from Building Claude Code: How We Use Skills | Thariq Shihipar | 2026-06-03 | https://claude.dev/blog/lessons-from-building-claude-code-how-we-use-skills/ |
| 13 | A harness for every task: dynamic workflows in Claude Code | Thariq Shihipar, Sid Bidasaria | 2026-06-02 | https://claude.dev/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code/ |
| 14 | Using Claude Code: The unreasonable effectiveness of HTML | Thariq Shihipar | 2026-05-20 | https://claude.dev/blog/using-claude-code-the-unreasonable-effectiveness-of-html/ |
| 15 | Lessons from building Claude Code: Prompt caching is everything | Thariq Shihipar | 2026-04-30 | https://claude.dev/blog/lessons-from-building-claude-code-prompt-caching-is-everything/ |
| 16 | Seeing like an agent: how we design tools in Claude Code | Thariq Shihipar | 2026-04-10 | https://claude.dev/blog/seeing-like-an-agent/ |

---

## 2. Per-post digest

### P1. How the Claude Code team does code review (most relevant)
- Goal: use the least human review time that does not raise reverts or incidents. Tracked metrics: human approvals per PR (median and p95), human comments, and time a PR waits on a person. Revert and incident rates are guardrails.
- Self-reported figures:
  - Claude writes ~90% of Claude Code's code.
  - Merged PRs more than doubled from June to September, while the revert share fell by more than 40%.
  - Monorepo changes rose from ~2,500/day (May) to >30,000/day (September).
  - PRs with substantive review comments rose from 16% to 54%.
  - Pass after one review-and-fix round rose from 65% to 78%.
  - PRs needing 5+ rounds fell from 8.9% to 1.5%.
  - Mean reviews to pass fell from 2.08 to 1.32.
  - ~59% of repeat flags were on code present since the first pass.
  - One PR ran >50 rounds overnight.
  - Verification makes results "2–3x better" (Boris Cherny's personal estimate).
- Seven layers:
  1. Verification before the PR, including a fresh-context subagent review.
  2. AI review on every PR. Parallel bug hunters run first, then **verifier agents vote** and only findings with enough agreement are posted. Style, diagnostic, and cleanup findings are dropped. Tags are Important, Nit, and Pre-existing. Review depth is Quick, Standard, or Deep.
  3. Blocking security review. A finding clears automatically when a later commit fixes it.
  4. Required CI. The PR template asks for Testing, Risk, and How-to-revert.
  5. Risk-tiered human approval: Low needs 0 approvals, Medium needs 1, High needs 2.
  6. Merge queue, flags, and a `latest`→`stable` release path.
  7. Rollback, plus incident reviews that ask "which layer should have caught it".
- Approval rules:
  - A classifier sets the risk tier from the change, and a second classifier can force a human regardless of tier.
  - CI workflow files always need a human.
  - A bot's approval never counts as a human's.
  - Approvals are tied to a commit and are dismissed when new commits land.
- Reviewer improvement:
  - Incident and revert PRs, confirmed by experts, form an eval set that runs **daily**. Every prompt, tool, or model change is scored against it.
  - Each comment is classified by the author's response: fixed, wrong, intentional, or doesn't matter.
  - The reviewer also runs on known-clean PRs.
- Convergence controls:
  - After 3 rounds, ask the author to reconsider the approach.
  - After the first few passes, re-review only the diff.
  - Flag the bug class so the author fixes every instance.
  - Do not repeat posted findings.
  - Run the reviewer nightly over existing code.
- Configuration: a short `CLAUDE.md` describes the project. A `REVIEW.md` written by code owners states what failure looks like, caps nits, lists paths to skip, and requires file and line evidence. Recurring comments get fixed at the source (skills, lint rules, config files). The post admits a gap: there are no circuit breakers that tighten approval when revert or incident rates rise.

### P2. Building effective agent automations
- Reference implementation: Managed Agents (beta) daily brief, `anthropics/claude-quickstarts/managed-agents/daily-brief`. Cron `32 7 * * 1-5`. Budget cap 500 cents ($5) per run. A run that hits the cap stops with `budget_reached` instead of failing. Set the cap at 3–5x a normal run's cost.
- Components: sources (read-only), one destination, a versioned agent, a schedule, two memory stores (user-owned `preferences` that the agent can only read, and agent-owned `state`), and guardrails.
- Bookmarks instead of fixed windows: each source stores the timestamp of the newest item read. A failed read keeps its bookmark and is reported as "unreadable", never as a quiet day.
- Confirm before recording:
  - A post counts as sent only after the API returns ok and a message id.
  - Before posting, check whether today's edition already exists.
  - Use the run states "posting", "posted", and "maybe posted".
- Re-verify each item against its live source just before posting. Drop resolved items and list unverifiable ones in the run record. Copy links from source fields, never build them by hand.
- Ledger: one record per reported item (stable ID, source, last status), so open items are carried forward and not re-reported. Credentials stay in a vault outside the sandbox and are injected only for allowed hosts.

### P3. Claude Code in the cloud
- One fresh VM per task (~4 vCPU, 16 GB RAM, 30 GB disk) with its own branch. A proxy holds the GitHub token outside the VM, and the session gets a short-lived credential that can push only to its own branch.
- Network levels: None, Trusted (the default), Custom, and Full. All egress is logged by hostname. Self-hosted environments are in beta for Team and Enterprise.
- Setup script runs as root, must exit 0, and should finish within ~5 min to be cached. The cache rebuilds when the script or allowed hosts change, and about every 7 days. Services do not survive a cache restore, so start them per session (for example in a SessionStart hook; `CLAUDE_CODE_REMOTE` detects cloud).
- Demo: a flaky-test race was fixed and the suite passed 40 of 40 runs. Five doc errors were fixed by curling each endpoint. The logging session found the same race (a test failed in 5 of 8 reruns) and left it out of scope. Run times are inconsistent within the post (61/65/72 s in the text, 65/62/87 s in the figure).
- Routines can be triggered by a schedule (hourly at most), an HTTP call, or a GitHub event. They run without approval prompts and push to `claude/` branches. Auto-fix pushes fixes for failing checks and review comments. Commits carry a `Claude-Session` transcript trailer.
- Run untrusted PRs in a disposable VM with network set to None. Parallel sessions use plan limits in parallel. Projects can start up to 200 threads per day.

### P4. Getting started with Claude Code mods
- A mod is a JS/TS module that runs inside a Claude Code session (v2.1.287+). It exports `register(on, options)` and hooks events with `on(event, matcher?, async ($, e, next) => …)`. A hook can observe, rewrite, or answer (`{deny}`). The module has no Node or DOM; all external access goes through `$`.
- Mods are long-lived and stateful (`$.state`). They can render UI, register slash commands, and register model-callable tools. Settings hooks are per-event shell commands.
- The "Blast Radius" example holds risky Bash calls (`rm -rf`, `git reset --hard`, force push, migrations), shows a dry-run report, and asks the user to Proceed or Cancel. Hooks get 10 s of their own time per dispatch. The post says text matching can be bypassed, so hard blocks belong in permission rules.
- Tooling: `claude plugin validate` and `claude plugin test` (runs `*.test.ts` against the real runtime).

### P5. Automating eval design and hillclimbing
- A good eval has four properties:
  - Tasks mirror production.
  - Scores rise with model capability and effort.
  - The best model at max effort stays well below 100%.
  - Run-to-run variance is low.
- Avoid adversarial sampling (keeping only cases the current model fails). It measures that model's failure pattern, not what is hard. Include hard cases only when you can say why they are hard.
- Input priority: production transcripts, then bug reports and tickets, then 5–10 hand-written cases, then cases synthesized from the code.
- Graders, cheapest that fits first: programmatic checks (exact match, label set, JSON schema, tests), then an LLM judge that scores a rubric of checkable claims (not a 1–5 scale), sees outputs in random order without knowing the baseline, and is a different model from the one under test.
- Diagnostic checks:
  - Run the grader twice on the same output and confirm the verdict holds.
  - Check plumbing for timeouts, API errors, and truncation.
  - Warn on headroom when the baseline is ≥~95%.
- Hillclimbing loop:
  - Split cases into train and test.
  - Make one root-cause patch per round.
  - Decision rule: train↑ and test↑ means keep. Train↑ and test flat means revert (overfitting). Any regression means revert.
  - If the score stalls for 2–3 rounds, bucket the remaining failures by cause.
  - Never paste failure content into the prompt.
  - Report confidence intervals, and do not merge a gain within noise.
- Results:
  - 44-ticket support benchmark: 74.4% at 4.6¢ (Opus 4.8, high effort) became 98.9% at ~1¢ (Sonnet 5, low effort, audited prompt). Held-out: 90.5% vs 78.6% at ~1/5 the cost.
  - claude-api skill: 66.1%→87.9% over 24 rounds. The last phase fixed broken graders, not the model.

### P6. Building with Claude Sonnet 5.5
- Sonnet 5.5 is 30% faster than Sonnet 5 at the same per-token price, and costs up to 30% less for most workloads. Recommended for well-scoped repeated agent tasks, including **review**. Opus 5.5 is for complex judgment.
- Pricing, Sonnet 5.5 vs Opus 5.5, per MTok:
  - Input: $2 vs $4. Output: $10 vs $20.
  - Cache write 5 min: $2.50 vs $5. Cache write 1 h: $4 vs $8.
  - Cache read: $0.20 for both.
  - US-only inference costs 1.1x.
- Context and output: 1M context, 128k max output (300k on Batch with a beta header). The minimum cacheable prompt is now 512 tokens (was 1,024).
- Breaking API changes:
  - Forced `tool_choice` (`any`/`tool`) returns 400. Use `auto` with `strict: true` tools and `additionalProperties: false` on every object.
  - `thinking: disabled` returns 400. Use `between_tools`, which works only at low, medium, and high effort.
- Effort is recalibrated, so re-run your effort sweep. At low effort the model may skip verification; add a system-prompt paragraph that requires a real test, type-check, or build. Changing top-level effort invalidates the cache, so use per-message effort (beta) for one-off changes.
- Refusals return HTTP 200 with `stop_reason: "refusal"` and `stop_details` set to one of five categories. Server-side fallback retries only `cyber` and `frontier_llm`.

### P7. Spending your effort
- Higher effort means more independent judgment and verification. The gain is largest where verification matters: hardware, **code review**, and security.
- Terminal-Bench 3.0 pass rate by category, low to top effort:
  - Security (7 tasks): 64%→87%. Hardware (5): 34%→75%. ML (13): 54%→73%.
  - Science (15): 41%→61%. Software (20): 43%→56%.
  - Media (4): 18%→30%. Operations (10): 12%→22%.
- Failure analysis, Fable 5.1, 370 attempts per level:
  - Passed: 140 at low, 214 at max.
  - "Missed a case": 59→24, including "bug its tests missed" 40→14.
  - "Made the wrong call": 133→107.
  - "Picked the wrong reading" rose from 25 to 47.
  - Conclusion: effort fixes missed edge cases, not wrong approaches.
- Case studies, Opus 5.5:
  - `mvcc-lsm-compaction`: 0/5 at low, 4/5 at xhigh. The xhigh run reproduced the bug first and confirmed the test failed on half-finished fixes.
  - `cli-2ph-simplex`: 0/5 at low, 5/5 at high.
- Rule of thumb: low for sketching, medium for regular software work, high where verification or edge cases matter, max for autonomous hard problems or security. Suggested loop: implement at low effort, then verify at high effort.

### P8. What a task costs on Opus 5.5
- Cost drivers: turns × context, cache hit rate, output tokens (including thinking), and price.
- Opus 5.5 pricing per MTok: $4 input, $20 output, $0.20 cache read. Cache writes cost 1.25x input (5 min) or 2x (1 h). Batch is half price. An output token costs 100x a cache read.
- Worked example: 40 turns grow context from 20K to 120K, ~2.8M input tokens in total. Cost is $11.20 with no cache, $1.62 at 90% hits, and ~$0.99 at 96%. 60K output costs $1.20, the same as 6M cache reads.
- Adding 20K thinking tokens costs ~$0.40, about the same as a 10-turn retry loop. So high effort pays off only when it prevents a retry.
- Expect a cache write after any of these:
  - A pause longer than the TTL (5 min on API keys, 1 h on subscriptions).
  - A model switch.
  - An MCP connect or disconnect.
  - A compaction.
  - An effort change on Bedrock, Vertex, or a gateway.
- Routing: Sonnet or Haiku for lookups, log reading, test-output reading, and search subagents. Opus for writing code and review. Agent teams use ~7x the tokens of a standard session.
- `/claude-api prompt-audit` example: the model migration cut cost ~18%, and the audit cut ~9% more (~25% total).
- Baseline: ~$13 per developer per active day; 90% of users spend under $30. Diagnostics: low cache share points to a pause, model switch, or late MCP connection. Input many times larger than the conversation points to a looping session.

### P9. How we made claude.ai 3x faster in two weeks
- ~3x faster (geometric mean of 13 metrics). More than 3,000 changes merged with no customer-facing incident or rollback. Examples:
  - Fresh-load time to typeable page: 3,085→550 ms at p75.
  - Cowork send message: 928→48 ms.
- Principle: "measuring something makes it tractable". Lab benchmarks (instruction counts, React commits, DOM mutations) were kept only if they correlated with user latency.
- **CI ratchets**: each benchmark has a ceiling that can only go down. A PR that raises a count fails CI, and a daily job lowers ceilings after improvements.
- Guardrails:
  - Every PR got automated review plus at least one human approval.
  - Unit tests were written before optimizations.
  - ~200 flags were introduced, each classed as a kill switch or ramp, and more than half were removed by the end.
  - Rollout went employees → 1% → everyone.
- Scale: more than 150 parallel threads, some producing 50–100 PRs. Humans owned taste, direction, and sequencing.

### P10. Getting the most out of Opus 5.5
- Define "done" and when to stop. Remove "think step by step" lines; tune effort instead.
- Put stop rules in CLAUDE.md: continue unless blocked, and stop before destructive actions. Use subagents for audits and **check each subagent's evidence** before accepting it.
- Review pass before human review: report only **merge-blocking problems**, each with file, line, reason, and how to show the failure. Ask the model to **mark unconfirmed findings** and say where it looked.
- One tester reports that Opus 5.5 at lowest effort caught more bugs than Opus 5 at high, with fewer false alarms. This is anecdotal; no counts are given.

### P11. The new rules of context engineering (Claude 5 generation)
- Anthropic removed more than 80% of Claude Code's system prompt for Opus 5 and Fable 5 with no measurable eval loss. The TodoWrite description shrank from ~9,100 characters to one line.
- Old rule → new rule:
  - Rules → judgment.
  - Examples → interface design (enums, parameters).
  - Everything upfront → progressive disclosure.
  - Repetition → simple tool descriptions.
  - CLAUDE.md memory → auto-memory.
  - Simple specs → rich references.
- Conflicting instructions were common (for example, "leave documentation as appropriate" next to "DO NOT add comments"). Audit for them and delete them.
- CLAUDE.md should be light and mostly codebase gotchas. Move verification steps into a skill. Prefer code-based references, such as a test suite as the spec.
- Rubrics plus verifier agents let Claude check work against a team's taste.

### P12. How we use skills
- Skills fall into nine categories. Two matter here: #2 product verification (`signup-flow-driver`, `checkout-verifier`, which record video and assert state at each step) and #6 code quality and review (`adversarial-review`, ideally backed by deterministic scripts, run in hooks or GitHub Actions).
- Verification skills had "the most measurable impact" on output quality. No figure is given.
- The **Gotchas section** is the highest-signal content. It grows from real failures, from 1 line to 4 lines over 3 months in the example. Do not state the obvious, and avoid railroading the model with step lists.
- `SKILL.md` acts as a hub that points to spoke files (progressive disclosure). Descriptions are written for the model and say when to trigger.
- Persist data in an append-only log, JSON, or SQLite under `${CLAUDE_PLUGIN_DATA}`. A PreToolUse hook logs skill usage to find skills that under-trigger.

### P13. Dynamic workflows
- A JS harness built from `agent(prompt, {schema, model, isolation, agentType})`, `parallel()`, and `pipeline()`. Interrupted workflows resume where they stopped.
- It counters three single-context failures:
  - Agentic laziness (for example, 35 of 50 items addressed in a security review).
  - Self-preferential bias when the agent verifies its own work.
  - Goal drift across turns and compactions.
- Six patterns: classify-and-act, fan-out-and-synthesize (with a barrier), adversarial verification, generate-and-filter (with dedup), tournament (pairwise judging), and loop until done (for example, until no new findings).
- Relevant uses:
  - One verifier per rule, then a skeptic agent that filters false positives.
  - Cluster recurring review corrections into CLAUDE.md rules.
  - Quarantine: reader agents with no privileges handle untrusted content and emit structured summaries; a privileged actor acts on them.
  - A classifier routes tasks to Sonnet or Opus.
- Cost: example review-changes run used 14 agents, 482k tokens, 6m 12s. Example deep-research run used 22 agents, 1.1M tokens, 11m 3s. A panel of 5 reviewers is more than most coding tasks need. Set token budgets.

### P14. The unreasonable effectiveness of HTML
- Markdown gets hard to read past ~100 lines. HTML can carry tables, CSS, SVG, code, interactions, and layout.
- The code-review example renders the real diff with inline margin notes, color-coded by severity (blocking, nit, nice).
- End interactive pages with an export ("copy as JSON/prompt/diff") so changes flow back into the agent. Opinion piece with no performance metrics.

### P15. Prompt caching is everything
- The Claude Code team alerts on cache hit rate and **declares SEVs** when it drops. A few points of miss rate matter.
- Prefix order: static system prompt and tools → CLAUDE.md → session context → messages. Things that broke the cache: a timestamp in the static prompt, non-deterministic tool order, and changing tool parameters.
- Rules:
  - Send updates as messages (`<system-reminder>`), not system prompt edits.
  - Never switch model mid-session; use a subagent with a hand-off.
  - Never add or remove tools; use `defer_loading: true` stubs.
- Cache-safe forked compaction reuses the parent's exact prefix and costs ~1/10 of an uncached call (summary ~20k tokens). Side calls (summaries, skills) should share the parent prefix.

### P16. Seeing like an agent
- Structured tools beat parsing free-form output: AskUserQuestion beat both markdown parsing and a question array on the plan tool.
- Tools that helped weaker models can constrain stronger ones. TodoWrite with reminders every 5 turns was replaced by Tasks.
- Agentic search (Grep) replaced RAG: a pre-indexed vector DB was fast but fragile, and it handed Claude context instead of letting Claude find it.
- Claude Code has ~20 tools and a high bar for adding one. Expand capability through subagents (for example, the Claude Code Guide subagent) or progressive disclosure.

---

## 3. Cross-cutting themes

| Theme | Statement | Posts |
|-------|-----------|-------|
| T1 Measure first, then ratchet | Build a comparable number first. Lock wins with ceilings that can only tighten. Use outcome metrics as guardrails | P1, P5, P8, P9, P15 |
| T2 Risk-tiered human approval | The change, not the author, sets the tier. A bot never replaces a required human. Some paths always need a human. Start strict and loosen with evidence | P1, P9 |
| T3 Verify before review | Run tests, builds, or a reproduction before judging. Verification is the largest lever (P1 "2–3x" estimate; P12 "most measurable impact") | P1, P3, P6, P7, P10, P12 |
| T4 Verifiers filter findings | Independent agents vote or refute. Post only findings with agreement. Require file, line, reason, and how to reproduce. Mark unconfirmed findings | P1, P10, P13 |
| T5 Convergence and noise control | Cap rounds, re-review only the diff, do not repeat findings, flag the bug class, keep a ledger of reported items | P1, P2 |
| T6 Cache-aware design | Stable prefix, no mid-run tool or model changes, updates as messages, cache-safe side calls, hit rate monitored | P6, P8, P15 |
| T7 Effort and model routing | Effort pays off for code review and security edge cases. Cheap models do lookup and search. Re-run effort sweeps on each new model | P6, P7, P8, P13 |
| T8 Evals and hillclimbing | Incidents become evals. Train/test split, revert rule, grader consistency check, noise floor, cost at parity as an objective | P1, P5 |
| T9 Context-engineering rules | Fewer rules, no conflicts, interfaces over examples, progressive disclosure, gotchas over generic advice, agentic search over RAG | P11, P12, P16 |
| T10 Dynamic workflows | Fresh-context subagents counter laziness, self-preference, and drift. JSON schemas make synthesis reliable. Quarantine untrusted input | P13 |
| T11 Unattended-run hygiene | Bookmarks, confirm-before-record, re-verify before posting, spend caps, read-only tokens, vaulted credentials | P2, P3 |
| T12 HTML as output | Severity-annotated diffs and explainers for humans, with an export back to the agent | P1, P14 |

---

## 4. Mapping to our AI code review system

Reference points (self-reported by Anthropic in P1, not independently verified):
- Substantive-comment rate: 16%→54%.
- One-round pass rate: 65%→78%.
- Share needing 5+ rounds: 8.9%→1.5%.
- Mean reviews to pass: 2.08→1.32.
- ~59% of repeat flags were on pre-existing code.

Use these as orientation only. They are not targets.

| # | Finding | Source post | Relevance to our bugs-JSON pipeline or review layers | Suggested action |
|---|---------|-------------|------------------------------------------------------|------------------|
| M1 | Parallel bug hunters plus verifier agents that vote. Only findings with agreement are posted | P1, P13 | Addresses P4 (nitpicks) and §9 false positives. Fits between Analyze bugs and Submit review (§2.2) | Proposed: add a Verify stage. N independent verifiers check each candidate bug against the code and post it only on a majority vote. Use different prompts and contexts from the hunter (answers §10 Q12) |
| M2 | Tags Important / Nit / Pre-existing. Nits dropped | P1 | §4 keys are fixed. We have no severity or pre-existing field | Proposed: keep the schema unchanged. Drop nits before emitting. Track pre-existing findings in a side-channel ledger, not in `bugs`. Any key change needs a spec revision |
| M3 | Every finding needs file, line, reason, and how to show the failure. Unconfirmed findings are marked | P1, P10 | `changed_file`, `changed_lines`, and `comment` already cover location and reason | Proposed: require `comment` to name the failing call site or a reproduction. Drop or hold unconfirmed candidates instead of posting them |
| M4 | Convergence controls: 3-round cap, diff-only re-review, no repeated findings, flag the bug class | P1, P2 | Addresses P5 (new bugs every run) and §9 run-to-run stability | Proposed: keep a findings ledger per PR keyed by (file, symbol, category). Suppress re-posts. After round 3, post one "reconsider approach" note. Re-review only the incremental diff after round 2 |
| M5 | ~59% of repeat flags were on pre-existing code. Nightly runs over existing code | P1 | Separates PR-introduced bugs from old ones. Matters for addressed-rate accuracy | Proposed: use `git blame` to filter candidates whose lines predate `base_ref`. Send them to a scheduled nightly sweep, not to PR comments |
| M6 | Incident and revert PRs, confirmed by experts, form an eval run daily. Every prompt, tool, or model change is scored | P1, P5 | Matches §5.1 dataset sources (reverts, incidents) and §2.4 wiki inputs | Proposed: one labelled set feeds both the wiki and the §5 eval. Run it on every reviewer change and daily. Add known-clean PRs to measure false positives |
| M7 | Train/test split with a keep/revert rule. Grader run twice. Noise floor measured before tuning. Never paste failures into the prompt | P5 | §5.2 loop has no overfitting guard or noise check | Proposed: split the PR dataset into train and test. Apply P5's decision rule. Use the §5.2 matching rule as the programmatic grader. Report confidence intervals. Forbid copying dataset bugs into prompts |
| M8 | Avoid adversarial sampling. Include hard cases only with a stated reason. Warn when headroom is ≥~95% | P5 | Dataset construction (§10 Q11) | Proposed: give each dataset record a `why_hard` note. Do not build the set only from cases the current reviewer misses |
| M9 | Classify each comment by author response: fixed, wrong, intentional, or doesn't matter | P1 | Sharpens §9 addressed-rate and false-positive metrics | Proposed: store per-comment outcome with these four labels. Derive precision from them |
| M10 | Risk classifier sets the tier: Low 0, Medium 1, High 2 humans. A second classifier forces a human. CI workflow files always need a human. A bot never counts as a human | P1 | Cross-check for §7.2 bands and §7.3 hard blocks | Proposed: add CI workflow files (`.github/workflows/**`) to the §7.3 hard blocks. Require 2 humans for the High band. Dismiss approvals on new commits. Never count our reviewer as an approver |
| M11 | No circuit breakers yet (gap stated in P1) | P1 | §7.2 "start conservative" has no feedback trigger | Proposed: tighten the auto-approve band automatically when the rolling revert or incident rate rises above a threshold (value not decided) |
| M12 | Blocking security review clears automatically when a later commit fixes the finding | P1 | §4.3 `security` category; §7.1 layer 3 | Proposed: `security` findings block merge until resolved. Re-run on the next push and resolve automatically when the check passes |
| M13 | `REVIEW.md` from code owners: failure modes, nit cap, skip paths, evidence requirement. Keep `CLAUDE.md` short | P1, P11 | §2.3 already proposes `review.md` | Proposed: define `REVIEW.md` sections: failure-looks-like, skip-paths, gotchas, evidence. Load it always. Keep each one short. Move recurring comments into lint rules or the file |
| M14 | Verification gives "2–3x" better results (personal estimate). Real tests, not syntax checks | P1, P6, P7 | §2.5 autonomous testing, §8 evidence | Proposed: before the frontier model confirms a `contract-mismatch`, run the affected caller (or a generated test) in the sandbox. Attach the output as evidence |
| M15 | Effort helps most on code review and security. It fixes missed cases, not wrong approaches | P7 | §6 routing has no effort axis | Proposed: run Analyze bugs at high effort. Run hard-blocked or high-risk PRs at xhigh. Run context selection at low. Re-sweep against the §5 eval whenever the model changes |
| M16 | Sonnet or Haiku for search, logs, and test-output reading. Opus for judgment. Sonnet 5.5 suits repeated review tasks | P6, P8 | §6.1 cheap model vs frontier model (§10 Q5) | Proposed: candidates are Sonnet 5.5 or Haiku for Find Context and Opus 5.5 for the bug decision. Compare Sonnet 5.5 vs Opus 5.5 on the §5 eval with cost at parity as the objective |
| M17 | A classifier routes tasks to Sonnet or Opus by complexity | P13 | Ties to the §7.4 trunk-vs-leaf distinction | Proposed: route leaf-only, low-risk PRs to Sonnet 5.5 for the full review. Route trunk or high-risk PRs to Opus 5.5 |
| M18 | Prompt-cache prefix: static first, no mid-run tool or model changes, updates as messages. Hit rate monitored as an SLO | P8, P15 | §6 cost; §9 cost per review. Each review run is a long agent loop | Proposed: fix the order as static prompt + tools → `REVIEW.md` → PR context → messages. Use deterministic tool order. Use subagents, not a mid-session switch, for model changes. Add cache hit rate to §9 metrics and alert on it |
| M19 | Cost model: turns × context. 90% vs 96% cache hits ≈ $1.62 vs $0.99. An output token costs 100x a cache read | P8 | §9 cost per review by tier | Proposed: log turns, cache read/write, and output tokens per run. Batch API (half price) for nightly sweeps and eval runs |
| M20 | Forced `tool_choice` returns 400 on Sonnet 5.5. Use `strict` tools and `additionalProperties: false` | P6 | §4.2 "reject or retry invalid output" | Proposed: emit `bugs` through a strict tool or JSON schema with `additionalProperties: false` and an enum for `bug_category`. Keep post-hoc validation as a backstop |
| M21 | Refusals return HTTP 200 with `stop_reason: "refusal"`. Cyber safeguards apply on Sonnet 5.5 and Opus 5.5 | P6, P10 | The `security` category could trigger cyber declines | Proposed: handle `refusal` explicitly as a "review incomplete" status. Never treat it as `{"bugs": []}` |
| M22 | Failed reads are not quiet days. Confirm before recording. Re-verify before posting | P2 | Webhook pipeline: a crashed branch could look like "no bugs" | Proposed: if any decision-critical branch fails, post a "review incomplete" status, not an empty review. Record a finding as posted only after the GitHub API confirms. Re-check that the head SHA is unchanged before posting |
| M23 | Per-run spend cap set at 3–5x a normal run. Read-only tokens. Vaulted credentials | P2, P3 | Sandbox (§2.1) cost and security | Proposed: cap spend per review run. Give the sandbox a read-only repo token plus a comment-only credential held outside it. Use network None or Trusted |
| M24 | Untrusted PRs run in a disposable VM with network None. Quarantine pattern: unprivileged readers, privileged actor | P3, P13 | PR content (code, comments, linked tickets) is untrusted input that could carry prompt injection | Proposed: split the reviewer into an unprivileged reader (diff, MCP tickets, wiki) that emits structured candidates, and a privileged poster that holds the comment credential. Untrusted text never reaches the poster as instructions |
| M25 | Agentic search (Grep) replaced RAG, which was fragile and pre-chosen | P16 | §2.1 codebase indexer, §2.4 wiki retrieval | Proposed: keep the indexer and call graph as tools the agent queries, not as pre-stuffed context. Compare the two approaches on the §5 eval |
| M26 | Fewer, conflict-free rules. Enums over examples. Prompt audit cut ~9% cost | P5, P8, P11 | §5 prompt tuning. Risk of rule accretion as nitpick rules pile up | Proposed: audit the reviewer prompt for conflicts each tuning cycle. Express categories through the `bug_category` enum, not long examples |
| M27 | Skills: product-verification skills (signup and checkout drivers) and a growing Gotchas section | P12 | §2.5 sign-in and purchase flows; §2.4 wiki | Proposed: package each browser flow as a verification skill that records video and asserts state. Turn wiki incident patterns into per-area gotchas that feed `known-incident-pattern` |
| M28 | Dynamic workflow patterns: fan-out with a barrier, a verifier per rule plus a skeptic, loop until no new findings, JSON `schema` per agent | P13 | §2.2 Plan Review fan-out; §6.2 "wait for decision-critical results" | Proposed: implement Plan Review as fan-out → barrier → verify → synthesize, with a JSON schema on every subagent. Budget tokens per run. Example cost reference: 14 agents, 482k tokens |
| M29 | CI ratchets: ceilings only go down. A daily job tightens them | P9 | §9 metrics regress silently as prompts change | Proposed: ratchet eval recall per category (floor only rises) and false positives on known-clean PRs (ceiling only falls) as CI gates on reviewer changes |
| M30 | Flags (kill switch vs ramp) and staged rollout (employees → 1% → all) | P1, P9 | §7.5 feature flags; rollout of the reviewer itself | Proposed: roll the reviewer to one repo first, tune for a few weeks, then expand (P1 advice). Put each review layer behind a kill switch |
| M31 | HTML diff with inline severity margin notes. On-demand explainers for reviewers | P1, P14 | §8 evidence for the human layer (§7.6) | Proposed: optionally render an HTML evidence page (diff, findings, test output, screenshots) linked from the PR comment. The PR comment itself stays rendered from `bugs` JSON |
| M32 | Mods or skill hooks can hold risky commands for a dry run. Text matching can be bypassed | P4, P12 | §7.1 layer 2 (local review before push) | Proposed: optional local `/review` skill for authors. Hard blocks stay in server-side branch protection, never in client-side hooks |

---

## 5. Open questions and gaps

1. **Self-reported metrics.** Every P1 figure is internal to Anthropic. The 2–3x verification gain is one person's estimate. P10's "lowest effort beat Opus 5 at high" is a single anecdote with no counts. We need our own baseline before setting targets.
2. **Schema tension.** P1's severity and Pre-existing tags help triage, but §4 forbids new keys. Decide whether severity lives outside `bugs` (ledger or PR-comment rendering) or whether the schema gets a version bump.
3. **Verifier quorum.** P1 does not give the number of verifiers or the agreement threshold. P13 says 5 reviewers is more than most coding tasks need. Our N and threshold are not decided, and the cost per finding is unknown.
4. **Round cap vs our trigger.** P1 caps at 3 rounds. We have not defined what counts as a round (each push, or each review request).
5. **Effort mapping.** The P7 gains come from Terminal-Bench, not PR review. We have no data on how effort changes recall or precision on our §5 dataset.
6. **Model names.** The spec's "Jev" example is not a model in any of these posts. Our candidates from the posts are Haiku 5.5 (announced as coming "in the coming weeks" in P6, so availability is unconfirmed), Sonnet 5.5, Opus 5.5, and Fable 5.1 (escalation only).
7. **Indexer vs agentic search.** P16 moved away from RAG, but our spec keeps an indexer and wiki. No post measures review recall for RAG vs search. Run our own A/B.
8. **Pre-existing bug handling.** The nightly sweep (P1) and the git-blame filter need an owner and a destination (issues? wiki?). Not decided.
9. **Circuit breakers.** P1 lists these as an open gap at Anthropic too. The threshold and window are not decided.
10. **Language support.** None of the posts address non-Python call-graph analysis (§10 Q1).
11. **Prompt-injection risk.** No post measures injection through PR content into a reviewer. M24 is a design inference from P13's quarantine pattern, not a tested result.
12. **Cache economics for short reviews.** P8 and P15 figures assume long sessions. For small PRs (few turns), cache-write overhead may dominate. Measure before optimizing.
13. **Fetch fidelity.** Pages were read through a summarising fetch tool. Figures in tables and figure captions (for example, P3 timings and P13 panel examples) may be incomplete. Re-check any figure before it gates a decision.
14. **Managed Agents and cloud routines** (P2, P3) are options for hosting the webhook sandbox. Whether to use them instead of our own sandbox is not decided.
