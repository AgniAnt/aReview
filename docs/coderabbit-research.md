# CodeRabbit Research Digest

Status: research input. Audience: coding agents that implement or extend the system in `docs/ai-code-review.md` (the "spec").

Conventions:
- **SR** = self-reported by CodeRabbit (a commercial vendor). Not independently verified. Do not treat as established fact.
- **Illustrative** = example or demo data on a marketing page, not a measurement.
- **Proposed** = a suggestion for our system. It is not a decided requirement.
- § numbers refer to sections of the spec.

---

## 1. Purpose and sources

Purpose: pull techniques, architecture details, and measurement ideas from CodeRabbit's product pages and blog, then map them to our pipeline (AST diff, call graph, contract check, `bugs` JSON, review layers, autonomous testing, context selection).

Fetched 2026-11-09 with WebFetch. WebFetch returns a model-written summary of each page, not raw text. Exact figures below come from those summaries. Re-check any figure against the live page before relying on it.

### 1.1 Source list

| # | Title | URL | Date | Relevant |
|---|-------|-----|------|----------|
| P1 | CodeRabbit Review | https://www.coderabbit.ai/review | undated | Yes |
| P2 | CodeRabbit Triage | https://www.coderabbit.ai/triage | undated (public beta) | Yes |
| P3 | CodeRabbit Change Stack | https://www.coderabbit.ai/change-stack | undated (© 2026) | Yes |
| P4 | CodeRabbit Secure | https://www.coderabbit.ai/security | undated | Yes |
| P5 | CodeRabbit Engineer for Slack | https://www.coderabbit.ai/agent | undated (© 2026) | Partly |
| B0 | Blog index (page 1 of 21) | https://www.coderabbit.ai/blog | n/a | Index only |
| B1 | GPT-6 Cyber Security Evaluation: CodeRabbit's Results | https://www.coderabbit.ai/blog/gpt-6-cyber-security-evaluation | 2026-11-05 | Yes |
| B2 | How Runtime Verification works in CodeRabbit Secure | https://www.coderabbit.ai/blog/runtime-verification-in-coderabbit-secure | 2026-11-03 | Yes |
| B3 | Why it's time to fight AI with AI | https://www.coderabbit.ai/blog/fighting-ai-with-ai | 2026-10-28 | Yes |
| B4 | CodeRabbit Secure now hunts across repos and beyond the merge | https://www.coderabbit.ai/blog/coderabbit-secure-cross-repository-security | 2026-10-27 | Yes |
| B5 | We're covering $1 million in security scans and fixes through November 8, 2026 | https://www.coderabbit.ai/blog/one-million-security-scans-and-fixes | 2026-10-24 | Low |
| B6 | CodeRabbit joins the SpaceX Super Intelligence Partner Network | https://www.coderabbit.ai/blog/coderabbit-spacex-super-intelligence-partner-network | 2026-10-22 | No |
| B7 | Which software engineering skills matter in the age of AI? | https://www.coderabbit.ai/blog/software-engineering-skills-ai-kunal-kushwaha | 2026-10-21 | No |
| B8 | CodeRabbit brings independent validation to Cursor Origin | https://www.coderabbit.ai/blog/coderabbit-independent-validation-cursor-origin | 2026-10-13 | Yes (narrow) |
| B9 | Open-source reviews on CodeRabbit topped 1 million in a single month | https://www.coderabbit.ai/blog/one-million-open-source-reviews-september-2026 | 2026-10-08 | No |

Blog page 1 lists exactly B1–B9, which are the nine named posts. There were no fetch failures.

Coverage gap: the index has 21 pages. Pages 2–21 were not fetched. Older posts are therefore not covered (see §5).

### 1.2 NOT RELEVANT

- **B6 SpaceX partner network**: partnership announcement. Its only content is a customer quote (SR: "100% to 150% more efficient" reviews; "2 to 3 times the effort" without the tool) with no method behind it.
- **B7 Kunal Kushwaha skills**: career advice. Its one relevant point (stakes, not authorship, should set review depth) is already in spec §7.2–7.3.
- **B9 One million OSS reviews**: usage volume and sponsorship figures (SR: 1,049,817 OSS reviews in Sept 2026, +14.4% vs Aug; 300,000+ public repos; counts include repeat reviews of the same PR). These describe adoption, not technique.
- **B5 $1M offer** (low relevance): a promotion that has ended. The page contradicts itself, since the header says the offer has concluded but the body says it runs through Nov 8. Its Deep Scan description repeats B3/B4. The one new detail is that Secure names "GPT 6.1 Sol and GPT Cyber" models (SR).

