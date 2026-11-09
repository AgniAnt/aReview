# Graphite Blog Research Digest

Audience: agents implementing `docs/ai-code-review.md` (the spec). Read-only input to a future spec revision.
Conventions follow the spec. **(SR)** means self-reported by Graphite or a named customer and not independently verified. Use SR figures as orientation, never as targets. All suggestions in §4 are **Proposed**.

---

## 1. Purpose, index, sources

**Purpose.** Pull out techniques, figures, and recommendations from Graphite's blog that bear on our pipeline: AST diff, call graph, contract check, `bugs` JSON, review layers, autonomous testing, and context selection. Graphite sells stacked PRs, a merge queue, and an AI reviewer (Graphite Reviewer → Diamond → Graphite Agent). Graphite agreed to join Cursor in Dec 2025.

**Index read.** https://graphite.com/blog on 2026-11-09: 99 posts (Sep 2023 to Apr 2026), not paginated.

**Fetch method and failures.**
- Every live post URL (`graphite.com/blog/<slug>`) returned HTTP 404 on 2026-11-09, through both WebFetch and curl. Only the listing page loads. `graphite.com/feed.xml` holds summaries only.
- WebFetch is blocked from `web.archive.org`. Posts were read as raw HTML from Wayback Machine snapshots (2026 captures) via curl, with text extracted locally. Archive.org rate-limited the downloads, but after retries all 66 selected posts were retrieved.
- Unlike the CB, AN, and GR digests, this one was built from raw page text, not model summaries. It is still an archived copy, not the live page.
- Archive metadata dates run one day after the listing dates (a UTC offset). The listing dates are used below.
- 11 low-relevance posts were read by keyword skim, not in full. They are marked "(skim)" in the brief table.

**Sources (digested in §2; GT = Graphite).** Dates are listing dates.

| ID | Title | Date |
|----|-------|------|
| GT1 | Introducing Code Tours: a new way to review | 2026-04-01 |
| GT2 | Introducing Cursor Cloud Agents in Graphite | 2026-03-02 |
| GT3 | Building the future of software development with Cursor | 2025-12-19 |
| GT4 | Reimagining the PR Page (+ changelog 11-20-2025) | 2025-11-24 |
| GT5 | Meet Graphite Agent (+ changelog 10-16-2025) | 2025-10-07 |
| GT6 | How we sped up code search for Graphite Chat | 2025-09-09 |
| GT7 | Introducing Graphite Chat (+ changelog 09-17-2025) | 2025-08-18 |
| GT8 | AI is writing code—here's why it also needs to review that code | 2025-07-28 |
| GT9 | How I got Claude to write code I could actually ship | 2025-07-24 |
| GT10 | How we built the first stack-aware merge queue | 2025-06-02 |
| GT11 | How we organize our monorepo to ship fast | 2025-05-27 |
| GT12 | Making AI code review available to everyone | 2025-03-26 |
| GT13 | Graphite raises $52M and launches AI code review agent Diamond | 2025-03-17 |
| GT14 | Why AI will never replace human code review | 2025-03-17 |
| GT15 | How Graphite's eng team ships code remarkably fast | 2024-12-14 |
| GT16 | Why we chose Anthropic's Claude to power Graphite Reviewer | 2024-12-11 |
| GT17 | AI code generation will remain fragmented | 2024-11-25 |
| GT18 | Introducing Graphite Reviewer | 2024-09-30 |
| GT19 | How AI code review reduces review cycles | 2024-09-23 |
| GT20 | What if you could get instant feedback on your code? | 2024-08-28 |
| GT21 | The new developer toolchain for the age of AI | 2024-08-07 |
| GT22 | Not Rocket Science – Bors and Google's TAP | 2024-07-27 |
| GT23 | State of code review 2024 | 2024-07-17 |
| GT24 | Going from 0 to 1: better unit tests | 2024-06-12 |
| GT25 | Parallel CI is now generally available | 2024-06-03 |
| GT26 | Cheaper CI & faster merging with batching (+ CI optimizer, 05-09) | 2024-05-08 |
| GT27 | How Google does code review | 2024-05-08 |
| GT28 | From the 80's to 2024 – how CI tests were invented | 2024-04-10 |
| GT29 | The core principles of building a good AI feature | 2024-03-20 |
| GT30 | Moving fast breaks things: staging environment | 2024-02-17 |
| GT31 | PR practice guides: Types of code reviews; Improving team velocity; 8 best practices | 2024-01-19 to 02-27 |
| GT32 | Q1 launch week: Accelerating code review | 2024-01-15 |
| GT33 | The practical and philosophical challenges of AI code review | 2024-01-03 |
| GT34 | Empirically supported code review best practices | 2023-12-18 |
| GT35 | Large PRs: "Your GitHub PR workflow is slowing everyone down" + "How large pull requests slow down development" | 2023-11-20 / 12-04 |
| GT36 | How long should your CI take? | 2023-11-01 |
| GT37 | Experimenting with AI code review | 2023-10-30 |

