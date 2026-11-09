# DeepSource Blog: Research Digest for the AI Code Review Project

Status: research input. Audience: agents implementing `docs/ai-code-review.md` (the spec).
Conventions: **SR** = self-reported by DeepSource (a commercial vendor), unaudited. **Proposed** = our suggestion, not decided.

## 1. Purpose, index, sources

Purpose: pull techniques and figures from DeepSource's blog that bear on our pipeline (AST diff, call graph, contract check, `bugs` JSON, review layers, autonomous testing, context selection) and mark which we should adopt.

Index read: https://deepsource.com/blog (single page, no pagination, 148 posts, Nov 2018 to Sep 2026). Read 2026-11-09.

Method and caveats:
- Each post was fetched with WebFetch. The tool returns a model-processed extraction, not verbatim text. Treat quotes and figures as high-confidence but re-check before citing externally.
- Page content was treated as untrusted data. No embedded instructions were followed.
- Fetch failures: none.
- Nearly every page carries "82% accuracy on real vulnerabilities" in site-wide JSON-LD metadata. It is SR marketing, not in any article body, with no method attached. Ignore it as evidence.
- Extraction inconsistencies spotted: the Reports post says "six reports" but five were listed; the MCP post says "30 tools across 8 categories" but lists 6; the Log4Shell post names rule JAVA-A0122 but links JAVA-A1022.

### 1.1 Posts read (55)

| # | Title | Date |
|---|-------|------|
| 1 | DeepSource is joining Harness | 2026-09-09 |
| 2 | DeepSource MCP Server | 2026-04-07 |
| 3 | Your AI toolchain is now an attack surface | 2026-03-25 |
| 4 | BYOK for AI Review | 2026-03-24 |
| 5 | Code review for AI agents | 2026-03-05 |
| 6 | Every AI code review vendor benchmarks itself, and wins | 2026-02-26 |
| 7 | DeepSource: Next | 2026-02-24 |
| 8 | Hybrid AI Agent for Secrets Detection | 2025-11-04 |
| 9 | Introducing, Autofix AI | 2025-04-26 |
| 10 | Introducing, DeepSource SCA | 2025-04-08 |
| 11 | Why we're building DeepSource SCA | 2025-01-14 |
| 12 | From Zero To Secure | 2024-02-28 |
| 13 | Introducing, Community Analyzers | 2024-01-03 |
| 14 | Introducing, DeepSource Runner | 2023-08-07 |
| 15 | Introducing support for Monorepos | 2023-08-02 |
| 16 | Improve Security Posture with SAST | 2023-04-19 |
| 17 | Introducing, DeepSource Reports | 2023-02-24 |
| 18 | Balancing code quality and developer productivity | 2023-02-16 |
| 19 | DeepSource's Guide to SAST | 2023-02-02 |
| 20 | Problems & pains in parsing: a story of lexer-hack | 2022-11-06 |
| 21 | (Re)Introducing DeepSource | 2022-10-13 |
| 22 | Engineering Manager's Guide to Static Analysis | 2022-07-24 |
| 23 | Lightweight linting with tree-sitter | 2022-04-09 |
| 24 | Don't use Math.random() | 2022-03-18 |
| 25 | Rust stdlib vulnerability in fs::remove_dir_all | 2022-01-21 |
| 26 | Learn Python ASTs by building your own linter | 2022-01-14 |
| 27 | Log4Shell: Apache Log4j Vulnerability | 2021-12-14 |
| 28 | Guidelines for Java code reviews | 2021-04-02 |
| 29 | Python code review checklist | 2021-03-24 |
| 30 | Leveraging static code analysis in a Ruby CI pipeline | 2021-02-24 |
| 31 | Common Python security pitfalls | 2021-02-04 |
| 32 | Beginner's guide to JavaScript static code analysis | 2020-12-30 |
| 33 | Ruby security pitfalls | 2020-12-03 |
| 34 | Setup static code analysis for Python | 2020-08-21 |
| 35 | Code formatting on autopilot | 2020-08-06 |
| 36 | Automate code formatting with Transformers | 2020-07-08 |
| 37 | Tips for writing glob patterns in DeepSource configuration | 2020-06-24 |
| 38 | Make your Dockerfile better with DeepSource | 2020-06-18 |
| 39 | How we ensure less than 5% false positive rate | 2020-06-05 |
| 40 | Actionable issue reporting with DeepSource | 2020-05-14 |
| 41 | A hands-on introduction to static code analysis | 2020-04-24 |
| 42 | Autofix: The future of code reviews is here | 2020-03-27 |
| 43 | Release: Static analysis for Terraform | 2020-03-05 |
| 44 | OWASP Top 10: Sensitive Data Exposure | 2019-11-02 |
| 45 | Release: Static type checker in Python | 2019-11-02 |
| 46 | Release: Re-thinking test coverage | 2019-09-10 |
| 47 | OWASP Top 10: Broken Authentication | 2019-08-20 |
| 48 | Release: Ignore issues in all test files | 2019-08-06 |
| 49 | Release: Permanently silence issues in code | 2019-07-30 |
| 50 | OWASP Top 10: Injection | 2019-07-26 |
| 51 | Release: Granular analysis on diffs | 2019-07-02 |
| 52 | Release: Actions on issues, ignore rules | 2019-05-28 |
| 53 | Code review best practices | 2019-03-05 |
| 54 | The exponential cost of fixing bugs | 2019-01-30 |
| 55 | Benefits of Continuous Quality | 2019-01-23 |