---

## 2. Per-page digest

### P1 CodeRabbit Review
- Positions review as a layer that stays **separate from the coding agent** that wrote the code ("built for agent-driven development").
- A "purpose-built harness" routes work across multiple models and **verification agents** before posting. The named agents are: Context agent (call-path analysis), Review agent (mutation analysis), and Security agent (sandbox scenario). No internals are given.
- **Code Graph**: a semantic map of repo relationships, dependencies, and "blast radius".
- **Sandbox reproduction**: suspected failure modes can be reproduced in a sandbox to add evidence to *selected* findings. Not every finding is reproduced.
- **Learnings**: reviewer corrections persist as team standards and feed future reviews.
- Agent loop: a coding agent (Codex and Claude are named) applies fixes, then review reruns. The demo shows bot flag → agent fix with a regression test → bot confirms. There are no figures and no noise-reduction claim.

### P2 CodeRabbit Triage
- Ranks **PRs** (not findings) on value, risk, and review effort, plus recent human activity and relationships between PRs (from the FAQ).
- Risk-based review depth: flags changes to sensitive systems so the team can review them more deeply.
- Flags **overlapping** and **stale** PRs ("Safe to close" bucket). Suggests reviewers from ownership, contribution, and review history.
- Kanban with P0/P1/P2 and a "Now (within daily capacity)" lane. Illustrative snapshot: 988–1,049 PRs, 80 "Requires Action", 14 "Now", 8 "Safe to close".
- Public beta, GitHub Cloud only. No architecture is described.

### P3 CodeRabbit Change Stack
- Large-PR explainability: groups changes into **cohorts by intent** (not file order), then into **ordered layers** tied to line ranges. Demo: Data model → API → UI, where each layer depends on the one before.
- Per-PR panels: architectural impact, blast radius (the risks plus the diff lines that cause them), dependencies (files and services), behavior analysis, and **Uncertainties** (what the agents cannot infer from the diff).
- Demo open question: "can an expired invitation still grant access?". This is an explicit unknown, not a finding.
- Can generate sequence, state-machine, and ER diagrams. Two-way sync with GitHub review comments and approvals.
- Illustrative demo: 9 files (+377/−3), 3 layers, 2 findings (1 Critical merge conflict, 1 Minor UX), 9/11 pre-merge checks passed. The comprehension-gap chart is labelled conceptual.

### P4 CodeRabbit Secure
- Pipeline: **Map → Hunt → Verify**. Agents use "semantic reasoning, not static rules" to trace attack paths across files, services, and trust boundaries.
- **Noise control by verification**: checks reachability, exploitability, and blast radius. Findings that an attacker cannot reach are **capped at low severity**.
- Scan types: **PR scan** (inline, pre-merge) and **Deep Scan** (whole repo plus infra files and secrets, on demand or scheduled).
- **Auto-repair** ("Fix with AI") drafts a PR. A human reviews and merges it, and the vendor states it never auto-merges.
- **Posture map**: a per-component verified vs "not yet walked" status (Illustrative: auth 12/14).
- Outputs: a findings API with CSV and **SARIF** export. A merge-blocking GitHub check is off by default and only blocks when an admin makes it required.
- SR figures: "50% more vulnerabilities in AI-written code" (their own report); "17k+ teams"; time-to-exploit 847 days (2018) vs <1 day (2026). The page does not source the time-to-exploit figures.
- Illustrative vuln classes: IDOR across services, prompt injection, logic flaws (discount after payment capture), injection chains (sanitised for HTML, not SQL), SSRF, and check-then-act races.

### P5 CodeRabbit Engineer for Slack (agent)
- A general SDLC agent in Slack. The FAQ says it **shares an engine with PR review**.
- Context layer: code, tickets, docs, monitoring, and cloud data via connectors, plus custom **MCP** servers.
- **Three-tier memory**: team (org-wide), channel (team patterns and runbooks), and thread (task). Privacy boundaries apply: public channels use global knowledge, private channels and DMs use private knowledge.
- Runs in sandboxes and supports custom sandboxes and skill files. Governance: hierarchical scopes limit repos, tools, and spend.
- Pricing (SR): $0.4/min of active agent time (~$24/h), billed per second. The page sets this against per-token pricing, which it calls hard to forecast.
- Illustrative incident: a Datadog p99 alert (380ms → 12.4s) correlated to a Terraform change in PR #3301, then a revert PR and a drafted postmortem.