Brief-table posts (§2 end): stacked-prs, stacking-for-distributed-teams, code-freeze, frozen branches, build-or-buy, AWS deploys, CI/CD for trunk-based dev, ban merge commits, trunk-based dev (skim), best PR title (skim), Linear integration, Hack the North, changelog 11-04-2025, and 8 more skims.

**Skipped (33), with reasons.**
- Changelogs 10/2023, 3/13, 3/27, 4/10, 4/25/2024: feature lists that newer posts supersede.
- Company or event posts: Developer Week 2025, startup program, launch-week roundups (1/2024 and 5/2024), Q1 "What's next" / "Making stacking seamless" / "Stacking with the tools you love": announcements with no technique or data.
- Git and industry history: Linus and BitKeeper, Understanding Git, git reflog, Facebook and Git, Perforce to Piper, GitHub vs SourceForge, Stack Overflow vs Experts Exchange.
- Unrelated engineering: landing page redesign, CRA to App Router, AWS vs Vercel, Node.js server optimization, call-site SQL attribution, post-mortem 12-06-23.
- Culture and career: trust, eng estimations, Mom Test, startup learning curve, onboarding roulette.
- Product tie-ins: Tower stacking, GitHub PR filters/inbox, Phabricator alternatives list.

---

## 2. Per-post digest

**GT1 Code Tours** (2026-04-01)
- Turns a PR into a guided, narrative walkthrough built from the description, conversation, stack, and code. The narrative sits beside the diff.
- Planned follow-ups: "visual artifacts" (the visual impact of a change) and "test validation" (show which tests validated the change).
- Reviewers, not only authors, can now start a Cursor Agent from the PR to run scripts, execute tests, and make tool calls.

**GT2 Cursor Cloud Agents in Graphite** (2026-03-02)
- Each Cursor cloud agent gets its own VM to build, test, and record a **video demo**, then opens a draft PR.
- The in-PR agent sees the diff, CI failures, and reviewer comments, and **commits directly to the branch**.
- Users can pick any coding model Cursor supports for the task.

**GT3 Graphite joins Cursor** (2025-12-19)
- "Previously, we were limited by how quickly we could write code, but now the bottleneck is how quickly we can review it."
- Plan to combine Graphite's AI Reviewer with Cursor's Bugbot. Graphite remains a separate product.
- Users: "hundreds of thousands of engineers" (SR).

**GT4 PR page redesign** (2025-11-24, changelog 2025-11-20)
- Floating comments sit beside the code rather than inline. Reviewers read top to bottom and lose their place when threads break up the diff.
- A stack sidebar was tried and dropped because PR cards were too dense; the horizontal stack view stayed.
- Action Cards surface PR status and open Agent Chat to fix the issue. One panel shows CI summary, logs, and jobs. PR page loads 2x faster (SR, 09-17 changelog).

**GT5 Graphite Agent and pricing** (2025-10-07, changelog 2025-10-16)
- Diamond (reviewer) and Chat merged into "Graphite Agent". The pitch: other reviewers "stop at comments", while the agent fixes issues, updates PRs, and merges.
- Workflow framing: Create (stacked PRs) → Review (AI review + chat) → Merge (stack-aware queue).
- Pricing: Hobby free; Starter $20/user/mo; Team $40/user/mo with unlimited reviews, described as "the same price as Cursor BugBot".

**GT6 Code search for Graphite Chat** (2025-09-09)
- The requirement is fast search **at any commit**. GitHub and Sourcegraph index only the default branch, but agent searches "almost never target main".
- `git grep` on EFS/EBS was fast (<10 s) only with the repo fully in page cache. EBS ran ~3x faster than EFS on I/O-bound searches (react-native: 1.54 s vs 4.56 s).
- Elasticsearch took <500 ms for one commit, but indexing every commit means "tens of billions of documents".
- Solution: store Git-style **blobs and trees** in a document DB (Turbopuffer). Each query runs in parallel: (1) fetch the tree for the commit SHA, (2) find blobs matching the term. Intersect in memory as a stream. Total documents ≈ 3x the working-tree file count.
- Live index: "tens of millions" of files, median latency <100 ms (SR). Replaced GitHub API calls, removing rate limits and main-only scope. Embeddings for semantic search are "on the table".

