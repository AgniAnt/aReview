# AI-Assisted Code Review: Design for Implementing Agents

Status: draft. Audience: coding agents that implement or extend this system.

Conventions:
- **Source**: stated in the source notes (architecture sketch, user spec, or reference material).
- **Proposed**: not in the source notes. Treat as a default to confirm, not a decided requirement.
- MUST / SHOULD / MAY keep their RFC 2119 meanings.

---

## 1. Goal and problems solved

Goal (Source, sketch title block): "Codebase Context. It runs your code. The right model for each task."

The system reviews a pull request (PR) automatically and posts actionable bug findings back to it. It targets five problems (Source):

| # | Problem | Main countermeasure in this design |
|---|---------|------------------------------------|
| P1 | Missed bugs | Change-impact analysis (§3), autonomous testing (§2.5), prompt-tuning loop against known bugs (§5) |
| P2 | Missing context | MCP docs and tickets, codebase indexer, living wiki (§2.3, §2.4) |
| P3 | Expensive reviews | Cheap model for context selection, frontier model only for review (§6) |
| P4 | Nitpicky comments | Bug-only output contract (§4), skip-nitpicks rule (§5) |
| P5 | New bugs on every run | Fixed JSON schema and categories (§4), regression dataset (§5). Proposed: stabilise output across runs by deduplicating against prior findings (§9) |

Non-goals (Source, ref. G and H): replacing human review of architecture, product behaviour, migrations, and infrastructure. Merge-ready is not launch-ready (§7.6).

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

The sketch labels the review agent with its two context inputs (MCP and indexer) as "model inversion". The notes do not define the term further.

### 2.2 Data flow

1. GitHub sends a PR webhook to the sandbox.
2. The review agent receives the PR diff and runs **Plan Review**.
3. Plan Review fans out to three parallel branches:
   1. **Find Context**: the cheap model receives candidate files and selects relevant context for the frontier model. Context sources are MCP (docs, tickets), the indexer (codebase), and the living wiki.
   2. **Test user flows**: the autonomous tester (§2.5).
   3. **Analyze bugs**: change-impact analysis (§3) plus frontier-model review.
4. All three branches feed **Submit review**.
5. Submit review emits the `bugs` JSON (§4). The agent renders it as a PR comment and posts it.

Proposed: Submit review MUST wait for every branch whose result is decision-critical before it emits output (§6.2, "wait for decision-critical results").

### 2.3 Context sources

| Source | Transport | Content |
|--------|-----------|---------|
| Docs | MCP | Design docs, coding standards |
| Tickets | MCP | Issue or ticket linked to the PR |
| Codebase | Indexer | Symbols, files, call relationships |
| Living wiki | Periodic crawl | Codebase knowledge plus history of reverts and incidents |

Proposed: a project `review.md` (ref. G) holds review rules and coding standards, and is always loaded into context.

### 2.4 Living wiki

- Inputs (Source): periodic crawls of the codebase, reverts, incidents.
- Output (Source): wiki content that feeds a review agent running in its own sandbox.
- Proposed: index wiki entries by file path and symbol so Find Context can retrieve entries for the changed code and its callers. Crawl cadence is not decided (§10).

### 2.5 Autonomous testing

Source:
1. Check out the PR branch and install dependencies in the sandbox.
2. Start the app (dev server).
3. A browser agent drives the app. Flows depend on what changed:
   - auth code changed → run the sign-in flow;
   - checkout code changed → run the purchase flow.
4. Output: a bug report that feeds Submit review.

Proposed: map each flow to path globs in config. Capture screenshots or video as evidence (§8). Map findings to `bug_category` values `auth-regression` and `checkout-regression` (§4.3).

Proposed: DAST tooling such as OWASP ZAP (see §11) can run against the same running app. The notes list it as a reference only.

---

## 3. Change-impact analysis pipeline