### B1 GPT-6 Cyber Security Evaluation (2026-11-05)
- Benchmark: **33 cases**, each with **one known security issue**, across 9 languages. Compares the ensemble with and without GPT-6 Cyber. High reasoning effort was used in the investigation stage only.
- **Scoring on the final developer-facing report only, after verification.** A catch requires naming the issue *and* explaining its cause and impact. A nearby or different issue does not count.
- SR results: catches 22/33 (66.7%) vs 19/33 (57.6%), +9.1 points (+3 cases). High and medium findings 361 vs 218 (+65.6%). All-severity findings 979 vs 592.
- **Overlap analysis**: 17 both, 5 new-only, **2 lost**, 9 neither. The net gain is 3, which hides churn.
- The vendor acknowledges: volume ≠ validity (finding validity was not yet assessed); some investigations needed recovery runs; model effect is not separated from reasoning effort; n=33 is small.
- Recommendations (vendor): keep already-caught issues in the regression set. Judge a model by additional issues worth fixing and by verification effort, not by report volume. SR context: 13,000+ Deep Scans across 2,700+ orgs (Oct 23–Nov 3).

### B2 Runtime Verification in CodeRabbit Secure (2026-11-03)
- Step 1, **Recheck**: confirm the finding still exists on current code. Stop if it is gone or the recheck is inconclusive.
- Step 2, **Reproducer Guide**: prerequisites, steps, expected observations, and **stated assumptions**. It lists values the developer must supply.
- Step 3, **Review before run** (unless auto-verify is on). The run uses the **revision pinned in the guide**, not latest.
- Step 4, **Environment**: a prepared sandbox with dependencies, config, and startup instructions, or the user's own test environment. "A running process does not prove readiness."
- Step 5, **Result** reports two gates, **Application ready** and **Target exercised**, plus evidence, completed steps, and limitations. Verdicts: Reproduced / Not reproduced / Inconclusive. Each verdict holds only for the tested revision and environment. "Not reproduced" is narrower than "safe". A 403 counts only if the request reached the ownership check.
- Safety: use a dedicated test environment and no production credentials. The agent can read env vars, so scripts must not print secrets, because output is kept as evidence. No accuracy figures and no sandbox internals are given.

### B3 Why it's time to fight AI with AI (2026-10-28)
- **Hypothesis testing**: a finding must be backed by code that executes. Comments and function names can guide the search but cannot prove a safeguard exists.
- **Independent verifier**, separate from the finder, checks reachability, existing safeguards, and exploit conditions, then grades reachability and exploitability. The vendor concedes that a second model can repeat the first model's wrong assumptions.
- **Preserved uncertainty**: the Attack Surface Map marks points verified / unverified / stale.
- **Delta scans**: after a full baseline, later scans cover only changed code plus stale or unverified points. A verified-coverage target triggers scans.
- Worked example: two $60 refunds on a $100 payment each pass a per-request check (missing cumulative check). The vendor notes that provider limits or an atomic reservation might block the over-refund, so the finding must be verified along the full execution path.
- SR: 8,000+ high or critical vulns across 600 orgs in 30 days. Broken access control, SQLi, and auth failures were >70%. Cited third-party figure: an Anthropic report (Nov 2025) says AI did 80–90% of the tactical work in an espionage campaign against ~30 orgs.

### B4 CodeRabbit Secure cross-repository security (2026-10-27)
- **Cross-repo**: Deep Scan reads linked repos (discovered during review, or linked manually with notes on how the repos relate). The target repo is scanned. Linked repos give **read-only context**. Up to **20 linked repos** per scan. Safeguards in other repos are checked before a finding is published (SR: this "reduces noise").
- Every Deep Scan finding goes through a **verification agent** (reachability plus safeguards). Eligible findings are also replayed in a throwaway sandbox, never against production.
- **Delta scans** run on files changed since the baseline plus stale map areas. Coverage target (Illustrative: 88%) with a monthly spend cap. Scans can be **paused at budget and resumed**.
- Finding lifecycle: recheck against default-branch tip; severity adjustment (can be saved as a learning); **auto-resolve when a fix merges**. The merge-blocker is not cleared by resolving the GitHub thread. It persists until the fix is verified.
- Org reporting: **acceptance rate**, unaddressed findings, **findings per 100 reviewed PRs** per repo, remediation by PR owner, breakdowns by reachability and exploitability. Jira/Linear auto-ticketing above a severity threshold.