**GT7 Graphite Chat** (2025-08-18)
- Chat answers from the full codebase, linked PR history, CI, and the web. Suggested prompts: "Could this cause a race condition?", "does it have test coverage", "does it introduce regressions".
- Authors can have it resolve reviewer comments and "even fix CI" with one click, and edit, run tests, and commit without checking out locally.
- Agent (as Diamond) "has reviewed millions of PRs across thousands of customers" (SR).

**GT8 AI needs to review AI code** (2025-07-28)
- 30–35% of all actionable review comments at orgs using Graphite Agent come from the AI (SR; "actionable" not defined).
- Recommended split: AI does the first pass (logic, security, performance, org style); humans handle architecture and business requirements.
- Track PRs reviewed, issues found, and the rate at which AI suggestions are **accepted or dismissed**.

**GT9 Teaching Claude to stack (GT MCP)** (2025-07-24)
- Asked to refactor well-tested code, Claude produced "a few thousand lines" and **modified the test files too**. "Though the PR was 'green,' I was no longer sure it was safe to land."
- Only 24% of PRs over 1,000 lines receive any review comments (Graphite data, SR).
- A naive "use gt to stack" prompt gave PRs that didn't build: tests before their dependencies, a refactor split so no PR passed CI. Mapping Claude's Todos to one PR each fixed it (example: Wordle as 4 PRs of +268, +129, +29, +33 lines).
- GT MCP (CLI v1.6.7) lets any agent split large diffs into stacks. One user reported saving 30–45 min (SR).

**GT10 Stack-aware merge queue** (2025-06-02)
- Run CI once on the stack's top PR using a speculative `gtmq_` branch. If it passes, fast-forward merge the whole stack. On failure, bisect and evict only the failing PR and its dependents.
- Bisection isolates one failure in a 32-PR batch with 5 CI runs. Strategy choice: N-sect (parallel) when ≤8 stacks or historical failure rate >0.3; otherwise bisect.
- Flaky tests caused good PRs to be evicted, so eviction now requires **≥2 consecutive failures**. Bisection is topology-aware: it never tests a child without its parent.
- Partitioned queues route by changed-file glob, so frontend doesn't wait on backend CI. Repo-level and stack-level locks with expiry; orphaned draft PRs cleaned up after 2 h.
- Customer results (SR): Ramp 74% lower median time between merges; Asana 7 h/engineer/week saved and 21% more code; Shopify projects 15–25% CI savings.

**GT11 Monorepo organization** (2025-05-27)
- Owned interfaces (`GitHubClient`, `AIClient.generateCodeReview(diff, context)`) let them swap LLM providers without touching business logic. Uses the Vercel AI SDK.
- Shared Zod schemas give compile-time and runtime contract checks across frontend and backend: "When you change an API contract, TypeScript tells you everywhere that needs updating."
- 15 engineers ship 1,000+ PRs/month (SR). Turbo DAG builds are 10x+ faster (SR).

**GT12 Making AI review available** (2025-03-26) / **GT13 Series B + Diamond** (2025-03-17)
- Diamond became standalone (any GitHub repo), free up to 100 PRs/month. Reviewing "tens of thousands of PRs every week" (SR).
- Features: custom rules (import a style guide), **comment settings that filter unwanted comment types**, codebase awareness, review insights by issue category, one-click fixes.
- Platform claims: summarizes PRs, turns comments into suggestions, and "self-heals failing CI".

**GT14 AI won't replace human review** (2025-03-17)
- AI handles objective checks: style, common bugs, race conditions, SQL injection and buffer overflows, test-coverage adequacy. Cites Google Tricorder for offloading nits.
- Humans handle business context (it lives in tickets and meetings), architecture (models do "local" reasoning and struggle with "global" reasoning), and mentorship.

**GT15 How Graphite ships fast** (2024-12-14)
- Q4 2024, 9 engineers (SR): median 324 PRs/engineer/quarter vs industry 33; P90 489 vs 114. Median PR 34 lines vs 36. Median time-to-merge **5.7 h vs industry 2.7 h**.
- Their reading: throughput comes from stacking, not from smaller PRs or faster merges.

**GT16 Why Claude for Graphite Reviewer** (2024-12-11)
- Early models gave "confident but incorrect suggestions"; trust "erodes quickly" with false positives. Claude had "the lowest false positive rate of any LLM we tested" (no numbers or method given).
- With Claude (SR): feedback loop 1 h → 90 s ("40x"); **96% positive feedback** on AI comments; **67% implementation rate** of suggested changes.
- Anthropic advised on prompt caching, retries, and input structure.