(Row 1 is read for product direction only.)

### 1.2 Skipped (93), grouped, one reason per group

- **Compliance/pricing/business** (Vanta SCA, Vanta SOC2, SOC 2 Type II, Rethinking SCA Pricing, $2.6M seed, free for private repos, Startup Program, GitHub Education): no review technique.
- **SCM/platform integration launches** (Bitbucket DC, Azure DevOps, GitLab Pipelines, Bitbucket support, GitLab support, repo onboarding, improved issues interface, Pinned Reports, DeepSource Discuss): UI or integration news, no technique.
- **Language analyzer launches** (Kotlin, Swift, C#, PHP, Rust, Scala, Java, JS beta, Ruby beta/GA, Go beta/GA): announce coverage only; techniques covered by posts read.
- **Language tutorials and trivia** (Ruby WASI, Go marshalers, Ruby percent notation, Go generics, Roslyn IL interpreter, Python 3.8–3.11 news, monadic parsers, JVM invocation, Node worker threads, dynamic values, descriptors, walrus, positional-only args, Go modules, Vue key/reactivity, Django async, Python perf tips, web framework comparisons, self-referential title): language education, not review tooling.
- **Anti-pattern/gotcha lists** (Common anti-patterns in Go, 5 mistakes in Go, Python anti-patterns, Python gotchas, Python beginner mistakes, JS best practices, Ruby best practices, state machines, Python formatters list): rule-content examples; no pipeline insight beyond posts read.
- **Infra/ops** (Heisenbugs, zero-downtime k8s, Redis replication, Vault on k8s, distributed tracing, Tailscale, git profiles, git branch naming, git best practices): off-topic for review.
- **Events, podcasts, culture** (Hack 'n Camp, 2021 in review, Hacktoberfest x2, Next '21, PyCon/PyConf reports x3, Good Code Podcast 1–4, events March 2019, Masonite Q&A, Good First Issue, "Breaking builds" interview, business strategy, documentation, technical debt): no technique.
- **Older release notes on test coverage/meta config** (Release: Test coverage, meta config, Mar 2019): superseded by posts 46 and 51.

---

## 2. Per-post digest

### 2.1 AI review, agents, benchmarks (2025–2026)

**[7] DeepSource: Next (2026-02-24)**
- Hybrid engine: 5,000+ static analyzers (SR) run first and produce a high-confidence baseline; the AI agent is seeded with those findings.
- A harness builds "code intelligence stores": data-flow graph, control-flow graph, taint source/sink maps, reachability, import graph, per-PR ASTs. The agent queries them as tools instead of grep/file reads.
- AI targets what rules miss: business-logic flaws, subtle injection, cross-function and cross-file vulnerabilities.
- Claim: LLM-only review often emits zero output on real CVEs because the model skips vulnerable code; rules cannot reason past patterns.
- "Most accurate" on OpenSSF CVE benchmark vs Claude Code, Codex, Devin, BugBot, Greptile, CodeRabbit, Semgrep (SR, self-run, no score in body).

**[6] Every AI code review vendor benchmarks itself, and wins (2026-02-26)**
- No shared benchmark exists. Vendor figures: Greptile 82% recall on 50 PRs/5 repos, re-run by Augment at 45%; Qodo 60.1% F1 (synthetic bugs, LLM judge); Augment 59% F1; Propel 64% F1; Macroscope 48% detection (118 bugs, 45 repos); Entelligence 47.2% F1 (67 bugs). All SR by each vendor.
- DeepSource's own: 165 real CVEs from OpenSSF, filtered (pre/post-patch commits exist, CWE present, files < 1,000 lines), judged blind by Claude Opus 4.5 (SR).
- Recommendations: real ground truth (CVEs, bug-fix commits), published versioned datasets, blinded judge, publish raw outputs/prompts/verdicts, ≥ 100 entries; a few edge cases swing scores 10+ points.
- Admits code-quality ground truth is unsolved; expert disagreement and no CVE equivalent.

**[5] Code review for AI agents (2026-03-05)**
- CLI with `--output json` returns issues by file, Report Card grades, CVSS-scored dependency vulns, metrics.
- Loop: agent opens PR → hybrid review → agent pulls findings via CLI → fixes → repeat until clean.
- Report Card has five dimensions: Security, Reliability, Complexity, Hygiene, Coverage. Argues a graded summary beats a flat issue list for agents.
- Filters by file, severity, category (`--severity critical --category security`).

**[2] DeepSource MCP Server (2026-04-07)**
- MCP server exposes findings, PR report cards, dependency vulns with reachability, coverage, compliance reports; OAuth auth.
- Tool categories: code issues (with source context per occurrence), PRs (issues raised and resolved), metrics with trends, OSS vulns (CVSS, EPSS, fix version, reachability), reports (OWASP Top 10, SANS Top 25), suppression-rule config.
- Two loops: autonomous PR fix loop; vuln remediation loop that skips unreachable vulns.

**[4] BYOK for AI Review (2026-03-24)**
- Two model deployments required: flagship for Autofix and AI review; smaller model for descriptions, filtering, summarisation. Stated aim: lower token cost and latency without quality loss (no figures).
- Providers: Claude (Bedrock/direct), OpenAI (Azure/direct), Gemini (Vertex/direct). Switching is a key change.
- Non-AI features (static analysis, SCA, secrets, IaC, Report Card) are model-independent.

**[8] Hybrid AI Agent for Secrets Detection (2025-11-04)**
- Two stages: fast regex/pattern candidates, then an AI classifier (Narada, described as open source) judges each with semantic context.
- SR: false positives 222 → 16 (−93%), precision 97%, recall 96.3%. Dataset and method not described.
- Filters test credentials, example code, config templates. Names alert fatigue and trust erosion as the problem.

**[9] Introducing, Autofix AI (2025-04-26)**
- Legacy deterministic Autofix covered "perhaps 30%" of issues; LLM Autofix covers "nearly all" (both SR).
- Fix context: flagged code plus imports, related functions, project conventions.
- Safety is human-only: diff view, approve/reject per change, delivered as a PR. No automated test validation described.
- Planned: diff-level feedback with regeneration; bring-your-own-model.

**[1] DeepSource is joining Harness (2026-09-09)**
- Thesis: the bottleneck moved from writing code to verifying it; verification must live inside the delivery pipeline, not as a standalone product.
- "Deep tools too noisy, tolerated tools too shallow." Goal: review trustworthy enough to act on automatically.

### 2.2 Supply chain and SCA

**[3] Your AI toolchain is now an attack surface (2026-03-25)**
- LiteLLM 1.82.7/1.82.8 on PyPI (2026-03-24) stole SSH keys, cloud creds, tokens, k8s secrets; chain started at Trivy (03-19) then KICS (03-23).
- 1.82.8 used a `.pth` file that runs on every interpreter start; invisible to import review, static analysis of source, and `--require-hashes` (legit publish).
- No commit or diff existed: injection was between source and artifact. Code review of PRs cannot catch this.
- Recommendations: pin Actions to SHAs, no auto-update of AI toolchain in CI, monitor outbound traffic from runners, scan site-packages `.pth` files for `import`/`exec`/`subprocess`/`base64` (cheap, low false positives per author).

**[11] Why we're building DeepSource SCA (2025-01-14)**
- AST-based reachability: vulns in unused modules are de-prioritised as noise.
- Multi-variate remediation: evaluates every upgrade path for breaking-change risk and shows affected code.
- Cited: average JS package pulls 377 deps; 10% pull > 1,400 (Black Hat 2020). Customers want 10x noise reduction (aspiration, no SR claim).

**[10] Introducing, DeepSource SCA (2025-04-08)**
- Manifests and lock files are "targets"; data from NVD and others.
- "Dynamic Risk" engine: custom weighting beyond CVSS/EPSS. Reachability via static engine. Incremental PR scans flag only new packages.

### 2.3 False-positive control and scoping

**[39] How we ensure less than 5% false positive rate (2020-06-05)**
- SR: FP rate < 5%; report turnaround target < 72 h, typical 12–18 h; ≥ 50 valid FP reports resolved per week; 7 processors (3 static, 4 dynamic); analysis < 10 s typical PR, ~36 s large PR/full repo.
- Static processors: changeset-based pruning (only issues on changed code); issue+filename deny-pairs (e.g., unused import in `__init__.py`); known-FP exclusion by context and path.
- Dynamic processors: excluded globs, `skipcq` mute comments, ignore rules, test-file-only suppression.
- Relevance engine: weights issues by history (usually fixed vs often ignored) and sorts; orders, does not remove.
- Patches third-party tools (Pylint, Flake8) for known FPs; cites Google 2018 paper on FP and adoption.

**[51] Granular analysis on diffs (2019-07-02)**
- PR reports restricted to added/modified lines plus new issues those changes cause elsewhere in the file.
- Motivation: whole-file reports felt "out of context." Boy-scout rule scoped to touched lines.

**[49] Permanently silence issues in code (2019-07-30)**
- `skipcq: CODE[, CODE]` on the line or the line above; bare `skipcq` silences all and is discouraged.
- Motivating FP: `mark_safe` fine in trusted admin views, risky in user-facing ones; context the rule cannot see.

**[52] Actions on issues, ignore rules (2019-05-28)** / **[48] Ignore issues in all test files (2019-08-06)**
- Per-issue actions: change severity per repo, ignore by file glob, ignore per file, ignore repo-wide; rules auditable in settings.
- `test_patterns` in config marks tests; a curated set of issues is suppressed in tests only.

**[37] Tips for writing glob patterns (2020-06-24)**
- `exclude_patterns`, `test_patterns`; `**` must be its own segment.
- Example: `*/tests/**` → `tests/**` changed issue count ~1,700 → ~1,500 on one OSS repo. Wrong globs create noise silently.

**[12] From Zero To Secure (2024-02-28)**
- Baseline analysis: record existing issues on default branch; PRs block only on issues they introduce (example: 35 existing, 1 new → only 1 blocks).
- Quality gates start report-only, switch to blocking after weeks or months.
- Analysis runs outside customer CI; config is one `.deepsource.toml`.

**[19] DeepSource's Guide to SAST (2023-02-02)**
- SR: 4,000+ teams; > 70% had no prior SAST; FP rate < 5% "guaranteed"; > 75% of FP reports resolved within a week (last 6 months).
- Four adoption blockers: CI slowdown, false positives, legacy debt flooding PRs, weak reporting.
- No taint/data-flow detail in this post (that appears later in [7]).

### 2.4 Autofix and formatting

**[42] Autofix: The future of code reviews is here (2020-03-27)**
- Launch: 14 Python and 21 Go issues autofixable; target ≥ 30% of detected issues (SR).
- Flow: pick issue → pick files → diff preview → choose changes → PR. Only human review as safeguard.

**[40] Actionable issue reporting (2020-05-14)**
- Actionable issue fields: title, issue code, type (bug risk, anti-pattern, performance, security), analyzer, first/last seen, occurrences, description (why, cause, fix), file+line+column.
- Required checks block merge per category until fixed or marked intentional.
- Autofix motivated by repetitive work (e.g., 50 occurrences across files).

**[35] Code formatting on autopilot (2020-08-06)** / **[36] Transformers (2020-07-08)**
- Formatters (Black, YAPF, gofmt) run per PR; changes land as a separate commit, so review and revert stay clean.
- No-op runs commit nothing; conflicting formatters are re-run until stable.
- Cited: 40–80% of lifetime cost is maintenance; 150K+ GitHub commits titled "run black".

**[30] Static analysis in a Ruby CI pipeline (2021-02-24)**
- RuboCop: 7 offenses, 6 safe-autocorrectable (~70% of work removed); unsafe fixes and `Kernel#open` needed manual change.
- Pattern: apply "safe" autofixes automatically, gate "unsafe" ones behind human review.
- Linter caught a real bug (variable assigned in block, used outside).

### 2.5 Static analysis technique

**[26] Learn Python ASTs by building your own linter (2022-01-14)**
- `ast.parse`, `NodeVisitor` with `visit_<Node>`; pre/post-order via placement of `generic_visit`; `ctx` Load/Store/Del separates reads and writes.
- Checks: duplicate set items; unused variables per scope (Module, ClassDef, FunctionDef) with nested visitors.
- Report format `file:line:col: CODE: message`. AST drops comments/whitespace; use libcst for rewrites.
- Pitfall: forgetting `generic_visit` stops traversal; forgetting `fix_missing_locations` breaks compile.

**[41] A hands-on introduction to static code analysis (2020-04-24)**
- Pipeline: tokenize → parse → AST → visitor checks; some checks need two passes (collect, then evaluate).
- Known limits in toy checks: aliased imports, `locals()`, dotted imports, indirect nesting.

**[23] Lightweight linting with tree-sitter (2022-04-09)**
- Query DSL over lossless CSTs: captures, `#eq?`/`#match?`, anchors; same skill across grammars; incremental reparse.
- Limits: regex predicates can be imprecise; anchors over-match.

**[20] Problems & pains in parsing (2022-11-06)**
- Snippet-only parsing can be wrong: meaning may need whole-program info (C `(A) * B`). For C++ "just use libclang."

**[45] Static type checker in Python (2019-11-02)**
- mypy integrated as analyzer; 80+ typing issues (SR); respects repo `mypy.ini`; suppress with `# type: ignore` or `skipcq`.

**[13] Community Analyzers (2024-01-03)**
- Third-party tools run in customer CI and upload SARIF 2.1.0; gain gates, issue diffing vs main, ignore rules.

**[46] Re-thinking test coverage (2019-09-10)**
- Coverage uploaded from CI via CLI; flags uncovered lines. No diff-coverage concept in this post.

### 2.6 Security checks

**[16] Improve Security Posture with SAST (2023-04-19)**
- SAST covers secrets pattern matching, IaC misconfig (e.g., Terraform TF-AWS012), standards; run on every PR.
- Tool selection: report quality, FP/FN, speed, OWASP/CWE coverage. Onboard repos gradually.

**[43] Terraform (2020-03-05)** / **[38] Docker (2020-06-18)**
- Terraform: 30+ security checks (SR) such as public S3 ACL, `/0` inbound rules, unencrypted buckets, secrets in defaults.
- Docker: 4,000+ issues across 150+ repos (SR). Many repos had Dockerfiles but analyzer off.

**[31] Python security pitfalls (2021-02-04)** / **[33] Ruby security pitfalls (2020-12-03)**
- Python: `yaml.load` with `Loader` (RCE), `eval`/`exec` on input, `assert` for auth (stripped by `-O`), unmanaged deps. Bandit detects via AST.
- Ruby: `YAML.load`, `Kernel#open` with `|cmd`, string-interpolated SQL, enumerable sequential IDs. RuboCop security cops, Brakeman.

**[50] Injection / [47] Broken Auth / [44] Sensitive Data Exposure (2019 OWASP series)**
- Injection: concatenated SQL, EXECUTE IMMEDIATE in procs, user-supplied identifiers; use parameterisation.
- Auth: weak hashes (MD5/SHA1), session ID in URL or not rotated after login, enumerating error messages, default creds.
- Data exposure: hardcoded secrets, secrets in logs/caches, weak crypto, clients not validating certs.

**[27] Log4Shell (2021-12-14)** / **[25] Rust remove_dir_all (2022-01-21)** / **[24] Math.random (2022-03-18)**
- Pattern: within days of a CVE, ship a rule (Java JAVA-A0122/A1022 ambiguity; Rust RS-S1002), re-run analysis on all enabled repos, surface in Security category.
- Rust TOCTOU (CWE-363) flagged only on vulnerable toolchain versions: rule conditioned on version.
- `Math.random()` for secrets → use `crypto.getRandomValues()`.

### 2.7 Review process and workflow

**[53] Code review best practices (2019-03-05)**
- Small, coherent PRs; do not mix refactor, behaviour, and style; self-review; automate style and bug patterns so humans focus on design.

**[29] Python code review checklist (2021-03-24)** / **[28] Java code review guidelines (2021-04-02)**
- Python: 23 questions in 8 areas; splits automatable (lint, format, Bandit, mypy, coverage) from human (atomicity, design fit, duplication, scalability).
- Java bug patterns relevant to contracts: returning null into unboxing; exposing mutable internal lists; catch-order shadowing; `equals` without `hashCode`; "default to private, expose consciously" (public API changes are hard to undo).

**[22] Engineering Manager's Guide (2022-07-24)**
- Cited: review is 10–15% of feature time (IEEE); 3,126-project study: ~5x lower error rate, +14% productivity; Edmundson et al.: no reviewer found all injected vulns, FP correlated with TP (r = 0.39).
- Unsourced: automation handles ~50% of review; ~4 h saved per engineer per week (SR).

**[54] Exponential cost of fixing bugs (2019-01-30)** / **[55] Continuous Quality (2019-01-23)** / **[18] Balancing quality and productivity (2023-02-16)**
- Field fixes "as high as 30x" (unsourced); NIST chart cited. Stripe 2018: 17+ h/week on tech debt; tech debt wastes 23–42% of dev time (InfoQ).
- Analyse deltas before merge; automate trivial feedback.

**[32] JS static analysis (2020-12-30)** / **[34] Setup Python static analysis (2020-08-21)**
- "Testing checks if code works; static analysis checks if it is written well." `index || 3` falsy-zero bug as example.
- Sample run: 278 issues, 64 autofixable; six categories (anti-pattern, bug-risk, style, performance, security, documentation).

**[14] Runner (2023-08-07)** / **[15] Monorepos (2023-08-02)** / **[17] Reports (2023-02-24)** / **[21] (Re)Introducing (2022-10-13)**
- Runner: control plane in cloud, data plane (single binary) in customer infra; only metadata leaves.
- Monorepo mode: per-folder reporting.
- Reports: OWASP/SANS, code health trend (net new issues per week), issue distribution, issues autofixed, issues prevented.
- SR: 3,700+ teams (2022).

---

## 3. Cross-cutting themes

1. **Deterministic first, LLM second.** DeepSource's 2026 architecture runs 5,000+ rules, builds graph stores (CFG, DFG, taint, reachability, imports, ASTs), then seeds the LLM with baseline findings and exposes the stores as tools [7]. Same shape in secrets: regex candidates → AI classifier [8]. Their stated failure of LLM-only review is skipping vulnerable code entirely; of rules-only, no reasoning past patterns.
2. **LLM as filter, not only finder.** Secrets AI cut FPs 93% (SR) by classifying rule hits [8]; BYOK routes "filtering" to the small model [4]. A cheap model judging candidates is a recurring pattern.
3. **Scope to the change.** Diff-line scoping [51], changeset pruning [39], baseline vs default branch [12, 19], issue diffing for SARIF tools [13]. New issues block; legacy issues do not.
4. **False-positive control is a process, not only a model.** Explicit FP-rate target, user FP reports with SLA, deny-pairs (issue, path), test-file suppression, inline `skipcq` with codes, relevance ranking from fix history [39, 48, 49, 52]. Wrong path globs inflate noise silently [37].
5. **Autofix safety is mostly human-gated.** Every Autofix post relies on diff preview + approve + PR [9, 42]; no automated test validation is described. Ruby post splits safe vs unsafe autocorrect [30]. Formatting changes go in separate commits [35].
6. **Agent-readable output.** JSON CLI, MCP tools, Report Card grades [2, 5]; findings designed for agent loops, not just humans.
7. **Two-tier model routing.** Flagship for review and fixes, small model for descriptions, filtering, summaries [4]. Matches our §6.
8. **Benchmarks: real ground truth, blind judge, ≥ 100 items.** [6] directly informs our §5 tuning loop. Re-runs of others' benchmarks moved 82% → 45%.
9. **Reachability over raw severity.** SCA de-prioritises unreachable vulns [10, 11]; MCP vuln loop skips noise [2]. Analogous to call-graph reachability for our hard blocks.
10. **Supply chain is outside the diff.** LiteLLM attack had no PR [3]. PR review cannot see artifact tampering; needs separate controls.
11. **Rapid rule rollout per CVE**, version-conditioned rules, re-run across repos [25, 27].
12. **Gradual enforcement.** Report-only gates → blocking later [12]; onboard repos gradually [16].

---

## 4. Mapping to our system

All suggestions are **Proposed**.

| Finding | Source | Relevance to our pipeline | Suggested action |
|---------|--------|---------------------------|------------------|
| LLM seeded with deterministic findings and given graph stores as tools | [7] | Contract check, call graph, review layers | Proposed: pass §3.3 candidate bugs plus `call_sites` to the frontier model as seeded findings; expose call graph as a tool (callers-of, callees-of) rather than raw files |
| LLM-only review skips vulnerable code, emits nothing | [7] (SR claim) | Review layers, bugs JSON | Proposed: treat an empty `{"bugs": []}` with non-empty §3 candidates as suspicious; require the model to explicitly confirm or reject each candidate |
| Regex candidates → AI classifier cut FPs 222 → 16 (SR) | [8] | Context selection, bugs JSON | Proposed: add a cheap-model verify pass that labels each candidate `confirm`/`reject` before the frontier model writes comments |
| Two model tiers: flagship review/fix, small for filter/summarise | [4] | Context selection, cost (§6) | Proposed: route dedup, summary text, and FP filtering to the cheap model; keep bug decision on frontier (consistent with §6.2) |
| Diff-line scoping plus "new issues caused by the change" | [51, 39] | AST diff | Proposed: only emit bugs whose `changed_lines` overlap Step 1 `changed_lines`, except `contract-mismatch` at `in_diff:false` call sites, which must cite the changed definition line |
| Baseline: block only on newly introduced issues | [12, 19] | Review layers, risk scoring | Proposed: run §3 on `base_ref` too and drop findings already present there; supports P5 stability |
| Issue + path deny-pairs; test-file suppression via `test_patterns` | [39, 48, 37] | bugs JSON, tuning loop | Proposed: config with `test_patterns`, `exclude_patterns`, and per-category path suppressions; validate globs in CI |
| Inline suppression with explicit code (`skipcq: CODE`) | [49] | bugs JSON | Proposed: support `# review-ignore: <bug_category>` on the line or line above; ignore bare form |
| Relevance engine weights issues by historical fix/ignore | [39] | Measurement (§9), tuning loop | Proposed: feed addressed-rate per `bug_category` back into ordering and per-category confidence thresholds |
| FP report SLA, FP rate target < 5% (SR) | [39, 19] | Measurement | Proposed: set an explicit FP-rate target per category; add a one-click "wrong" reaction that writes to the tuning dataset |
| Actionable issue fields (code, type, why, fix, location) | [40] | bugs JSON | Proposed: no schema change; require `comment` to state cause, impact, and fix; consider an issue-code prefix inside `summary` only if §4.2 is amended |
| Report Card grades summarise for agents | [5] | Submit review | Proposed: render a short per-PR header (counts by category) above bug comments; keep JSON unchanged |
| JSON CLI and MCP for agent loops | [2, 5] | bugs JSON, review layers (layer 2) | Proposed: expose our `bugs` JSON via a CLI/MCP so authoring agents can run local review (spec §7.1 layer 2) |
| Benchmark rules: real bugs, blind judge, ≥ 100 items, publish verdicts | [6] | Tuning loop (§5) | Proposed: dataset from reverts/bug-fix commits only, ≥ 100 PRs, blinded LLM judge where matching is fuzzy, store per-item verdicts |
| Re-runs of a vendor benchmark dropped 82% → 45% | [6] | Tuning loop | Proposed: hold out a test split never used for prompt tuning; report both |
| Autofix safety relies on human approval; safe vs unsafe split | [9, 30, 42] | bugs JSON (`diff_fix_suggestion`) | Proposed: before posting, apply each `diff_fix_suggestion` in the sandbox, re-parse with `ast`, re-run §3.3 binding and tests; drop or flag the fix if any fails |
| Fix context = flagged code + imports + related functions + conventions | [9] | Context selection | Proposed: cheap model's context set for each candidate = changed symbol, its callers from §3.2, imports, and `review.md` |
| Snippet parsing can mislead; need whole-program info | [20] | AST diff, call graph | Proposed: keep §3.2 whole-project scan; never classify contracts from hunks alone |
| `ctx` Load/Store and per-scope visitors | [26] | AST diff | Proposed: use scope-aware visitors in Step 1 to attribute hunks to nested functions/methods correctly |
| tree-sitter queries are language-agnostic | [23] | AST diff (open Q §10.1) | Proposed: evaluate tree-sitter for non-Python Step 1 and call-site collection |
| Reachability de-prioritises unused vuln code | [10, 11, 2] | Call graph, risk scoring | Proposed: use reverse reachability from entry points to weight blast radius and hard-block detection (§7.3) |
| Taint source/sink and DFG stores | [7] | Call graph, `security` category | Proposed: later phase: add intra-procedural taint for `security` bugs (input → `eval`/SQL/`yaml.load`) |
| Python/Ruby pitfalls detectable by AST (Bandit) | [31, 33, 50] | Review layers (layer 1) | Proposed: run Bandit/Semgrep as layer-1 tools; feed hits as seeded candidates, not as posted bugs |
| `assert` as auth guard stripped under `-O` | [31, 37] | `auth-regression`, `security` | Proposed: add deterministic check for `assert` in non-test code guarding permission calls |
| SARIF ingestion of third-party tools | [13] | Review layers | Proposed: accept SARIF 2.1.0 from layer-1 tools as input candidates |
| Formatting in separate commit; style not reviewed | [35, 53] | Skip-nitpicks rule (§5.3) | Proposed: confirms our rule; reject any finding whose fix diff is whitespace-only |
| Report-only gates before blocking | [12, 16] | Risk scoring, gating | Proposed: launch in comment-only mode; turn on blocking per category once addressed rate and FP rate meet targets |
| Rule shipped within days of CVE, version-conditioned | [25, 27] | Living wiki, `known-incident-pattern` | Proposed: let wiki entries carry a pattern plus version/path conditions; re-run on open PRs when added |
| LiteLLM: tampering between source and artifact; no diff | [3] | Sandbox, autonomous testing | Proposed: sandbox installs with hash-pinned lockfiles, SHA-pinned Actions, egress allow-list; scan `.pth` files; treat dependency-only PRs as needing SCA, not LLM review |
| Runner: code stays in customer data plane | [14] | Sandbox | Proposed: keep review sandbox and model calls in our infra or BYOK path; record which provider saw code |
| Java contract patterns (null return, exposed mutable state, public API change) | [28] | Contract check, `bug_category` | Proposed: consider extending §3.3 beyond arity to return-type/nullability changes where annotations exist |
| Coverage tracked on uncovered lines | [46] | Evidence (§8), risk | Proposed: compute diff coverage of changed symbols; low coverage raises verification-gap factor |

---

## 5. Open questions and gaps

1. **No evidence of autofix validation.** No post describes testing generated fixes. Our sandbox could do it; cost and latency unknown.
2. **Hybrid benchmark numbers unpublished in-body.** [7] claims "most accurate" without a score; the 82% site metadata has no method. Need the actual dataset/verdicts before comparing.
3. **Secrets FP figures lack dataset.** 222 → 16 and 97%/96.3% (SR) have no corpus or baseline description.
4. **No quality (non-security) benchmark exists** [6]. Our bug categories (`contract-mismatch`, `missing-caller-update`) lack public ground truth; we must build our own.
5. **How the AI agent uses graph stores** (tool schemas, depth limits, cost) is not described. Relevant to our call-graph depth question (§10.6).
6. **FP measurement method** behind "< 5%" (2020) is undefined: denominator, who labels, which analyzers.
7. **Relevance engine details** (weights, decay, cold start) not given.
8. **Cross-file reasoning claims** for AI review are qualitative only.
9. **Supply-chain attacks outside PRs** [3] are out of scope for diff review. Decide who owns sandbox dependency hygiene.
10. **Extraction fidelity.** Content came via a summarising fetch tool; three internal count/ID inconsistencies noted in §1. Re-verify any figure before using it in decisions.
11. **Post-acquisition direction** (Harness, 2026-09) may change product docs; links may move.
