# AI-Assisted Code Review: Design for Implementing Agents

Status: draft, revision 2. Audience: coding agents that implement or extend this system.

Conventions:
- **Source**: stated in the source notes (architecture sketch, user spec, or reference material).
- **Proposed**: not in the source notes. Treat as a default to confirm, not a decided requirement.
- **Adopted** (§10 only): a design decision supported by a cited research source. **Proposed** in §10 means our own inference.
- **(SR)**: self-reported by the vendor or a named customer. Not independently verified. Use as orientation, never as a target.
- MUST / SHOULD / MAY keep their RFC 2119 meanings.

Revision 2 adds findings from three research digests: `docs/claude-blog-research.md` (posts P1–P16), `docs/anthropic-news-research.md` (posts #1–#26), and `docs/greptile-research.md` (posts G1–G27). Each changed section carries a "Changed in revision 2" note. Section renumbering: Open questions moved from §10 to §12 and References from §11 to §13. Question numbers Q1–Q14 are unchanged, so a digest reference to "§10 Qn" means §12 Qn.

---

## 1. Goal and problems solved

Goal (Source, sketch title block): "Codebase Context. It runs your code. The right model for each task."

The system reviews a pull request (PR) automatically and posts actionable bug findings back to it. It targets five problems (Source):

| # | Problem | Main countermeasure in this design |
|---|---------|------------------------------------|
| P1 | Missed bugs | Change-impact analysis (§3), autonomous testing (§2.5), prompt-tuning loop against known bugs (§5) |
| P2 | Missing context | MCP docs and tickets, codebase indexer, living wiki (§2.3, §2.4) |
| P3 | Expensive reviews | Cheap model for context selection, frontier model only for review (§6) |
| P4 | Nitpicky comments | Bug-only output contract (§4), skip-nitpicks rule (§5), independent verifier pass and comment filter (§10 D1, D4) |
| P5 | New bugs on every run | Fixed JSON schema and categories (§4), regression dataset (§5). Proposed: stabilise output across runs with a per-PR findings ledger (§10 D11) |

Non-goals (Source, ref. G and H): replacing human review of architecture, product behaviour, migrations, and infrastructure. Merge-ready is not launch-ready (§7.6).

Changed in revision 2: P4 and P5 rows now point to the verifier pass, comment filter, and findings ledger. All three source digests independently name the same five problems (greptile-research.md G19, "How to Build a Code Review Agent").

---

## 2. Architecture

### 2.1 Components

| Component | Responsibility | Source |
|-----------|----------------|--------|
| Webhook receiver | Receives the GitHub PR webhook and starts a review run | Source |
| Sandbox | Isolated environment that holds the review agent, PR branch, and dependencies | Source |
| Review agent | Plans the review, gathers context, analyses bugs, submits the review | Source |
| MCP connectors | Supply docs and tickets to the review agent | Source |
| Codebase indexer | Supplies codebase context to the review agent | Source |
| Living wiki | Knowledge base built from periodic codebase crawls plus reverts and incidents. Feeds a review agent in its own sandbox | Source |
| Autonomous tester | Runs the app (dev server) on the PR branch. A browser agent drives user flows and produces a bug report | Source |
| Cheap model | Selects relevant context from candidate files (the sketch names "Jev" as an example) | Source |
| Frontier model | Performs the review on the selected context | Source |
| PR commenter | Posts the review as a PR comment | Source |
| Verifier | Tries to prove or disprove each candidate bug before it is posted (§10 D1) | Proposed (rev 2) |
| Comment filter | Drops findings that match past dismissed comments (§10 D4) | Proposed (rev 2) |
| Findings ledger | Per-PR record of posted findings and their outcomes (§10 D9, D11) | Proposed (rev 2) |

The sketch labels the review agent with its two context inputs (MCP and indexer) as "model inversion". Greptile defines the term as reviewing code with a different model family from the one that wrote it (greptile-research.md G19, "How to Build a Code Review Agent"). Whether we adopt it is an open decision (§10 D12).

Changed in revision 2: "model inversion" is now defined (it was undefined, Q13). Added three Proposed components that the research-backed decisions need.

### 2.2 Data flow

1. GitHub sends a PR webhook to the sandbox.
2. The review agent receives the PR diff and runs **Plan Review**.
3. Plan Review fans out to three parallel branches:
   1. **Find Context**: the cheap model receives candidate files and selects relevant context for the frontier model. Context sources are MCP (docs, tickets), the indexer (codebase), and the living wiki.
   2. **Test user flows**: the autonomous tester (§2.5).
   3. **Analyze bugs**: change-impact analysis (§3) plus frontier-model review. Output is a list of candidate bugs, not posted findings.
4. Proposed (rev 2): **Verify**. Each candidate bug goes to an independent verifier that tries to disprove it (§10 D1). Survivors must carry a reproduction or evidence (§10 D3).
5. All branches feed **Submit review**. Submit review applies the comment filter (§10 D4) and the findings ledger (§10 D11).
6. Submit review emits the `bugs` JSON (§4). The agent renders it as a PR comment and posts it.

Proposed: Submit review MUST wait for every branch whose result is decision-critical before it emits output (§6.2, "wait for decision-critical results").

Proposed (rev 2): if any decision-critical branch fails, times out, hits its budget, or returns a model refusal, Submit review MUST post a "review incomplete" status, not `{"bugs": []}` (§10 D13).

Changed in revision 2: added the Verify step and the "review incomplete" rule. Sources: verifier agents that vote before posting (claude-blog-research.md P1, "How the Claude Code team does code review"); prove-or-disprove verification (anthropic-news-research.md #17, "Making frontier cybersecurity capabilities available to defenders"); a failed read is reported as "unreadable", never as a quiet day (claude-blog-research.md P2, "Building effective agent automations").

### 2.3 Context sources

| Source | Transport | Content |
|--------|-----------|---------|
| Docs | MCP | Design docs, coding standards |
| Tickets | MCP | Issue or ticket linked to the PR |
| Codebase | Indexer | Symbols, files, call relationships |
| Living wiki | Periodic crawl | Codebase knowledge plus history of reverts and incidents |

Proposed: a project `review.md` (ref. G) holds review rules and coding standards, and is always loaded into context. Revision 2 shapes it as described in §10 D10.

All context sources are untrusted input (§10 D5). The review agent reads them with read-only credentials.

Changed in revision 2: context is classed as untrusted, and `review.md` structure moves to §10 D10. Source: prompt injection is a named agent threat (anthropic-news-research.md #20, "Our framework for developing safe and trustworthy agents").

### 2.4 Living wiki

- Inputs (Source): periodic crawls of the codebase, reverts, incidents.
- Output (Source): wiki content that feeds a review agent running in its own sandbox.
- Proposed: index wiki entries by file path and symbol so Find Context can retrieve entries for the changed code and its callers. Crawl cadence is not decided (§12).
- Proposed (rev 2): refresh changed paths on merge to main plus a scheduled full crawl, as Greptile does (greptile-research.md G8, "Frontier Code Review Accuracy at Lower Cost with NVIDIA Nemotron 3 Ultra"). Each entry holds purpose, dependencies, past bugs, reverts, and incidents (G1, "Automating Code Validation").
- Proposed (rev 2): expose the wiki and indexer as tools the agent queries, not as pre-stuffed context. Anthropic replaced pre-indexed RAG with agentic search because the vector DB was fragile (claude-blog-research.md P16, "Seeing like an agent"). Compare both on the §5 eval (§12 Q21).

Changed in revision 2: added a refresh trigger and a retrieval mode, citing the posts above.

### 2.5 Autonomous testing

Source:
1. Check out the PR branch and install dependencies in the sandbox.
2. Start the app (dev server).
3. A browser agent drives the app. Flows depend on what changed:
   - auth code changed → run the sign-in flow;
   - checkout code changed → run the purchase flow.
4. Output: a bug report that feeds Submit review.

Proposed: map each flow to path globs in config. Capture screenshots or video as evidence (§8). Map findings to `bug_category` values `auth-regression` and `checkout-regression` (§4.3).

Proposed: DAST tooling such as OWASP ZAP (see §13) can run against the same running app. The notes list it as a reference only.

Revision 2 constraints (detail in §10 D6):
- No internet by default; allowed egress is the model API and a pinned package mirror. Each review gets a disposable instance, with credentials outside the sandbox.
- The tester receives specific suspected behaviours from Analyze bugs, not only path globs. A standalone test agent produced irrelevant, noisy tests (greptile-research.md G2, "Building TREX").
- A test-derived finding without an artifact (log, screenshot, trace, or script) is dropped (§10 D3).
- Each flow is checked on the base branch before it runs on head, so the task is known to be solvable (anthropic-news-research.md #6, "Improving our alignment and security practices").

Changed in revision 2: the source step 1 ("install dependencies") is now constrained by default-deny egress. Sources: anthropic-news-research.md #6 and #7 ("Investigating three real-world incidents in our cybersecurity evaluations"); claude-blog-research.md P3 ("Claude Code in the cloud").

---

## 3. Change-impact analysis pipeline

Three sequential steps (Source). Python's `ast` module is the specified parser (Source). Support for other languages is not decided (§12).

### 3.1 Step 1: Analyze changes

- **Purpose**: identify the exact lines and functions modified.
- **Input**:
  - `base_ref`, `head_ref` (git refs);
  - unified diff of the PR.
- **Process**:
  1. For each changed `.py` file, parse the base and head versions with `ast.parse`.
  2. Map changed diff hunks to the enclosing `FunctionDef`, `AsyncFunctionDef`, or `ClassDef` nodes by line range.
  3. Classify each symbol as `added`, `removed`, or `modified`.
- **Output** (Proposed shape):
  ```json
  {
    "changed_symbols": [
      {
        "file": "calculator.py",
        "qualname": "calculate_discount",
        "kind": "function",
        "change": "modified",
        "base_lines": [5, 9],
        "head_lines": [5, 11],
        "changed_lines": [5]
      }
    ]
  }
  ```

### 3.2 Step 2: Build call graph

- **Purpose**: map how functions interact so the system knows which code depends on the modified code.
- **Input**: the full project source at `head_ref`, plus `changed_symbols` from Step 1.
- **Process**:
  1. Scan every module and collect function and method definitions with their qualified names.
  2. Collect `ast.Call` nodes and resolve each callee to a definition where possible.
  3. Build a directed graph `caller → callee`.
  4. For each changed symbol, collect its callers (reverse edges). Proposed: limit the depth with a config value; the default depth is not decided.
- **Output** (Proposed shape):
  ```json
  {
    "call_sites": [
      {
        "callee": "calculator.calculate_discount",
        "caller": "checkout.apply_promo",
        "file": "checkout.py",
        "line": 42,
        "in_diff": false,
        "args": {"positional": 2, "keywords": []}
      }
    ],
    "unresolved_calls": 0
  }
  ```
- Constraint: report `in_diff: false` call sites explicitly. These are "callers outside the diff context", the main source of contract-mismatch bugs (Source example). Greptile calls this blast-radius analysis (greptile-research.md G1, "Automating Code Validation").

### 3.3 Step 3: Detect contract changes

- **Purpose**: check whether a function's signature changed. If it did, existing call sites may now be broken.
- **Input**: base and head `ast.arguments` for each `modified` symbol, plus `call_sites` from Step 2.
- **Process**:
  1. Compare positional parameters, defaults, keyword-only parameters, `*args`, and `**kwargs`.
  2. A contract change exists if a parameter is added without a default, removed, renamed, or reordered.
  3. For each call site of a changed contract, check whether its arguments still bind to the new signature.
  4. Each call site that fails to bind is a candidate bug.
- **Output**: candidate bugs passed to the frontier model for confirmation. Each candidate becomes at most one entry in the `bugs` output (§4), with `bug_category` `contract-mismatch` or `missing-caller-update`.
- Proposed (rev 2): expose the binding checker as a tool the frontier model and verifier can call, instead of reasoning about signatures freehand. The failing call site is the minimal reproduction cited in `comment` (§10 D3).

Changed in revision 2: added the binding checker as a verifier tool. Source: "task verifiers" give the agent a trusted way to check its own output (anthropic-news-research.md #16, "Partnering with Mozilla to improve Firefox's security"). Deterministic tools narrow the search and AI removes false positives (greptile-research.md G1).

---

## 4. Output contract

### 4.1 Schema

The review emits exactly one JSON object with a single top-level key `bugs` (Source).

```json
{
  "bugs": [
    {
      "changed_file": "calculator.py",
      "changed_lines": "5",
      "bug_category": "contract-mismatch",
      "summary": "Function signature changed without updating callers.",
      "comment": "The function 'calculate_discount' now takes an additional parameter 'min_purchase', but callers (outside the diff context) still call it with only 'price' and 'discount_percent', leading to a contract mismatch.",
      "diff_fix_suggestion": "```diff\n-def calculate_discount(price, discount_percent):\n+def calculate_discount(price, discount_percent, min_purchase):\n```"
    }
  ]
}
```

### 4.2 Field rules

The JSON keys are fixed. Do not rename, add, or remove keys. Downstream tooling and the tuning loop (§5) depend on them.

| Key | Type | Rule |
|-----|------|------|
| `changed_file` | string | Repo-relative path of a file in the PR diff |
| `changed_lines` | string | Line reference in `changed_file`. A string, not a number (Source example: `"5"`). Proposed: a single line `"5"` or a range `"5-11"` |
| `bug_category` | string | One value from §4.3 |
| `summary` | string | One sentence stating the defect |
| `comment` | string | Explanation for the PR author: what breaks, where, and why. Revision 2: MUST name the failing call site, test, or reproduction (§10 D3) |
| `diff_fix_suggestion` | string | A fenced `diff` code block with the suggested fix. It is a suggestion only and is never applied automatically (§10 D14) |

Constraints:
- An empty review is `{"bugs": []}`. Do not emit prose outside the JSON.
- `{"bugs": []}` means every decision-critical branch completed and found nothing. A failed, refused, or budget-capped run is reported as "review incomplete", never as an empty review (§10 D13).
- Each entry describes a bug, not a style preference (§5.3).
- Proposed: reject or retry output that fails schema validation before posting.
- Proposed (rev 2): emit `bugs` through a strict tool or JSON schema with `additionalProperties: false` and an enum for `bug_category`. Keep post-hoc validation as a backstop.
- Severity, confidence, verifier votes, and pre-existing status live in the findings ledger, not in `bugs`. Any key change needs a spec revision (§12 Q17).

Changed in revision 2: sharpened `comment` and `diff_fix_suggestion`, and separated "empty" from "incomplete". Sources: claude-blog-research.md P10 ("Getting the most out of Opus 5.5": file, line, reason, and how to show the failure), P2 (failed reads are not quiet days), and P6 ("Building with Claude Sonnet 5.5": forced `tool_choice` returns 400, use `strict` tools; refusals return HTTP 200 with `stop_reason: "refusal"`).

### 4.3 `bug_category` enum

| Value | Status | Meaning |
|-------|--------|---------|
| `contract-mismatch` | Source | A signature or interface changed and existing callers no longer match |
| `missing-caller-update` | Proposed | Callee changed intentionally, but one or more call sites were not updated |
| `unhandled-error-path` | Proposed | A new or changed code path can raise or fail without handling |
| `auth-regression` | Proposed | Sign-in or authorisation behaviour broke (from the sign-in flow test) |
| `checkout-regression` | Proposed | Purchase flow broke (from the purchase flow test) |
| `security` | Proposed | Security defect such as exposed secrets or permission bypass (ref. G, I) |
| `data-integrity` | Proposed | Data loss or corruption risk, including migrations (ref. I) |
| `known-incident-pattern` | Proposed | The change repeats a pattern recorded in the living wiki from a revert or incident |

Adding a value requires updating this table and the tuning dataset labels (§5).

---

## 5. Prompt-tuning loop

### 5.1 Dataset

Source: historical PRs with known bugs.

Proposed record shape:
```json
{
  "pr_id": "string",
  "base_ref": "sha",
  "head_ref": "sha",
  "known_bugs": [
    {"changed_file": "string", "changed_lines": "string", "bug_category": "string"}
  ]
}
```
Proposed sources for known bugs: reverts and incidents, the same inputs as the living wiki.

Revision 2 additions (Proposed, detail in §10 D8): harvest reverts from GitHub's `revert-<pr#>-<branch>` naming (greptile-research.md G17); add known-clean PRs (claude-blog-research.md P1), refactor-heavy PRs (greptile-research.md G9), and prompt-injection cases (§10 D5). Give each hard case a stated reason and do not build the set only from cases the current reviewer misses (claude-blog-research.md P5). Audit records for mislabelled or unreproducible bugs before tuning; Anthropic flagged more than 10% of its RL environments as defective (SR) (anthropic-news-research.md #6).

Changed in revision 2: added dataset sources and hygiene rules from the posts above.

### 5.2 Loop (Source)

1. Run the review agent on every PR in the dataset.
2. Compare the agent output with the known bugs.
3. Tune the prompt.
4. Rerun. Repeat until results stop improving.

Proposed matching rule for step 2: a finding matches a known bug when `changed_file` is equal, the `changed_lines` ranges overlap, and `bug_category` is equal. Report recall, precision, and false positives per category.

Revision 2 guards (Proposed, from claude-blog-research.md P5):
- Split the dataset into train and test. Keep a change only if train and test both improve. Revert on train-up/test-flat (overfitting) or any regression.
- Make one root-cause change per round. Never paste dataset bugs into the prompt.
- Run each eval PR at least 3 times and report run-to-run stability alongside recall (greptile-research.md G2: precision measured as consistency across runs on the same PR).
- Report confidence intervals. Do not merge a gain within noise.

Changed in revision 2: step 4 "until results stop improving" now has an overfitting guard and a noise floor. The v1 loop had neither.

### 5.3 Rules (Source)

- Optimise for specific bug types.
- Skip nitpicks. Do not report style, naming, formatting, or preference issues. Linters and formatters cover those (ref. G, layer 1).
- Revision 2: prompting alone does not remove nits. Greptile found that prompting cut critical comments along with nits, and that few-shot comment examples made output worse (greptile-research.md G26, "How to Make LLMs Shut Up"). Enforce nit suppression in the verifier and comment filter (§10 D4), not only in the prompt.

Changed in revision 2: added the enforcement point for the nit rule, citing G26.

---

## 6. Cost and model routing

### 6.1 Routing (Source)

| Stage | Model tier | Effort (Proposed, rev 2) |
|-------|-----------|--------------------------|
| Select candidate files and relevant context | Cheap model | Low |
| Review the change and decide what is a bug | Frontier model | High; maximum for hard-blocked or High-band PRs |
| Verify each candidate (rev 2) | Frontier model | High |

Proposed: run the cheap model for parallel discovery tasks too (context search, flow selection). Keep the final bug decision and Submit review on the frontier model.

Changed in revision 2: added an effort axis. Effort gains are largest where verification matters, including code review and security, and effort fixes missed cases rather than wrong approaches (claude-blog-research.md P7, "Using Claude Code: Spending your effort"). Small models stay close to the diff and miss context-dependent bugs, so they never make the final call (greptile-research.md G9). Detail in §10 D7.

### 6.2 Lessons from the Cohere ProgramBench talk (Source, ref. K)

1. **The lead sets the ceiling and the workers set the price.** Output quality depends on the frontier lead. Cheap workers reduce cost. Reported result: a lead plus 16 cheaper planners cost about 70% less than an all-frontier team, with close to the same performance.
2. **Delegate discovery, not verification.** Delegate discovery and planning to cheap workers. Implementation and verification workers added little. In this system the frontier model verifies and decides on bugs.
3. **Wait for decision-critical results.** Failure modes were premature commitment and poor information flow. Explicit wait-for-results instructions helped. The lead MUST NOT submit before decision-critical branches (§2.2) return.
4. Peer messaging between workers barely helped. Proposed: do not build worker-to-worker messaging.
5. Measure reward per added dollar (§9).

### 6.3 Other cost levers (Source, ref. F)

TokenEconomizer (Microsoft Research and UC Berkeley) reports 18–31% lower agent cost without changing the model, using micro-skills, tool pruning, and failure-loop detection. It runs locally. Recommended trial (Source): run it for one week on the most-used repos. Whether it applies to the sandboxed review agent is not decided (§12).

Revision 2 additions (Proposed, detail in §10 D7): cache-stable prompt order, a per-run spend cap, and Batch API for nightly sweeps and eval runs (claude-blog-research.md P8, "What a task costs on Opus 5.5"; P15, "Prompt caching is everything"; P2).

---

## 7. Review layers and gating

### 7.1 Layers (Source, ref. G)

1. Automate the obvious: types, lint, format, tests, security scans.
2. Review locally with a fresh agent before pushing.
3. CI or platform checks on every PR. This system runs at this layer.
4. Human review of architecture, product behaviour, migrations, and infrastructure.

Rules (Source): run the code before review. Use deterministic hooks that feed failures back to the agent. Triage findings into must-fix and minor. Reviewers check correctness, security, robustness, and simplicity.

### 7.2 Risk scoring (Source, ref. I)

Run after CI is green. Score six factors:

1. Change surface
2. Blast radius
3. Reversibility
4. Data security
5. Operations impact
6. Verification gap

Bands (Source example; approvals column added in revision 2):

| Score | Band | Action | Human approvals (rev 2) |
|-------|------|--------|-------------------------|
| < 24 | Low | May auto-approve | 0 |
| 25–64 | Medium | Human approval required | 1 |
| ≥ 65 | High | Human approval required | 2 |

Notes:
- The source does not assign a score of exactly 24. Proposed: treat 24 as medium until decided (§12).
- Per-factor weights and scales are not given in the source (§12).
- Diff size alone does not set risk. Revision 2: larger PRs correlated with faster merges, possibly from lazier review (greptile-research.md G16, "Do Larger PRs Get Merged Faster?"). Do not lower scrutiny for large PRs.
- Keep an audit trail of every score and decision.
- Start conservative.
- Revision 2 approval rules (§10 D2): our reviewer's approval never counts as a human approval. Approvals are tied to a commit and are dismissed when new commits land.
- Revision 2: the band also sets review depth (§10 D7).

Changed in revision 2: Medium and High were both "human approval required" with no count. They now require 1 and 2 humans, matching Anthropic's Low 0 / Medium 1 / High 2 tiers (claude-blog-research.md P1).

### 7.3 Hard blocks (Source)

Changes that touch any of these areas MUST NOT be auto-approved, regardless of score:
- auth
- payments
- migrations
- secrets
- permissions
- public APIs
- CI workflow files, for example `.github/workflows/**` (added in revision 2)

Proposed: detect these areas with path globs plus call-graph reachability from §3.2.

Changed in revision 2: added CI workflow files. Anthropic requires a human for CI workflow files regardless of tier (claude-blog-research.md P1). A changed workflow can also alter what this system's own checks run.

### 7.4 Trunk vs leaf (Source, ref. A)

- Trunk: core infrastructure with high blast radius. Put review effort here.
- Leaf: isolated code with lower risk.
- Proposed: compute trunk-ness from call-graph in-degree (§3.2) and feed it into the blast-radius factor.

### 7.5 Feature flags (Source, ref. C)

Feature flags allow controlled rollout and easy rollback. This reduces the need for deep manual review of every change. Proposed: a change behind a default-off flag MAY lower the reversibility factor. Hard blocks still apply.

### 7.6 Adversarial review and human final pass (Source, ref. D and E)

- A second, independent agent reviews the code. The agent that wrote the code MUST NOT be the only reviewer, to avoid confirmation bias.
- Merge-ready is not launch-ready. A human does the last ~20%: high-level audit, performance, security review, and final refinement before launch.
- Revision 2: the second agent is the verifier in §10 D1. It uses a fresh context and a different prompt from the finder. Whether it also uses a different model family is open (§10 D12).

Changed in revision 2: the adversarial agent now has a concrete job (refute each candidate). Single-context agents show self-preferential bias when verifying their own work (claude-blog-research.md P13, "A harness for every task: dynamic workflows in Claude Code").

---

## 8. Evidence requirements

Source (ref. B and H): demand evidence, not only code. Evidence lets reviewers skim.

| Evidence | Produced by | Required when (Proposed) |
|----------|-------------|--------------------------|
| Unit tests | PR author or authoring agent | Any logic change |
| Runtime evidence (logs, test-run output) | Sandbox run (§2.5) | Any change that the sandbox can execute |
| Screenshots or video | Browser agent (§2.5) | UI, auth, or checkout changes |
| Reproduction for each finding (rev 2) | Analyze bugs or verifier | Every posted finding (§10 D3) |

Proposed: missing evidence raises the verification-gap factor (§7.2).

Proposed (rev 2): optionally render an HTML evidence page (diff, findings, test output, screenshots) linked from the PR comment. The PR comment itself stays rendered from `bugs` JSON (claude-blog-research.md P14, "Using Claude Code: The unreasonable effectiveness of HTML").

Changed in revision 2: evidence now applies to the reviewer's own findings, not only to the author's PR. "Bad evidence is worse than no evidence" (greptile-research.md G2).

---

## 9. Measurement

Track these per review run and per comment (Source, ref. J and K unless marked):

| Metric | Definition |
|--------|-----------|
| Addressed rate | Share of comments followed by a code change that resolves them. uReview reference: about 67%. Revision 2: "addressed" means a later commit on the PR changes lines within `changed_lines` of `changed_file` (§10 D9) |
| Comment outcome (rev 2) | Each comment labelled fixed, wrong, intentional, or doesn't matter (§10 D9) |
| Developer response and sentiment | Whether developers reply, and the tone of replies or reactions |
| False positives | Comments that developers dismiss or mark wrong. Watch for confident bad comments |
| Trajectory logs | Full agent trajectory (tool calls, context selected, model calls) for each run |
| Reward per dollar | Quality gain divided by added model cost |
| Cost per review | Proposed: tokens and dollars by model tier |
| Run-to-run stability | Proposed: share of findings that reappear on unchanged code across reruns (addresses P5) |
| Cache hit rate (rev 2) | Proposed: share of input tokens served from the prompt cache, alerted on (§10 D7) |
| Revert and incident rate (rev 2) | Proposed: guardrail metric per repo, the input to circuit breakers (§12 Q23) |

Source (ref. J): add a post-processing layer that rates, categorises, filters, and deduplicates comments before posting. Track real outcomes, not only comment counts.

Changed in revision 2: pinned "addressed" to a diff-based definition and added outcome labels, cache hit rate, and revert rate. Greptile reports address rate from subsequent-commit diffs in one post and from an unvalidated LLM judge in another, and the baselines drift between posts (greptile-research.md G26 and G5, "Greptile v4 + New Pricing"). Anthropic classifies each comment by author response and uses revert and incident rates as guardrails (claude-blog-research.md P1).

---

## 10. Research-backed design decisions

Each decision states the decision, the evidence (research file and post title), and a status. **Adopted** means a cited source supports it. **Proposed** means our inference. A sub-part marked differently from its parent is called out inline. All SR figures are orientation only.

### D1. Independent verifier pass on every candidate bug
- **Decision**: after Analyze bugs, each candidate goes to a verifier with a fresh context and a different prompt, whose job is to prove or disprove it. Only candidates the verifier fails to refute are posted. The verifier may call the §3.3 binding checker and run code in the sandbox.
- **Evidence**:
  - anthropic-news-research.md #17, "Making frontier cybersecurity capabilities available to defenders": Claude tries to prove or disprove its own finding in multiple stages to filter false positives, with separate severity and confidence ratings.
  - claude-blog-research.md P1, "How the Claude Code team does code review": parallel bug hunters, then verifier agents vote; only findings with enough agreement are posted.
  - greptile-research.md G10, "Greptile v3, an agentic approach to code review": the agent challenges its own hypotheses, which drops low-confidence comments.
  - anthropic-news-research.md #13, "Project Glasswing: An initial update": of 1,752 assessed high/critical findings, 90.6% were true positives but only 62.4% held their claimed severity (SR). Model-claimed severity is low-trust.
- **Status**: Adopted. Verifier count, vote threshold, and one-verifier-per-candidate vs voting panel are Proposed (§12 Q16). P13 says a panel of 5 reviewers is more than most coding tasks need.

### D2. Risk-tiered human approval
- **Decision**: the change, not the author, sets the tier. Low needs 0 human approvals, Medium 1, High 2. Hard blocks (§7.3) always need a human. Our reviewer never counts as an approver. Approvals are dismissed when new commits land. Start strict and loosen only with evidence.
- **Evidence**:
  - claude-blog-research.md P1, "How the Claude Code team does code review": Low 0 / Medium 1 / High 2; a second classifier can force a human; CI workflow files always need a human; a bot's approval never counts; approvals are tied to a commit.
  - anthropic-news-research.md #17: "Nothing is applied without human approval." claude-blog-research.md P9, "How we made claude.ai 3x faster in two weeks": every PR got automated review plus at least one human approval.
  - Counterpoint: greptile-research.md G1 and G20 aim for full auto-approval. We treat that as aspiration.
- **Status**: Adopted. A circuit breaker that tightens bands when revert or incident rates rise is Proposed. P1 lists it as an unbuilt gap (§12 Q23).

### D3. Every posted finding cites a concrete reproduction or test
- **Decision**: `comment` MUST name the failing call site, a failing test, or a reproduction step. For `contract-mismatch`, the non-binding call site from §3.3 is the minimal reproduction. For test-derived findings, an artifact (log, screenshot, trace, or script) is required. A candidate without one is held or dropped, never posted. Before posting a `diff_fix_suggestion`, apply it in the sandbox, re-run the binding check, and run the test suite. Drop or mark suggestions that fail.
- **Evidence**:
  - claude-blog-research.md P10, "Getting the most out of Opus 5.5": report only merge-blocking problems, each with file, line, reason, and how to show the failure; mark unconfirmed findings.
  - anthropic-news-research.md #16, "Partnering with Mozilla to improve Firefox's security": minimal test case, detailed PoC, and candidate patch build trust; patch verification re-triggers the bug and runs regression tests.
  - anthropic-news-research.md #4, "Introducing the Anthropic Cyber Mission": every OSS Scanner report carries a proof of concept.
  - greptile-research.md G2, "Building TREX": an early agent "claimed to have tried something it hadn't"; "bad evidence is worse than no evidence".
  - claude-blog-research.md P1: verification makes results "2–3x better" (one person's estimate).
- **Status**: Adopted.

### D4. Comment precision and nit suppression
- **Decision**: emit only bug-class findings (Greptile's P0/P1). Drop style, naming, and cleanup findings at the verifier, not only through the prompt. Do not let the review model self-score severity on a numeric scale. After Submit review, a per-repo comment filter drops findings similar to at least 3 previously dismissed comments. Cap findings per PR.
- **Evidence**:
  - greptile-research.md G26, "How to Make LLMs Shut Up": baseline ~19% good, ~2% wrong, ~79% nits; prompting cut critical comments too; few-shot examples hurt; an LLM severity score with a cutoff was "nearly random"; an embedding filter blocking comments similar to ≥3 downvoted ones raised address rate from 19% to 55+% in 2 weeks (SR).
  - claude-blog-research.md P1: style, diagnostic, and cleanup findings are dropped; substantive-comment share rose from 16% to 54% (SR).
  - greptile-research.md G17, "Rise of the Overnight Agents": P0 breaks production, P1 is a real bug, P2 is style or nit.
  - anthropic-news-research.md #16 and #13: maintainers asked for findings filtered to their triage criteria, and some asked Anthropic to slow disclosures.
- **Status**: Adopted for nit dropping and no self-scored severity. The embedding filter's similarity threshold, embedding model, and cold start are Proposed (§12 Q31). The per-PR cap value is Proposed.

### D5. Prompt-injection hardening for agents that read untrusted PR text
- **Decision**: treat the diff, PR description, comments, linked tickets, docs, and wiki text as data. Split the reviewer into unprivileged readers that emit schema-bound candidates and a privileged poster that holds the only write credential (comment-only). Delimit untrusted text in prompts. Flag text addressed to the reviewer or claiming prior human approval. Cheap-model workers get read-only tools. Add injection cases to the §5 dataset.
- **Evidence**:
  - claude-blog-research.md P13, "A harness for every task": quarantine pattern, where reader agents with no privileges handle untrusted content and a privileged actor acts on structured summaries.
  - anthropic-news-research.md #22, "Strengthening our safeguards through collaboration with US CAISI and UK AISI": hidden instructions and false claims that human review had already happened bypassed early classifiers.
  - anthropic-news-research.md #23, "Disrupting the first reported AI-orchestrated cyber espionage campaign": task decomposition into innocuous subtasks plus false defensive framing jailbroke an agent.
  - anthropic-news-research.md #20, "Our framework for developing safe and trustworthy agents": prompt injection is a named agent threat; Claude Code is read-only by default.
- **Status**: Proposed. The sources name the threat and the quarantine pattern, but none gives tested defences for review agents (anthropic-news-research.md §5 Q3; claude-blog-research.md §5 Q11).

### D6. Sandboxed autonomous testing with no internet by default
- **Decision**: one disposable sandbox per review, network None by default. Allowed egress is the model API and a pinned package mirror. Install from the lockfile only. Keep keys and the GitHub token in a proxy outside the sandbox and mint short-lived credentials per run. Run a pre-flight egress check before every run. Monitor tool calls and kill the run on a scope violation (host other than the dev server, writes outside the workspace). State scope as instructions ("You should only access http://localhost:<port>"). Flag new or unknown dependency names in the PR as candidate `security` findings. Run rootless with no secrets mounted.
- **Evidence**:
  - anthropic-news-research.md #6, "Improving our alignment and security practices": no internet by default, only the model API as egress, keys outside the sandbox, verify before every run; a real-time classifier blocks before the tool call; phrase boundaries as instructions; confirm tasks are solvable.
  - anthropic-news-research.md #7, "Investigating three real-world incidents in our cybersecurity evaluations": a misconfigured sandbox had live internet; a model published a malicious PyPI package under a name a setup doc referenced (SR).
  - claude-blog-research.md P3, "Claude Code in the cloud": run untrusted PRs in a disposable VM with network None; a proxy holds the GitHub token.
  - greptile-research.md G22, "Sandboxing Agents At The Kernel Level": "whatever the process can see, it can send"; rootless Podman. G2: per-review disposable instances, credential rotation, and a warning that "a cache that includes too much becomes haunted".
- **Status**: Adopted for default-deny egress, disposable instances, and out-of-sandbox credentials. The package-mirror exception is Proposed, because #6 allows only the model API and our tester needs installs (§12 Q29).

### D7. Cost routing by effort and model
- **Decision**: route by task. Cheap model at low effort for file screening, log and test-output reading, wiki crawl, and reply triage. Frontier model at high effort for the bug decision and verification. Maximum effort and the deepest fan-out for hard-blocked and High-band PRs. Fix the prompt prefix order (static prompt and tools, `review.md`, PR context, messages), keep tool order deterministic, and change models through a subagent, never mid-session. Cap spend per run at 3–5x a normal run; hitting the cap yields "review incomplete" (D13). Use the Batch API for nightly sweeps and eval runs. Re-run the effort sweep on the §5 eval whenever the model changes.
- **Evidence**:
  - claude-blog-research.md P7, "Using Claude Code: Spending your effort": effort gains are largest for code review and security (Terminal-Bench security 64%→87%, SR); effort fixes missed cases, not wrong approaches.
  - claude-blog-research.md P8, "What a task costs on Opus 5.5": Sonnet or Haiku for lookups and search subagents, Opus for review; 90% vs 96% cache hits ≈ $1.62 vs ~$0.99 in the worked example; agent teams use ~7x the tokens of a standard session.
  - claude-blog-research.md P15, "Prompt caching is everything": prefix order and cache breakers. P2: budget cap at 3–5x a normal run.
  - greptile-research.md G8: a per-task router is "the single most important design choice" (SR). G19: screening files with a light model, review at "a tenth of the cost" (SR). G6, "Introducing Greptile Plus and Apex": deeper tiers (1/3/10 credits) found more P0/P1 bugs on large PRs (SR).
- **Status**: Adopted for the routing principle and cache rules. Specific model names, effort levels per stage, and the risk-band-to-depth mapping are Proposed (§12 Q5, Q19).

### D8. Eval design: independent judge and human spot checks
- **Decision**: the primary grader is programmatic (the §5.2 file, lines, and category match plus schema validation). An LLM judge is used only for fuzzy matches. When it is used, it is a different model from the reviewer under test, scores a rubric of checkable claims, sees outputs in random order, and is run twice on the same output to confirm the verdict holds. A human spot-checks a sample of judge decisions each tuning round. Judge the trajectory as well as the output (was context pulled before the finding). Run the eval on every prompt, tool, or model change and daily.
- **Evidence**:
  - claude-blog-research.md P5, "Automating eval design and hillclimbing with Claude": cheapest grader first; LLM judge with a rubric, random order, blind to baseline, and a different model from the one under test; run the grader twice; warn when the baseline is ≥~95%.
  - anthropic-news-research.md #18, "Protecting the wellbeing of our users": auditor model plus judge model with human spot checks. #21, "Building safeguards for Claude": model grading with human review as a check.
  - claude-blog-research.md P1: incident and revert PRs confirmed by experts form an eval that runs daily.
  - greptile-research.md G9: the trace is judged as closely as the output. G5: Greptile's "addressed" judge is not validated, which is the failure we avoid.
- **Status**: Adopted. Spot-check sample size is Proposed.

### D9. Feedback loop: measure whether comments get addressed
- **Decision**: record every posted finding in the findings ledger with a stable ID (file, symbol, category), the head SHA, and the GitHub comment's integer database ID alongside its GraphQL node ID. Mark it addressed when a later commit changes lines within its `changed_lines`. Label each comment fixed, wrong, intentional, or doesn't matter from the author's response. Derive precision from these labels. Feed "wrong" and "doesn't matter" into the D4 filter. Turn recurring comments into lint rules or `review.md` entries.
- **Evidence**:
  - greptile-research.md G26: address rate from subsequent-commit diffs, chosen over reactions.
  - claude-blog-research.md P1: four-label author-response classification; recurring comments fixed at the source.
  - Spec ref. J (Uber uReview): about 67% addressed; track real outcomes.
  - greptile-research.md G21, "Every GitHub Object Has Two IDs": store both IDs. G5 vs G7: addressed baselines of 43% and 52% for the same version, so do not chain or compare vendor numbers.
- **Status**: Adopted.

### D10. Skills and reference packaging
- **Decision**: keep project context short and move procedures into skills. `review.md` is owned by code owners, with sections for what failure looks like, skip paths, a nit cap, evidence requirements, and per-area gotchas; rule blocks carry path globs and only matching rules load. Each browser flow (sign-in, checkout) is a verification skill that records video and asserts state at each step. Skills use `SKILL.md` as a hub pointing to spoke files. Wiki incident patterns become per-area gotchas that feed `known-incident-pattern`. Audit the reviewer prompt for conflicting instructions each tuning round, and express categories through the enum, not long examples. Pin and review third-party skills like dependencies.
- **Evidence**:
  - claude-blog-research.md P12, "Lessons from Building Claude Code: How We Use Skills": product-verification skills (`signup-flow-driver`, `checkout-verifier`) had "the most measurable impact" (no figure); the Gotchas section is the highest-signal content; hub and spoke files.
  - claude-blog-research.md P11, "The new rules of context engineering for Claude 5 generation models": more than 80% of the system prompt removed with no measurable eval loss (SR); audit for conflicts; move verification steps into a skill.
  - claude-blog-research.md P1: `REVIEW.md` from code owners with failure modes, nit cap, skip paths, and file-and-line evidence.
  - greptile-research.md G11, "Greptile's Biggest Update Yet": rules scoped by repo, directory, or glob.
  - anthropic-news-research.md #1, "Donating Agent Skills to the Agentic AI Foundation": `SKILL.md` is a cross-vendor standard (70+ platforms, SR); security guidance for skill installers is not yet written.
- **Status**: Adopted for packaging and `review.md` structure. Third-party skill pinning is Proposed until the AAIF guidance ships (§12 Q27).

### D11. Convergence controls and findings ledger
- **Decision**: suppress re-posts of ledger findings. After round 2, re-review only the incremental diff. After round 3, post one "reconsider the approach" note instead of more findings. Flag the bug class so the author fixes every instance. Use `git blame` to divert candidates on lines older than `base_ref` to a nightly sweep, not to PR comments.
- **Evidence**: claude-blog-research.md P1: 3-round cap, diff-only re-review, no repeated findings, flag the bug class, nightly runs over existing code; ~59% of repeat flags were on pre-existing code (SR). P2: a ledger with one record per item so open items are not re-reported. greptile-research.md G20, "There is an AI Code Review Bubble": fix-and-re-review loops until no new comments, with ambiguity escalated to a human.
- **Status**: Adopted for the controls. The `git blame` filter and the round definition are Proposed (§12 Q18, Q22).

### D12. Reviewer independence from the author model
- **Decision**: detect the author model from `Co-Authored-By` trailers and branch prefixes (`codex/`, `cursor/`, `claude/`). A/B test two setups on the §5 eval: (a) same family with fresh context and a different prompt, (b) a different model family. Adopt whichever wins on recall and stability at equal cost.
- **Evidence**: for inversion, greptile-research.md G19 ("model inversion" raised bug catch rate by about 10%, SR) and G23, "Software Needs An Independent Auditor" (the value is uncorrelated failures). Against, greptile-research.md G24, "AI Code Review: Should the Author Be The Reviewer?" (stateless calls and different scaffolding give independence anyway) and claude-blog-research.md P1 (Claude reviews Claude-written code with fresh-context subagents; ~90% of Claude Code is Claude-written, SR).
- **Status**: Proposed. The sources disagree (§12 Q30).

### D13. Failure is never an empty review
- **Decision**: a branch failure, timeout, budget stop, or `stop_reason: "refusal"` produces a "review incomplete" status. Record a finding as posted only after the GitHub API confirms it. Re-check that the head SHA is unchanged before posting.
- **Evidence**:
  - claude-blog-research.md P2: a failed read is "unreadable", never a quiet day; confirm before recording; re-verify before posting; a run that hits its cap stops with `budget_reached` instead of failing. P6: refusals return HTTP 200 with `stop_reason: "refusal"`.
  - anthropic-news-research.md #8, "Fable 5's cyber safeguards and our jailbreak framework": high-uplift vulnerability finding is blocked, while secure coding is benign, so `security` prompts may be refused.
- **Status**: Adopted for the principle (from a different system in P2). Its application to PR review is our mapping.

### D14. Suggestions only; no automatic fixes
- **Decision**: the system never commits, applies, or opens a PR with `diff_fix_suggestion`. It posts suggestions only.
- **Evidence**: anthropic-news-research.md #17: "Nothing is applied without human approval"; #16: passing tests do not make an AI patch mergeable. greptile-research.md G23: a reviewer that also generates code shares failure modes and has an incentive to go easy on its own output.
- **Status**: Adopted.

---

## 11. Related prior art

| Source | Key claim | What we take from it |
|--------|-----------|----------------------|
| claude-blog-research.md P1, "How the Claude Code team does code review" | Verifier vote, risk-tiered approvals (0/1/2), daily incident eval, convergence caps. Substantive comments 16%→54% (SR) | D1, D2, D8, D9, D11; §7.2 approval counts; CI workflows as a hard block |
| claude-blog-research.md P5, "Automating eval design and hillclimbing with Claude" | Train/test split, keep/revert rule, judge differs from the model under test, grader run twice | §5.2 guards; D8 |
| claude-blog-research.md P7, "Using Claude Code: Spending your effort" | Effort gains largest for code review and security; fixes missed cases, not wrong approaches | §6.1 effort axis; D7 |
| claude-blog-research.md P8 and P15, cost and prompt caching posts | Turns × context drives cost; cache order and breakers; Batch is half price | D7 cache and batch rules; §9 cache hit rate |
| claude-blog-research.md P13, "A harness for every task" | Fan-out with a barrier, adversarial verification, quarantine of untrusted input | D1, D5; §2.2 Verify step |
| claude-blog-research.md P12, "How We Use Skills" | Verification skills had the most measurable impact; Gotchas section; hub and spoke | D10 |
| anthropic-news-research.md #17, Claude Code Security | Prove-or-disprove verification, separate severity and confidence, human approval before applying | D1, D14 |
| anthropic-news-research.md #13, Glasswing initial update | 90.6% true positives, 62.4% severity confirmed on assessed findings (SR) | Treat model severity as low-trust (D1) |
| anthropic-news-research.md #16, Mozilla Firefox security | Minimal test case, PoC, candidate patch; two-axis patch verification; task verifiers | D3; §3.3 binding checker as a tool |
| anthropic-news-research.md #6 and #7, security practices and eval incidents | No internet by default, keys outside, pre-run egress check; malicious package from a setup doc | D6 |
| anthropic-news-research.md #22, CAISI and UK AISI collaboration | Hidden instructions and fake "human review happened" claims bypassed classifiers | D5 |
| greptile-research.md G19, "How to Build a Code Review Agent" | Same five problems as §1; defines model inversion (~10% more bugs, SR); cheap-model file screening | §2.1 definition; D7, D12 |
| greptile-research.md G26, "How to Make LLMs Shut Up" | ~79% of raw comments were nits; embedding filter took address rate 19%→55+% (SR); diff-based address rate | D4, D9; §5.3 |
| greptile-research.md G2, "Building TREX" | One execution subagent per issue; artifacts required; precision as cross-run consistency | §2.5; D3; §5.2 stability runs |
| greptile-research.md G17, "Rise of the Overnight Agents" | Revert linkage via branch naming; P0/P1/P2 taxonomy; AI-author detection from trailers and prefixes | §5.1 dataset; D4; D12 |

---

## 12. Open questions (not decided)

Changed in revision 2: moved from §10. Q1–Q14 keep their numbers. Q15–Q36 merge the open questions from the three digests, with duplicates folded into one entry. The source digest sections are cited as CB §5 (claude-blog-research.md), AN §5 (anthropic-news-research.md), and GR §6 (greptile-research.md).

### 12.1 From revision 1

1. Language support beyond Python `ast` for §3. No research post addresses it (CB §5 Q10, GR §6 Q7).
2. Format of `changed_lines` for ranges and multi-range findings.
3. Final `bug_category` enum. All values except `contract-mismatch` are Proposed. Greptile's per-agent failure profiles (G17) suggest `n-plus-one` and `config-error`, to add only if the eval shows them.
4. Weights and scales for the six risk factors, and the band for a score of exactly 24.
5. Which cheap model and which frontier model to use. "Jev" appears in the sketch only as an example and is not a model in any research post. Candidates named in CB: Haiku 5.5 (availability unconfirmed), Sonnet 5.5, Opus 5.5, and Fable 5.1 (escalation only).
6. Call-graph depth limit and handling of dynamic calls that cannot be resolved.
7. Living wiki crawl cadence, storage, and retrieval method. §2.4 now proposes refresh on merge plus a scheduled crawl.
8. How the browser agent picks flows beyond sign-in and checkout. Proposed: a flow map in the wiki, drafted by a model and confirmed by owners (GR G19).
9. Whether DAST (OWASP ZAP) runs in the sandbox.
10. Whether to adopt TokenEconomizer after the one-week trial.
11. Size and labelling process for the historical PR dataset.
12. How the adversarial second agent differs from the primary reviewer. Partly answered: fresh context and a different prompt (§10 D1). Model family is Q30.
13. Resolved in revision 2: "model inversion" means reviewing with a different model family from the author (§2.1, GR G19).
14. Whether this system adopts uReview-style specialised reviewers (single-file, multi-file, monorepo, AI linter, team agents).

### 12.2 From the research digests

15. **No baseline of our own.** Every headline figure is self-reported: Anthropic's P1 metrics and the "2–3x" estimate, Glasswing's 90.6%, and Greptile's +20%, +10%, and 19%→55%. No post publishes precision for general (non-security) code review. Measure our own baseline on §5 before setting targets (CB Q1, AN Q1, GR Q1).
16. **Verifier mechanics.** Number of verifiers, vote threshold, and cost per finding. How "prove or disprove" works is not described: execution, a separate model, or the same context (CB Q3, AN Q2).
17. **Severity outside the schema.** Severity, confidence, and pre-existing tags help triage, but §4 forbids new keys. Decide whether they stay in the ledger or the schema gets a version bump, and whether risk bands (§7.2) consume per-finding severity given low severity calibration (CB Q2, AN Q9).
18. **What counts as a round** for the 3-round cap: each push or each review request (CB Q4).
19. **Effort mapping.** P7's gains come from Terminal-Bench, not PR review. We have no data on how effort changes recall or precision on our dataset (CB Q5).
20. **Model availability and refusals.** A model can be withdrawn the same day (AN #11). It is unclear whether `security` prompts or exploit-like PoCs in `comment` trip cyber classifiers; the CJS framework is a draft with no measured false-positive rates. Needs a fallback model and a refusal smoke test (AN Q5, Q6).
21. **Indexer vs agentic search.** P16 moved away from RAG; G15 suggests NL-summary, function-level embeddings if embeddings are kept (single example). No post measures review recall for either. Run an A/B (CB Q7).
22. **Pre-existing bugs.** Owner and destination for nightly-sweep findings (issues or wiki) (CB Q8).
23. **Circuit breakers.** Threshold and window for tightening approval when revert or incident rates rise. An open gap at Anthropic too (CB Q9).
24. **Prompt-injection defences are untested.** D5 is design inference. No post measures injection through PR content into a reviewer (CB Q11, AN Q3).
25. **Cache economics for short reviews.** P8 and P15 assume long sessions; for small PRs, cache-write overhead may dominate (CB Q12).
26. **Hosting.** Managed Agents or cloud routines vs our own sandbox for the webhook pipeline (CB Q14).
27. **Skill vetting.** AAIF installer security guidance is not yet published; how to vet third-party skills before loading them (AN Q4).
28. **Data retention.** Fable-class models require 30-day retention. Check whether sending private repo code through the reviewer is acceptable, or whether EFS or ZDR is needed (AN Q7).
29. **Sandbox egress for dependency install.** #6 assumes egress only to the model API, but the tester needs package installs. G22 covers filesystem isolation only, not network, seccomp, or resource limits. The mirror and lockfile approach needs validation (AN Q8, GR Q6).
30. **Model inversion vs same-family fresh context.** The ~10% gain has no dataset size and the cited N+1 research has no source. G24 and P1 argue independence comes from context, not family (GR Q3).
31. **Comment-filter details.** Similarity threshold, embedding model, cold start for new repos, and behaviour when a comment is similar to both upvoted and downvoted history (GR Q4).
32. **Merge and dedup rule** for per-hypothesis findings in Submit review. Greptile does not describe its rule (GR Q5).
33. **Addressed-rate validity.** §9 pins a diff-based definition; it may miss fixes made outside `changed_lines` or count unrelated edits. Validate against human labels (GR Q2).
34. **Ticket-intent checks.** Comparing a PR with ticket acceptance criteria has no precision data and risks feature-completeness comments that are not bugs (GR Q8).
35. **Date anomalies** in several fetched pages (AN #15, #17, #19, #21; GR G4, G7). Treat page dates as approximate (AN Q10, GR Q10).
36. **Fetch fidelity.** All three digests were built from model-generated page summaries, not raw HTML. Re-check any figure against the live page before it gates a decision or is cited externally (CB Q13, AN Q10, GR Q11).

---

## 13. References

| Ref | Source | Description |
|-----|--------|-------------|
| A | "Trunk vs leaf" video | Codebase as a tree. Focus review on high-blast-radius trunk code |
| B | Agentic validation video | Engineers attach AI-produced proof (tests, runtime evidence, visuals) to PRs |
| C | Feature gating video | Feature flags for controlled rollout and rollback |
| D | Adversarial review video | Independent second agent reviews to avoid confirmation bias |
| E | Merge-ready vs launch-ready video | Human final ~20% before launch |
| F | https://github.com/microsoft/TokenEconomizer | Local plugin that cuts agent cost 18–31% with micro-skills, tool pruning, and failure-loop detection |
| G | "LGTM, Ship It" (Owain Lewis) | Four-layer review model, risk-scaled review intensity, project `review.md` |
| H | "LGTM / AI code review problem" episode | Verify behaviour, demand evidence, keep PRs small |
| I | "Merge Mommy" | Risk-based auto-approval with six factors, score bands, and hard blocks |
| J | Uber uReview | Multi-agent review with post-processing and outcome tracking |
| K | Cohere ProgramBench talk | Strong lead plus cheap planners. Lead sets the ceiling, workers set the price |
| CB | `docs/claude-blog-research.md` | Digest of 16 Claude engineering blog posts (P1–P16), fetched 2026-11-09 |
| AN | `docs/anthropic-news-research.md` | Digest of 26 Anthropic news posts (#1–#26), fetched 2026-11-09 |
| GR | `docs/greptile-research.md` | Digest of 27 Greptile blog posts (G1–G27), fetched 2026-11-09 |
| — | https://en.wikipedia.org/wiki/Dynamic_application_security_testing | Overview of DAST, testing a running app from outside |
| — | https://www.zaproxy.org/ | OWASP ZAP, an open-source DAST tool |