**GT17 AI codegen will remain fragmented** (2024-11-25)
- Code arrives from many models and interfaces, so standards cannot be enforced at creation. "The single point of leverage is the code review process."
- Split large agent changes into stacks so each PR is a testable, reviewable unit.

**GT18 Graphite Reviewer launch** (2024-09-30)
- **<3% false-positive rate** "across tens of thousands of code changes" (SR; FP definition not given).
- Codebase awareness uses **RAG over the team's past pull requests**. Custom rules include **regex matching for secrets and API keys** plus plain-English prompts.
- Price at launch: $20 per active committer per month.

**GT19 AI review reduces review cycles** (2024-09-23)
- Main lever is time to first review. Median engineer at a large company takes ~13 h to merge (cites GT23).
- Example: AI flags missing input validation before a human looks, "eliminating an entire review cycle". Useful across time zones.

**GT20 Instant feedback** (2024-08-28)
- Median time to merge across Graphite customers ≈ 14 h (SR). Copilot users complete tasks 55% faster but open only 8.7% more PRs (GitHub data, cited).
- An internal AI reviewer ran on every PR in their TypeScript monorepo from early 2024. It started as a "nuisance" and after a few months of iteration had "high accuracy" (no figure).
- Good at nits (spelling, **flipped booleans**, wrong imports) and some logic errors in function definitions.
- North star: AI helps the author iterate before a human approves, "less like AI autonomously approving PRs".

**GT21 The new developer toolchain** (2024-08-07)
- Engineers spend 30%+ of their time in the outer loop. Cited studies: 52% of ChatGPT Stack Overflow answers had bugs; Copilot produced vulnerable code in 40% of CWE Top-25 scenarios.
- Bot reviewers: "the majority of such generated comments are noise". Even an actionable comment needs another commit-test-review loop.
- The goal is AI that summarizes, prioritizes "the riskiest and most critical pieces", and self-heals PRs through CI and merge.

**GT22 Bors, TAP and merge queues** (2024-07-27)
- Merge skew: P(collision) = 1 − (1 − p)^n. With p = 0.01: 10 merges/day → 0.68 expected daily breakages; 40/day → 9.69.
- A serial Bors queue with 15-min CI caps throughput at 32 PRs per 8-hour day. Google TAP keeps two trunk pointers (head and last-green), batches post-merge tests, and auto-bisects and rolls back.

**GT23 State of code review 2024** (2024-07-17)
- Dataset: ~6M PRs merged Jul 2023–Jun 2024, >35k engineers; company size proxied by GitHub org (Graphite-synced orgs only).
- Median time to merge: 4 h at small companies (≤20 engineers) vs 13 h at large (50+). Small companies are 4.5x more likely to skip review.
- PRs ≤50 lines: ~100 min publish-to-merge; reviewed PRs over 50 lines: 19 h median. Half of all PRs are under 50 lines.
- Monorepo orgs merge 64% slower (16.4 h vs 10 h). Median CI per PR is 22 min; top-quartile CI is 76 min (small orgs) and 130 min (large).
- Merge queue adoption: 21% at ≤100 contributors, 50% at 100+.

**GT24 Better unit tests** (2024-06-12)
- Hard-to-test code signals a design smell. Separate pure logic from side effects (Command/Query); `prMergeBlockers(pr)` becomes testable without stubs.
- Use builder DSLs for fixtures, parameterized test tables, and assertions that print the whole actual value on failure.

**GT25 Parallel CI** (2024-06-03)
- Speculative groups (A, A←B, A←B←C) run CI in parallel. A failure evicts only that PR and cancels the groups that include it.
- Early adopters (SR): 1.5x faster merges (p95 −33%, p75 −26%); stack-heavy orgs 2.5x (p95 −60%). Internally, parallelism 4 halved merge time to ~12 min, of which 10 min is CI.

**GT26 Batching + CI optimizer** (2024-05-08/09)
- Batching trades per-commit revertability for lower CI cost. Head is green, but individual commits may not build.
- CI optimizer runs CI on only the bottom (or top and bottom) of a stack. Fast suites (lint) run on every update and slow suites (e2e) only before merge. Restacks change SHAs and trigger full rebuilds even when no code changed.

**GT27 How Google does code review** (2024-05-08)
- Three approvals: LGTM, code owners, readability. "Attention set" makes it explicit whose turn it is. Diff between CL versions; moved lines rendered differently; Done/Ack buttons.
- Google study: median review latency under 4 h; first feedback under 1 h for small changes, ~5 h for very large ones.