Three sequential steps (Source). Python's `ast` module is the specified parser (Source). Support for other languages is not decided (§10).

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
- Constraint: report `in_diff: false` call sites explicitly. These are "callers outside the diff context", the main source of contract-mismatch bugs (Source example).

### 3.3 Step 3: Detect contract changes

- **Purpose**: check whether a function's signature changed. If it did, existing call sites may now be broken.
- **Input**: base and head `ast.arguments` for each `modified` symbol, plus `call_sites` from Step 2.
- **Process**:
  1. Compare positional parameters, defaults, keyword-only parameters, `*args`, and `**kwargs`.
  2. A contract change exists if a parameter is added without a default, removed, renamed, or reordered.
  3. For each call site of a changed contract, check whether its arguments still bind to the new signature.
  4. Each call site that fails to bind is a candidate bug.
- **Output**: candidate bugs passed to the frontier model for confirmation. Each candidate becomes at most one entry in the `bugs` output (§4), with `bug_category` `contract-mismatch` or `missing-caller-update`.

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
| `comment` | string | Explanation for the PR author: what breaks, where, and why |
| `diff_fix_suggestion` | string | A fenced `diff` code block with the suggested fix |

Constraints:
- An empty review is `{"bugs": []}`. Do not emit prose outside the JSON.
- Each entry describes a bug, not a style preference (§5.3).
- Proposed: reject or retry output that fails schema validation before posting.

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

### 5.2 Loop (Source)

1. Run the review agent on every PR in the dataset.
2. Compare the agent output with the known bugs.
3. Tune the prompt.
4. Rerun. Repeat until results stop improving.

Proposed matching rule for step 2: a finding matches a known bug when `changed_file` is equal, the `changed_lines` ranges overlap, and `bug_category` is equal. Report recall, precision, and false positives per category.

### 5.3 Rules (Source)

- Optimise for specific bug types.
- Skip nitpicks. Do not report style, naming, formatting, or preference issues. Linters and formatters cover those (ref. G, layer 1).

---

## 6. Cost and model routing

### 6.1 Routing (Source)

| Stage | Model tier |
|-------|-----------|
| Select candidate files and relevant context | Cheap model |
| Review the change and decide what is a bug | Frontier model |

Proposed: run the cheap model for parallel discovery tasks too (context search, flow selection). Keep the final bug decision and Submit review on the frontier model.

### 6.2 Lessons from the Cohere ProgramBench talk (Source, ref. K)

1. **The lead sets the ceiling and the workers set the price.** Output quality depends on the frontier lead. Cheap workers reduce cost. Reported result: a lead plus 16 cheaper planners cost about 70% less than an all-frontier team, with close to the same performance.
2. **Delegate discovery, not verification.** Delegate discovery and planning to cheap workers. Implementation and verification workers added little. In this system the frontier model verifies and decides on bugs.
3. **Wait for decision-critical results.** Failure modes were premature commitment and poor information flow. Explicit wait-for-results instructions helped. The lead MUST NOT submit before decision-critical branches (§2.2) return.
4. Peer messaging between workers barely helped. Proposed: do not build worker-to-worker messaging.
5. Measure reward per added dollar (§9).

### 6.3 Other cost levers (Source, ref. F)

TokenEconomizer (Microsoft Research and UC Berkeley) reports 18–31% lower agent cost without changing the model, using micro-skills, tool pruning, and failure-loop detection. It runs locally. Recommended trial (Source): run it for one week on the most-used repos. Whether it applies to the sandboxed review agent is not decided (§10).

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

Bands (Source example):

| Score | Band | Action |
|-------|------|--------|
| < 24 | Low | May auto-approve |
| 25–64 | Medium | Human approval required |
| ≥ 65 | High | Human approval required |

Notes:
- The source does not assign a score of exactly 24. Proposed: treat 24 as medium until decided (§10).
- Per-factor weights and scales are not given in the source (§10).
- Diff size alone does not set risk.
- Keep an audit trail of every score and decision.
- Start conservative.