### B8 Independent validation in Cursor Origin (2026-10-13)
- Framing: the reviewer is independent of the author, whether that author is a human or an agent. Humans keep merge decisions.
- Flow: on PR open, post a walkthrough plus inline findings. The author replies in the thread to ask for an explanation or more investigation. After the fix is pushed, the tool **re-validates and posts a passing check on the commit**.
- Example finding: a cart total ignored quantity (3 × 1,250 cents should be 3,750). This is a concrete numeric counter-example in the comment.
- US region only. Origin is in early beta.

---

## 3. Cross-cutting themes

### 3.1 Runtime verification versus static reasoning
- CodeRabbit layers its checks: semantic (LLM) reasoning finds candidates, a verifier agent checks reachability and safeguards statically, and only **eligible** findings get a runtime replay (P1, P4, B2, B4).
- Runtime results carry explicit preconditions: an app-ready gate, a target-exercised gate, a pinned revision, and a three-state verdict. "Not reproduced" ≠ safe (B2).
- Lesson for us: our autonomous tester (§2.5) needs a **target-exercised** signal. Without one, a passing flow says nothing about the changed code.

### 3.2 Triage and noise reduction
- Noise is controlled after generation, not by generating fewer findings: verifier rejection, a severity cap for unreachable findings, and cross-repo safeguard checks (P4, B4).
- Uncertainty is shown as its own category ("Uncertainties", "unverified/stale") instead of being forced into a finding (P3, B3).
- Prioritisation happens at the **PR queue** level (P2): value, risk, effort, overlap, staleness. This is separate from per-finding severity.
- B1 shows the trade-off: a stronger model gave +65% findings but only +3 catches. Volume without validity is noise.

### 3.3 Security scanning and auto-fix
- There are two cadences: a diff-time PR scan, and a scheduled whole-repo Deep Scan with delta rescans (P4, B4).
- A finding has a lifecycle: recheck → severity adjust → fix PR → verified close → auto-resolve. Merge blocks persist until the fix is verified (B4).
- Auto-fix always goes through human review. There is no auto-merge (P4, B3, B5).
- Interop: SARIF and CSV export, a findings API, and ticketing (P4, B4).

### 3.4 Benchmarking with independent validation
- B1 is the most reusable method: a fixed case set with **one known issue per case**, scoring only the **final post-verification report**, catches that require cause plus impact, and an **overlap table** (both / new-only / lost / neither).
- This matches our tuning loop (§5). The overlap table makes regressions visible behind net gains.
- The vendor says report volume must be audited for validity. Its own validity audit is still pending.
- "Independent validation" in B6 and B8 is a positioning term (reviewer separate from author). There is no third-party benchmark in any fetched source.

### 3.5 Agent-based review
- Specialised agents: context (call paths), review (mutation analysis), security (sandbox) (P1). Finder and verifier are kept separate (B3).
- Memory: Learnings from reviewer corrections (P1, B4), and team / channel / thread memory tiers (P5).
- Closed loop with coding agents: flag → agent fix with a regression test → re-review → passing check (P1, B8).
- Cost model: billing by agent-time rather than tokens, plus scan spend caps with pause and resume (P5, B4).

### 3.6 Self-reported figures ledger

All rows are SR unless marked otherwise. None has a published method or third-party audit.

| Figure | Value | Source | Caveat |
|--------|-------|--------|--------|
| Known-issue catches, with vs without GPT-6 Cyber | 22/33 vs 19/33 | B1 | n=33; 2 cases lost; effort not isolated |
| High/medium findings, with vs without | 361 vs 218 | B1 | Validity not assessed |
| All-severity findings, with vs without | 979 vs 592 | B1 | Validity not assessed |
| Deep Scans run (Oct 23–Nov 3) | 13,000+ across 2,700+ orgs | B1 | Product-wide |
| High/critical vulns found (30 days) | 8,000+ across 600 orgs | B3, B4 | No precision figure |
| Share from access control, SQLi, auth failures | >70% | B3 | Same dataset |
| AI-written code vulnerabilities | "50% more" | P4 | Vendor's own report |
| Time to exploit | 847 days (2018) → <1 day (2026) | P4 | No source visible |
| Teams / customers | 17k+ | P4, P5 | Marketing |
| OSS reviews, Sept 2026 | 1,049,817 | B9 | Includes repeat reviews |
| Linked repos per Deep Scan | max 20 | B4 | Plan limits apply |
| Private-repo cache retention | ≤7 days | P4 | Policy statement |
| Agent price | $0.4/min (~$24/h) | P5 | Pricing, not performance |
| Review efficiency (Dub) | "100–150% more efficient" | B6 | Single customer quote |
| AI share of tactical work in espionage campaign | 80–90% | B3 citing Anthropic (Nov 2025) | Third-party; not re-fetched |