**GT28 History of CI** (2024-04-10)
- Practitioner target: 10–15 min CI. Test spending can reach 10–20% of cloud cost.
- Speculates AI review as a "fourth" speedup (probabilistic prediction) that scans diffs in about 10 s and "doesn't require completeness".

**GT29 Core principles of AI features** (2024-03-20)
- A good AI feature has lowest friction and/or **queries unique data**. AI PR descriptions use recursive summarization for oversized diffs, rated 3/5 friction and 2/5 data.

**GT30 Staging bake** (2024-02-17)
- Staging deploy → 1-hour bake → auto-promote. Employees dogfood staging. The pipeline is paused for regressions 1–2 times a week. Production regressions fell by about three-quarters (SR).
- Running migrations twice (staging, then prod) acts as a rehearsal. Prod promotion is locked outside weekday 9–6. Do not lock the pipeline for more than ~12 h, or changes pile up.

**GT31 PR practice guides** (2024-01-19 to 02-27)
- Teams averaging ~50-line PRs ship 40% more code than teams with 200+ line PRs (SR). Median time-to-merge is 3x higher for 5+ files (1.5M PRs).
- Cited studies: checklists raise defect detection by 66.7%; inspections find >60% of defects vs 25–45% for testing.
- PR template sections: What changed / Why / Risks / Testing / Checklist. SLAs: first review ≤24 h, publish-to-merge ≤5 days. Domain-matched reviewers.

**GT32 Accelerating code review** (2024-01-15)
- Automations: if-this-then-that rules on author, path, labels, and base branch that assign reviewers, label, and comment. Goes beyond CODEOWNERS.
- AI code suggestions turn an English comment into a one-click code suggestion.

**GT33 Practical and philosophical challenges of AI review** (2024-01-03; marked outdated 2024-08-29)
- GPT-4 on raw diffs gave mostly false positives from misread lines or missing out-of-diff context.
- Fixes tried: an "AI review guide" per team; **RAG vector search for similar code**; and **function calling where the model reports a concern only if it calls the function**. The last cut hallucinated vs real concerns "from about 9:1 to 1:1". Signal to noise was still not good enough.
- Limits of AI review: incremental-improvement judgment, author trust, reviewer learning, **accountability**, **security against bad actors**, and **risk context** (live vs demo, table size, past incidents). Average review takes ~18 h.

**GT34 Empirical best practices** (2023-12-18)
- Keep PRs under 200 lines changed; reviewer attention drops beyond 400 LOC (SmartBear/Cisco); review sessions of at most 60 min.
- 75% of review-found defects affect evolvability, not function. Limit nits: a "no nits" policy or a "nit keeper" role.
- Metrics: publish-to-merge time, review cycles until merge, reviewer workload.

**GT35 Large PRs** (2023-11-20, 2023-12-04)
- Over 1.5M PRs, a small rise in files changed can double time-to-merge. Recommended limit: ≤3 files.
- **Review time per file falls for PRs with 8+ files**, so reviewers skim. 24% of 1,000+ line PRs get any comment. 500+ line PRs take ~9 days to merge on average. The average GitHub PR is 900+ lines (SR).

**GT36 How long should CI take** (2023-11-01)
- Each extra 5 min of CI adds over 1 h to average time-to-merge (correlation, Graphite data). Rule of thumb: 1–2 h saved per 5 min cut.
- The most PRs merged per author per week came at 15–30 min CI. Their hypothesis: very fast CI marks simpler products.

**GT37 Experimenting with AI code review** (2023-10-30)
- Principles: label human vs generated content; **start with analysis, not generation**; dogfood.
- Worked: generated PR descriptions (>50% of PRs had empty descriptions); comment→suggestion translation; codebase Q&A with cited sources (more thorough than peer answers in a blind comparison); AI review checks for test coverage, conventions, duplication, and related past PRs. Review posts in about 30 s vs 5–10 min CI, so humans see the second or third draft.
- Failed: generated docs were ~5% hallucinated, which "had a corrosive effect" on trust. English lint rules had a ~5% FP rate, but because violations are rarer than that, **a flag was ~50/50 wrong**.
- Takeaways: compare against existing implementation context; prefer async and opt-in; trade some recall for fewer false flags. Author disagreement left on a flagged concern sped up the human's approval.

**Brief table (lower relevance)**