### 7.3 Hard blocks (Source)

Changes that touch any of these areas MUST NOT be auto-approved, regardless of score:
- auth
- payments
- migrations
- secrets
- permissions
- public APIs

Proposed: detect these areas with path globs plus call-graph reachability from §3.2.

### 7.4 Trunk vs leaf (Source, ref. A)

- Trunk: core infrastructure with high blast radius. Put review effort here.
- Leaf: isolated code with lower risk.
- Proposed: compute trunk-ness from call-graph in-degree (§3.2) and feed it into the blast-radius factor.

### 7.5 Feature flags (Source, ref. C)

Feature flags allow controlled rollout and easy rollback. This reduces the need for deep manual review of every change. Proposed: a change behind a default-off flag MAY lower the reversibility factor. Hard blocks still apply.

### 7.6 Adversarial review and human final pass (Source, ref. D and E)

- A second, independent agent reviews the code. The agent that wrote the code MUST NOT be the only reviewer, to avoid confirmation bias.
- Merge-ready is not launch-ready. A human does the last ~20%: high-level audit, performance, security review, and final refinement before launch.

---

## 8. Evidence requirements

Source (ref. B and H): demand evidence, not only code. Evidence lets reviewers skim.

| Evidence | Produced by | Required when (Proposed) |
|----------|-------------|--------------------------|
| Unit tests | PR author or authoring agent | Any logic change |
| Runtime evidence (logs, test-run output) | Sandbox run (§2.5) | Any change that the sandbox can execute |
| Screenshots or video | Browser agent (§2.5) | UI, auth, or checkout changes |

Proposed: missing evidence raises the verification-gap factor (§7.2).

---

## 9. Measurement

Track these per review run and per comment (Source, ref. J and K unless marked):

| Metric | Definition |
|--------|-----------|
| Addressed rate | Share of comments followed by a code change that resolves them. uReview reference: about 67% |
| Developer response and sentiment | Whether developers reply, and the tone of replies or reactions |
| False positives | Comments that developers dismiss or mark wrong. Watch for confident bad comments |
| Trajectory logs | Full agent trajectory (tool calls, context selected, model calls) for each run |
| Reward per dollar | Quality gain divided by added model cost |
| Cost per review | Proposed: tokens and dollars by model tier |
| Run-to-run stability | Proposed: share of findings that reappear on unchanged code across reruns (addresses P5) |

Source (ref. J): add a post-processing layer that rates, categorises, filters, and deduplicates comments before posting. Track real outcomes, not only comment counts.

---

## 10. Open questions (not decided)

1. Language support beyond Python `ast` for §3.
2. Format of `changed_lines` for ranges and multi-range findings.
3. Final `bug_category` enum. All values except `contract-mismatch` are Proposed.
4. Weights and scales for the six risk factors, and the band for a score of exactly 24.
5. Which cheap model and which frontier model to use. "Jev" appears in the sketch only as an example.
6. Call-graph depth limit and handling of dynamic calls that cannot be resolved.
7. Living wiki crawl cadence, storage, and retrieval method.
8. How the browser agent picks flows beyond sign-in and checkout.
9. Whether DAST (OWASP ZAP) runs in the sandbox.
10. Whether to adopt TokenEconomizer after the one-week trial.
11. Size and labelling process for the historical PR dataset.
12. How the adversarial second agent differs from the primary reviewer (model, prompt, or context).
13. Meaning of "model inversion" in the sketch.
14. Whether this system adopts uReview-style specialised reviewers (single-file, multi-file, monorepo, AI linter, team agents).

---

## 11. References

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
| — | https://en.wikipedia.org/wiki/Dynamic_application_security_testing | Overview of DAST, testing a running app from outside |
| — | https://www.zaproxy.org/ | OWASP ZAP, an open-source DAST tool |