---

## 4. Mapping to our system

All suggested actions are **Proposed**.

| Finding | Source | Relevance to our pipeline | Suggested action |
|---------|--------|---------------------------|------------------|
| Separate finder and verifier; verifier rejects findings the code does not support | B3, B4, B5 | Contract check (§3.3), review layers (§7.6) | Proposed: frontier model verifies each §3.3 candidate in its own pass, with a prompt that forbids trusting names and comments |
| Comments and function names cannot prove a safeguard | B3 | Contract check, frontier review | Proposed: require each `bugs` entry's `comment` to cite the executed code path (file:line), not identifiers alone |
| Unreachable findings capped at low severity | P4 | Call graph (§3.2), `bugs` JSON | Proposed: drop or downgrade candidates whose changed symbol has no caller path from an entry point in the call graph |
| Cross-repo safeguards checked before publish; linked repos are read-only context, max 20 | B4 | Call graph, context selection (§2.3) | Proposed: allow the indexer to load configured sibling repos as read-only context for caller resolution. Treat cross-repo callers as `in_diff: false` |
| Runtime verdicts gated on "Application ready" and "Target exercised" | B2 | Autonomous testing (§2.5), evidence (§8) | Proposed: tester output records `app_ready` and `target_exercised`. A pass without `target_exercised` raises the verification-gap factor (§7.2) |
| Three-state verdict (Reproduced / Not reproduced / Inconclusive); "not reproduced" ≠ safe | B2 | Autonomous testing, risk scoring | Proposed: adopt the three states in tester reports. Never use "not reproduced" to lower risk |
| Pinned revision for runtime runs; recheck finding still exists before acting | B2, B4 | Webhook / sandbox, P5 stability | Proposed: bind every run and finding to `head_ref` SHA. On a new push, recheck prior findings before reposting (spec §9 dedup) |
| Reproducer guide with stated assumptions and missing inputs | B2 | Autonomous testing, evidence | Proposed: for runtime-relevant findings, attach a repro-steps section in `comment`. Keep the `bugs` keys fixed |
| Secrets readable by agent; output retained as evidence | B2 | Sandbox, evidence | Proposed: sandbox uses test-only credentials and redacts env values from logs before they are stored as evidence |
| "Uncertainties" listed separately from findings | P3, B3 | `bugs` JSON (§4), skip-nitpicks | Proposed: do not emit uncertain items as bugs. Log them in the trajectory (§9) or a separate non-posted channel. `bugs` stays bug-only |
| Benchmark scores final post-verification report only; catch needs cause plus impact | B1 | Prompt-tuning loop (§5) | Proposed: §5.2 matching also requires `comment` to state cause and impact (LLM-judged), and scores only post-verification output |
| Overlap table (both / new-only / lost / neither) | B1 | Tuning loop, run-to-run stability (§9) | Proposed: report this table for every prompt or model change. Block changes that lose previously caught bugs without review |
| Volume up 65%, catches up 3 cases | B1 | Tuning loop, false positives (§9) | Proposed: treat finding count as a cost metric, not a quality metric. Track precision per category alongside recall |
| Keep already-caught issues in the regression set | B1 | Dataset (§5.1) | Proposed: the dataset is append-only. Every caught bug becomes a permanent regression case |
| Delta scans with stale tracking and coverage target | B3, B4 | Living wiki (§2.4), cost (§6) | Proposed: wiki crawl re-indexes only files changed since the last crawl plus entries marked stale (one answer to open question §10.7) |
| Spend cap with pause and resume | B4 | Cost routing (§6) | Proposed: per-run token budget. On hitting it, persist state and emit partial results marked incomplete, never silently truncate |
| Finding lifecycle: auto-resolve on verified fix; block persists until verified, not when the thread is resolved | B4, B8 | PR commenter, review layers | Proposed: on new push, re-evaluate open findings. Mark them resolved only when the rerun no longer reproduces them on the new SHA |
| Re-validate after fix and post a passing commit check | B8, P1 | PR commenter, layer 3 (§7.1) | Proposed: publish a GitHub check run (pass/fail) alongside the comment, so branch protection can require it |
| Merge-block check off by default; admin opts in; severity threshold | P4, B4 | Hard blocks (§7.3), gating | Proposed: ship the check non-blocking by default. Blocking needs explicit repo config and a category or severity threshold |
| Acceptance rate, findings per 100 PRs, remediation by owner | B4 | Measurement (§9) | Proposed: add "findings per 100 PRs per repo" next to the addressed rate. Already close to the spec's addressed-rate metric |
| Learnings: reviewer corrections persist and severity adjustments become rules | P1, B4 | Living wiki, context selection | Proposed: dismissed findings (with reason) feed the wiki and are retrieved by Find Context. They also become negative examples in §5 |
| Specialised agents: context (call paths), review (mutation analysis), security (sandbox) | P1 | Find Context, Analyze bugs | Proposed: evaluate mutation analysis (perturb changed code, check whether tests catch it) as an extra signal for the verification-gap factor |
| Change cohorts and ordered layers; blast radius tied to diff lines | P3 | AST diff (§3.1), call graph | Proposed: group `changed_symbols` by call-graph connectivity into cohorts for the frontier prompt. Topologically order them callee-first |
| PR-level triage on value, risk, effort, overlap, staleness | P2 | Risk scoring (§7.2) | Proposed: out of scope for v1. Reuse §7.2 factors if queue ranking is added later |
| SARIF / CSV export; findings API | P4 | `bugs` JSON | Proposed: add an optional SARIF converter downstream of `bugs`. Do not change the `bugs` keys |
| Three-tier memory (team / channel / thread) with privacy boundaries | P5 | Living wiki, MCP context | Proposed: scope wiki entries as org / repo / PR. Never surface private-repo entries in another repo's review |
| Agent-time billing vs token billing | P5 | Cost (§6), reward per dollar | Proposed: record wall-clock sandbox time per run next to tokens in cost-per-review |
| Second model can repeat the first model's wrong assumptions | B3 | Adversarial review (§7.6), open question §10.12 | Proposed: the verifier uses a different model family or a context without the finder's reasoning trace |
| Auto-fix PRs always human-reviewed; never auto-merge | P4, B3 | `diff_fix_suggestion`, gating | Proposed: keep the fix as a suggestion only. No auto-commit by the review agent |