| Post (date) | Key point |
|---|---|
| How to use stacked PRs (2024-01-08) | Team standards: PR <200 lines, review turnaround <24 h; `gt modify` restacks children automatically |
| Stacking for distributed teams (2025-02-09) | Parallelize review and development across time zones; split frontend/DB/API so domain experts review each layer |
| Beating code freeze (2024-12-23) | Stack under 50 lines/PR while frozen; a rollback of 40 lines is simpler than 400 |
| Frozen branches (2025-09-17) | `gt get` fetches teammate branches read-only; restack skips frozen branches |
| Build or buy (2025-04-16) | Vendor pros and cons; mentions Insights metrics (review response time, PRs merged per person) |
| AWS deploys (2024-05-28) | One-box → waves → bake; auto-rollback on alarms incl. dependencies; deployment blockers in off-hours |
| CI/CD for trunk-based dev (2023-11-29) | <10 min CI; path-filtered selective tests; cancel-in-progress; fail-fast; 20-min job timeouts |
| Ban merge commits (2023-11-06) | >60% of large repos (>10k PRs; n=217) ban merge commits (SR); squash-rebase makes reverts trivial |
| Trunk-based dev (2023-11-13, skim) | PRs based on trunk merge ~10 h faster; ~80% of 2023 PRs based on main (SR) |
| Best PR title (2023-09-27, skim) | 2M PRs: short titles get reviewed faster; median engineer has 3 PRs awaiting review (SR) |
| Linear integration (2025-03-18) | Linked issues in the PR sidebar; create or link issues from the PR |
| Hack the North (2025-10-14) | Students given Graphite Agent; no data |
| Changelog 11-04-2025 | Domain move; swap github.com→graphite.com in any PR URL |
| Skims: PR stats, keeping code simple, Phabricator workflow, Phabricator vs GitHub, merge strategy, auto-rebase, collaboration, outgrow GitHub, 10k devs/monorepo | Restate stacking, Google tooling (GT27), and metric definitions; one cited claim that PRs <400 LOC have a higher defect discovery rate |

---

## 3. Cross-cutting themes

**T1. PR size dominates review quality and speed (GT23, GT31, GT34, GT35, GT9).**
- The data all points one way: ≤50 lines merges in ~100 min, and larger PRs take ~19 h. Per-file attention collapses at 8+ files. Only 24% of 1,000+ line PRs get any comment.
- Large PRs are therefore under-reviewed by humans. That is where an automated reviewer adds the most marginal value, and where a rubber-stamp is most likely.
- This matches GR G16 (larger PRs merge faster, possibly from lazier review). Do not lower scrutiny for large PRs.

**T2. Merge speed is a misleading quality proxy (GT15, GT36, GT23).**
- Graphite's own team merges more slowly than the industry median (5.7 h vs 2.7 h) yet ships 10x the PRs (SR). Small companies are fastest partly because they skip review (4.5x).
- Time-to-merge alone rewards skipping review. Pair it with an outcome metric.

**T3. Stacks change what "the diff" is (GT10, GT25, GT26, GT9, GT17).**
- Each PR in a stack is diffed against its parent branch, not main. CI runs on the stack head. Restacks change SHAs without changing code.
- An agent-produced change should be a stack of buildable units; naive splitting yields PRs that don't build.
- A contract change in PR k can break callers added in PR k+1, or callers fixed only in k+1.

**T4. Speculation, bisection, and flake tolerance (GT10, GT22, GT25).**
- Merge queues treat CI as noisy evidence: speculative runs, topology-aware bisection, and eviction only after ≥2 consecutive failures.
- The same discipline applies to our autonomous tester's findings.

**T5. AI review in the merge flow moved from comments to actions (GT5, GT7, GT2, GT1, GT13).**
- 2023–24: comment bot plus comment→suggestion. 2025: chat that fixes CI and commits to the branch. 2026: reviewer-side agents that run tests, and planned "test validation" in Code Tours.
- Graphite now lets the agent commit. Our spec (D14) forbids automatic fixes.

**T6. False positives set the ceiling (GT16, GT18, GT33, GT37, GT21).**
- Every AI-review post centres on FP rate. Base-rate math decides trust: a 5% FP rate is 50/50 precision when real violations are rarer than 5% (GT37).
- Structural fixes beat prompt fixes: "only report via a function call" took FP:TP from 9:1 to 1:1 (GT33).
- Graphite's headline numbers (<3% FP, 96% positive, 67% implemented) give no definitions, denominators, or method.

**T7. Context means past PRs plus commit-accurate code (GT6, GT18, GT33, GT37, GT29).**
- Context sources: RAG over past PRs, vector search for similar code, comparison to existing implementations, and search at the PR's commit (not main).
- "Unique data" (PR history, review outcomes) is the moat (GT29).

**T8. Humans keep accountability (GT14, GT20, GT33, GT21).**
- Graphite consistently rejects AI auto-approval. Named reasons: accountability, security against malicious authors, author-trust calibration, and risk context.
- Contrast with Greptile's auto-approval aspiration (GR G1, G20).

**T9. Measurement Graphite uses (GT8, GT13, GT31, GT34, GT27).**
- Accept/dismiss rate, comments by category, share of actionable comments from the AI, publish-to-merge time, review cycles until merge, time to first review, and Google's "attention set".
- No post reports AI-review recall, or precision with a stated method.

---

## 4. Mapping to our system

All suggested actions are **Proposed**.

| Finding | Source | Relevance to our pipeline | Suggested action |
|---|---|---|---|
| Function calling ("report only if you call the tool") cut FP:TP from ~9:1 to ~1:1; prose completion invents concerns on clean PRs | GT33 | `bugs` JSON, review layers | Proposed: emit each bug through a non-forced `report_bug` tool call. "No call" yields `{"bugs": []}`. Add known-clean PRs to the §5 dataset to measure invented concerns |
| A ~5% FP rate gave ~50/50 precision because true violations were rarer | GT37 | Review layers, measurement | Proposed: report per-category precision against each category's base rate on the §5 dataset; gate posting per category on precision, not FP rate |
| ~5% hallucinated generated docs destroyed trust in the whole doc set | GT37 | Context selection (living wiki) | Proposed: every wiki entry cites a source (commit, PR, incident ID); the agent may not cite uncited entries in `comment` |
| Code search must work at arbitrary commits; blob+tree index, ~3x file count, <100 ms median (SR) | GT6 | Context selection, call graph | Proposed: index wiki and indexer content by blob SHA plus tree per commit so §3.2 and Find Context resolve against `head_ref`, not main. Compare with agentic grep in the sandbox (§12 Q21) |
| RAG over past PRs and similar-code retrieval as review context | GT18, GT33, GT37 | Context selection | Proposed: add "past PRs touching the same symbols" (from §3.1 `changed_symbols`) as a Find Context source |
| Each stacked PR is diffed against its parent; CI runs on the stack head | GT10, GT25, GT26 | AST diff, contract check | Proposed: set `base_ref` to the PR's actual base branch. When the base is not trunk, also run §3.3 against trunk and flag call sites that bind only in an upstack PR |
| AI split stacks that didn't build (tests before deps, refactor split across PRs) | GT9 | Contract check, autonomous testing | Proposed: for stacked PRs, run the binding check per PR in stack order; a PR whose own head fails to bind is a `contract-mismatch` even if the stack head passes |
| Claude rewrote tests along with code; CI "green" but unsafe | GT9 | Risk scoring, review layers | Proposed: when one PR changes both a function and its tests, raise the §7.2 verification-gap factor and have the verifier check that the changed assertions still test the base behaviour |
| ≥2 consecutive failures before eviction to absorb flaky tests | GT10 | Autonomous testing | Proposed: a test-derived finding must reproduce on 2 consecutive head runs and must not reproduce on base (§2.5) before it reaches the verifier |
| N-sect vs bisect chosen by batch size and historical failure rate | GT10 | Autonomous testing, cost | Proposed: when several flows fail, bisect the changed symbols to attribute the failure; record attribution in the ledger |
| Merge-queue partitions route by changed-file glob | GT10 | Review layers (hard blocks) | Proposed: reuse one glob config for hard-block areas (§7.3), flow selection (§2.5), and per-area `review.md` rules (D10) |
| Large PRs get skimmed: per-file time falls at 8+ files; 24% of 1,000+ line PRs get any comment (SR) | GT35, GT9 | Risk scoring | Proposed: add files-changed ≥8 or lines ≥1,000 as a change-surface input; never let size lower depth (consistent with §7.2 rev 2) |
| AI review posts in ~30 s vs 5–10 min CI; feedback before the human sees the PR | GT37, GT19 | Review layers, cost | Proposed: post Analyze-bugs findings as soon as they are verified, before autonomous testing completes, but keep "review incomplete" status until all decision-critical branches return (§2.2) |
| Staging bake: 1 h bake, regressions down ~75% (SR); migrations run twice | GT30 | Autonomous testing | Proposed: for `data-integrity` candidates, apply the PR's migration in the sandbox against a sanitized snapshot before posting |
| Graphite Agent commits fixes and fixes CI on the branch | GT5, GT7, GT2 | Review layers (D14) | Proposed: keep D14 (suggestions only). Record that the leading commercial reviewer diverges, and revisit only with eval evidence |
| Comment→suggestion translation; one-click fixes | GT32, GT37 | `bugs` JSON | Proposed: keep `diff_fix_suggestion` minimal and apply it in the sandbox before posting (D3); measure the acceptance rate separately from the addressed rate |
| Comment settings filter unwanted comment types; insights by issue category | GT13, GT12 | Review layers (D4), measurement | Proposed: expose per-repo `bug_category` enable/disable in `review.md`; report addressed rate per category |
| Accept/dismiss rate; "30–35% of actionable comments from AI" (SR) | GT8 | Measurement (§9) | Proposed: add "share of addressed comments on a PR that came from our reviewer" to §9 as a secondary metric |
| Author disagreement on a flagged concern sped up the human's approval | GT37 | Findings ledger (D9, D11) | Proposed: store author replies on a finding; label "intentional" from the reply and surface it to the human reviewer instead of re-posting |
| Regex rules for secrets and API keys alongside LLM rules | GT18 | Review layers (layer 1) | Proposed: run deterministic secret and pattern rules before the LLM; LLM `security` findings must not duplicate them |
| Accountability, malicious-author defence, author trust, and risk context argue against AI approval | GT33, GT14, GT20 | Risk scoring (D2) | Proposed: cite as further support for D2 (our approval never counts). Add "first-time author in path" as a risk input |
| Selective CI: fast suites always, slow suites before merge; restacks change SHAs without code changes | GT26, GT28 | Cost, findings ledger | Proposed: key the ledger and the re-review trigger on the tree or patch-id, not the head SHA, so a restack with no code change does not trigger a new review round (D11) |
| Graphite's median time-to-merge is slower than industry while throughput is 10x (SR) | GT15, GT23 | Measurement | Proposed: never report time-to-merge without the addressed rate and the revert rate |
| Models do "local" reasoning and miss "global" architectural effects | GT14 | Call graph | Proposed: include the §3.2 caller set (`in_diff: false`) in the verifier's context by default; this is our counter to the local-only limit |
| Code Tours plans to surface "tests that validated the change" | GT1 | Evidence (§8) | Proposed: the HTML evidence page lists, per finding, which tests and flows were run and their result |