### 4.1 Priority order (Proposed)

Suggested build order for the rows above, cheapest and highest-leverage first:

1. Verifier pass plus reachability downgrade. These need only the existing call graph and frontier model.
2. Overlap table and post-verification scoring in the tuning loop (§5). This is cheap and catches regressions.
3. Pin findings to SHA, recheck on push, auto-resolve. This addresses P5 (new bugs on every run).
4. `app_ready` / `target_exercised` gates and the three-state verdict for the autonomous tester.
5. Delta crawl for the living wiki; Learnings from dismissed findings.
6. Later: cross-repo context, SARIF export, change cohorts, mutation analysis.

---

## 5. Open questions and gaps

1. **Blog coverage**: only index page 1 of 21 was fetched. Older posts (for example earlier benchmark posts, architecture or context-engineering posts) may hold more detail. B1 mentions a "September code review evaluation" that was not fetched.
2. **Fetch fidelity**: WebFetch returned model summaries, not raw HTML. Exact figures and quotes should be spot-checked on the live pages before citing.
3. **No internals**: no fetched source describes how the Code Graph is built (parser, languages, dynamic-call handling), how call-path analysis works, or how "mutation analysis" is used. None of it maps directly to our `ast` call graph (§3.2).
4. **Validity of findings unmeasured**: B1 counts findings but has not assessed how many are true positives. No source gives a precision or false-positive rate for review or Secure.
5. **Runtime verification eligibility**: which findings qualify for runtime replay, and how "Target exercised" is detected (coverage, tracing, or log matching), is not stated.
6. **Independent validation is positioning, not third-party evaluation**: no external benchmark or audit was found in the fetched sources.
7. **Self-reported adoption figures** (17k teams, 8,000+ vulns per 30 days, 13,000+ Deep Scans, 1.05M OSS reviews) have no published method. Treat them only as indicators of scale.
8. **Inconsistency**: B4 and B5 say the $1M offer runs through Nov 8 but their headers say it has concluded. The time-to-exploit figures (P4) have no visible source.
9. For us: should a verifier pass (separate model or context) become a required stage between Analyze bugs and Submit review? This relates to spec §6.2 ("delegate discovery, not verification") and §10.12.
10. For us: should uncertain items get a non-posted output channel, or stay only in trajectory logs, given the fixed `bugs` schema (§4.2)?
11. For us: is cross-repo read-only context in scope for the call graph, and what is the repo-link config shape?