---

## 5. Open questions and gaps

1. **Source access.** All post URLs 404 on the live site as of 2026-11-09. Content comes from 2026 Wayback snapshots, which may differ from what was originally published. Graphite may have retired the blog after the Cursor deal (GT3); that is unconfirmed.
2. **Undefined metrics.** <3% FP (GT18), 96% positive feedback and 67% implementation (GT16), and 30–35% of actionable comments (GT8) give no definitions, denominators, or labelling method. The 67% matches the uReview reference in spec §9 by coincidence; do not treat it as corroboration.
3. **No recall data.** No Graphite post reports bugs missed, or performance on a known-bug set. This matches GR and CB (spec §12 Q15).
4. **Baselines drift between posts.** Median time-to-merge appears as 13 h (GT23, large companies), ~14 h (GT20, Graphite customers), ~18 h average review (GT33), 2.7 h (GT15, "industry"), and 1.34 days to review (PR-title post). Populations differ; do not compare them.
5. **Selection bias.** The datasets in GT23, GT35, and GT36 are Graphite-synced GitHub orgs, which skew toward teams that adopt review tooling. GT36 is correlational.
6. **Reviewer architecture is opaque after 2024.** Posts describe features, not mechanisms: no verifier, voting, dedup, or severity method. GT33 and GT37 (2023–24) are the only technical accounts, and GT33 is marked outdated.
7. **Agent writes to branches.** GT2, GT5, and GT7 let agents commit and "fix CI". No post addresses prompt injection via PR content, or the credentials the agent holds (compare spec D5, D6).
8. **Stacked-PR review semantics.** No post says how the AI reviewer handles a stack: per PR against the parent, or the whole stack. Our §3 needs a decision (base branch vs trunk), and so does the §5 dataset.
9. **Bugbot merge.** GT3 plans to combine Graphite Reviewer with Cursor Bugbot. No follow-up post in the index describes the outcome.
10. **Model choice.** GT16 says Claude had the lowest FP rate of the models tested in 2024; GT2 lets users pick any model. No comparative data is given, so it does not settle spec D12 (model inversion).
11. **Skimmed posts.** 11 posts were read by keyword skim only (marked "skim" in §2). Re-read them before citing any figure from them.
