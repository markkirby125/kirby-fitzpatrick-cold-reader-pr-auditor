# Cold Reader Pr Auditor — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [The EXACT System to Transform Messy Drafts into Clear Writing](https://www.youtube.com/watch?v=6HPNb0tiDNg)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: Zero-Context Reconstruction Load

No author writes from a blank mind. An author writes from *inside* a live model — the debugging session, the Slack thread, the failing CI run, the alternative already rejected, the two hours of profiler output that made the fix obvious. The artifact inherits almost none of it. What survives is a compressed residue: *"speeds up reconciliation"*, *"fixes the flaky retry"*, *"small cleanup"*.

**The Cold Reader Protocol** inverts that asymmetry. It audits every PR, RFC, ADR, runbook, and review comment from the stance of a reader who has **zero prior context** — never in the room, never in the thread, never inside the author's head — and treats any sentence whose meaning depends on that missing context as a **defect**, not a stylistic preference.

Three load-bearing definitions:

1. **Reconstruction Load (RL)** — the number of inferences a competent reader must make, from external knowledge alone, to recover what the author meant. Target: **RL = 0** on the decision-critical path. Every unit of RL is an unanswered question, and every unanswered question becomes a ping to the author.
2. **Tribal Knowledge** — an assertion that is only true because of shared, unwritten, local history: *"the usual config"*, *"the legacy path"*, *"as discussed"*, *"obviously"*, *"the known issue"*. Tribal knowledge is not context; it is **debt** — invisible in the artifact, payable by the next reader. When the reader who pays is not the author who borrowed, the review stalls and the decision leaks into a DM.
3. **The Binding Rule** — every claim in a summary, comment, or RFC must resolve to an artifact the reader can independently open, run, or diff: a path with line numbers, a commit SHA, a command plus its output, a metric, a benchmark, an incident ID, a log line. A claim with no artifact is not a claim; it is an opinion wearing a claim's clothes.

Why this binds harder in software than in prose: engineering artifacts are read by **future strangers under time pressure** — the 03:00 on-call engineer, the new hire in month one, the auditor, and the author's own self six months later. An ungrounded sentence is a latent defect that detonates at precisely the moment nobody can ask the author.

The cold reader's acceptance questions — all five must be answerable from the document alone:

| # | Question the cold reader asks | Boundary it enforces |
|---|---|---|
| Q1 | **What changed, in one sentence?** | A change statement, not a file list |
| Q2 | **Why now?** | Motivation grounded in an observed system state, not a preference |
| Q3 | **How do I know it works?** | Claim → mechanism → measured result → reproducible command |
| Q4 | **What can break, and how do I undo it?** | Blast radius + single-operation rollback |
| Q5 | **What am I being asked to do?** | The decision or action, stated explicitly |

An unanswered question does not fail politely. It silently converts into a withdrawal from the author's calendar.

```text
[ANTI-PATTERN: Insider Summary]  →  the reader must rebuild the author's context
+--- PR #812 — "perf + retry fixes" ----------------------------------------+
| "Speeds up the reconciliation path and fixes the flaky retry issue."       |
|    ^ claim        ^ which path?            ^ which issue? observed where?  |
+---------------------------------------------------------------------------+
Cold reader reconstruction load: 5 blocking questions
  Q1 what changed?  Q2 why now?  Q3 measured how?  Q4 breaks what?  Q5 undo?
Path: reviewer pings author -> author answers in a DM -> the answer never
      reaches the artifact -> the next reader pays it again (debt compounds)
Result: approval happens for social reasons, or the PR sits for two days.

[COLD READER PROTOCOL]  →  0 reconstruction load; evidence audited, not intent
+--- PR #812 ---------------------------------------------------------------+
| CLAIM      p95 reconciliation latency 480 ms -> 210 ms                    |
| ARTIFACT   bench/reconcile_bench.py ; make bench-reconcile RUNS=5         |
| BASELINE   main @ 4f9c1ab, same runner, median of 5 runs, spread +/-6 ms  |
| MECHANISM  N+1 invoice fetch -> one batched query (handler.go:88),        |
|            isolated by the query-count assertion in reconcile_test.go:214 |
| SCOPE      read path only; no schema change; revert = 1 commit            |
+---------------------------------------------------------------------------+
Cold reader reconstruction load: 0 blocking questions
Result: the reviewer spends their attention auditing the evidence.
```

**The audit is a separate pass from the composition.** The lecture's revision discipline applies directly: drafting is generative and permissive; cold reading is adversarial, artifact-first, and happens after a pause — preferably by reading the summary *before* the diff, so the text must carry intent on its own.

---

## 2. Core Transformation Protocols

1. **Assume zero context by default.** Write and audit for a reader who cloned the repository five minutes ago, has read no prior PR, and is mildly skeptical but fair. Never audit from the author's memory.
2. **Reject ungrounded tribal knowledge outright.** Any sentence that requires having been in a meeting, a channel, or a prior head to be true is rewritten to carry its own premise, or deleted. There is no third option.
3. **Bind every claim to an artifact.** For each assertion, ask: *what would the reader open or run to check this?* If the answer is "nothing", the assertion is a judgment — label it as one.
4. **Make the causal link explicit: claim → mechanism → measurement.** A benchmark number alone is a reading, not evidence; the reader needs the mechanism that explains *why* the number moved, because the mechanism is what generalizes to their workload. *"p95 fell 56%"* invites *"will that hold on my data?"* — *"p95 fell 56% because the per-invoice N+1 fetch is now one batched query; query count fell 1,240 → 41"* answers it in the same breath.
5. **Bind every benchmark to a named baseline.** Workload, runner, baseline ref (SHA/branch/tag), sample size, dispersion, and the raw command. A number without a baseline is an anecdote with units.
6. **Tag every sentence: `FACT` / `JUDGMENT` / `UNKNOWN`.** A `FACT` must resolve to an artifact now. A `JUDGMENT` must be labeled and its criterion named. An `UNKNOWN` must name an owner and a date. Never launder an assumption as a fact — that is the primary mechanism by which tribal knowledge survives review.
7. **Quantify or delete.** `significantly`, `much faster`, `cleaner`, `safer`, `a bit slower` carry zero information and are the most common unmeasured claims in PR bodies.
8. **State scope, blast radius, and rollback.** The reader's first instinct after "why" is "what happens if this is wrong". One line each: what is excluded, what breaks, and the single operation that undoes it.
9. **Patch answers back into the artifact.** When a reviewer's question is answered in a DM or a thread, the answer is not recorded — it is spent. Every clarifRication produced during review must be written back into the PR body, comment thread on the artifact, or doc.
10. **Run the "I was there" test per sentence.** *Does this sentence require having been present?* Any `yes` is a rewrite, regardless of how clear it feels to the author.
11. **Front-load nothing at the reader's expense — but answer Q1 in the first sentence.** The change statement precedes the narrative; headings must carry the thesis so a 15-second skim reaches the same conclusion as a full read.
12. **Audit in a different order than you wrote.** Read the title and summary only, then the evidence, then the diff. If the summary cannot be audited without the diff, the summary is decorative.

### 2.1 Claim class → required evidence

| Claim class | Typical phrasing | Required evidence | Fails the audit |
|---|---|---|---|
| Performance | "N× faster", "reduces latency" | Named benchmark + workload + runner + baseline ref + runs/dispersion + raw command | A screenshot, a local run, "feels snappier" |
| Correctness | "fixes", "now handles" | Failing input, before/after output, test name and line | "tested locally" |
| Reliability | "no longer flaky" | Failure rate before/after + CI run IDs | "green now" |
| Causality | "because of the N+1 query" | Mechanism + the assertion or measurement that isolates it | Correlation presented as cause |
| Scope/safety | "no breaking changes" | Contract list (API, schema, config, CLI, wire format) + the check performed | "should be fine" |
| Cost/trade-off | "cheaper", "3 engineer-weeks" | Source of the estimate: spike PR, measured run, quote | An unsourced number |
| Risk/unknown | "might affect X" | Owner + date + the test that would close it | An indefinite hedge |

### 2.2 Forbidden tribal-knowledge tokens

| Banned token | Why it fails | Required replacement |
|---|---|---|
| *"as discussed"*, *"per the meeting"*, *"per Slack"* | The reader has no access; the decision has no durable home | Restate the decision and its rationale in the artifact |
| *"the usual config"*, *"the standard flow"* | Undefined referent; two readers imagine two systems | Path + line, or the literal config block |
| *"the known issue"*, *"the flaky retry"* | Unbounded; nobody can look it up | Incident ID, CI build ID, or issue link |
| *"legacy"*, *"we've always done it this way"* | Appeals to history instead of an artifact | Name the module, the callers, and what replaces it |
| *"obviously"*, *"clearly"*, *"of course"* | Zero-information intensifier that hides a premise | State the premise |
| *"should be fine"*, *"looks good"* | Unfalsifiable verdict | The check performed and its result |
| *"etc."*, *"and so on"* | An unbounded list the reader cannot complete | Enumerate, or name the query that enumerates |
| *"in production"* | Not an environment | Region, cluster, version, and how it was measured |

### 2.3 Transformation table: anti-patterns and clean replacements

| Anti-Pattern | Cold reader's unanswered question | Clean Replacement |
|---|---|---|
| "Significantly faster after the refactor." | Faster on what workload, against what baseline, measured how? | "p95 480 ms → 210 ms, `make bench-reconcile WORKLOAD=prod-sample-100k RUNS=5`, baseline `main @ 4f9c1ab`, median of 5." |
| "Fixes the flaky retry issue." | Which failure, observed where, still fails how often? | "3 of 214 CI runs failed `TestRetryCap` with `context deadline exceeded` (build 4471); 0/214 since `retry.go:64` stopped resetting backoff." |
| "Also cleans up some legacy code." | Which paths, and why in this PR? | "Removes `LegacyReconciler`; `grep -rn LegacyReconciler` returns 0 hits after this commit." |
| "No breaking changes." | Breaking against which contract? | "No API signature, schema, config, or CLI changes; `orders.v1` payload byte-identical on the 100k sample." |
| "Improves performance without changing behavior." | Behavior verified how? | "0 row differences vs. baseline on 100k rows (`scripts/diff_reconcile.py`)." |
| "Fixes a few edge cases." | Which inputs, which outputs? | "Empty `invoice_ids` returns `[]` instead of raising `KeyError` (`handler.go:112`, `TestReconcileEmpty`)." |
| "Refactors to the usual pattern." | The pattern is defined where? | "Extracts `RetryPolicy`, consumed by all 7 previously duplicated call sites in `billing/*.ts`." |
| "Should be safe." | Per which check? | "14 unit cases + new query-count assertion; integration suite green on `main` and on the PR head." |
| "Might be slower under load." | How much, at what load, owned by whom? | "Unverified: no load test. Owner: author, before merge. Expected effect < 5 ms (one extra round trip, `tenant.ts:12`)." |
| "See the diff for details." | A diff states mechanics, never intent | One-sentence change statement + the rationale the diff cannot express |

### 2.4 Failure diagnostics

| Symptom in review | Audit diagnosis | Fix |
|---|---|---|
| "Can you share the numbers?" | Q3 unbound to a command | Bind claim → benchmark, workload, baseline ref |
| "What does this actually fix?" | Q1/Q2 absent; summary is a file list | Lead with the change statement and the observed state |
| "Is this safe to revert?" | Q4 omitted | Add blast radius + single-operation rollback |
| "Which retry issue?" | Tribal reference | Cite incident ID or CI build number |
| Reviewer reverse-engineers intent from the diff | No change statement; intent lives only in code | Write the intent into the summary; keep the diff as the mechanism |
| Thread degenerates into a terminology argument | A premise was never grounded | Ground the premise with an artifact, or downgrade it to `JUDGMENT` |
| Author answers in DM, edits nothing | Answer not patched back into the artifact | Rule 9: the clarification belongs in the PR body |
| PR approved, unexplainable six months later | Approval was social, not evidential | Evidence-bound claims; approvals cite the artifact they verified |

**Related dispatchers.** Bind claims to a falsifiable baseline with the [3-Part Proposal Engine](../../kirby-fitzpatrick-3part-proposal-engine/SKILL.md); strip zero-information intensifiers (*significantly*, *obviously*) with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md); replace weak verbs of state with measurable ones via the [Empty Verb Extractor](../../kirby-fitzpatrick-empty-verb-extractor/SKILL.md); announce the taxonomy before its parts with [Cathedral Taxonomy](../../kirby-fitzpatrick-cathedral-taxonomy/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — Reading a Foreign PR as a Cold Reader, and Commenting That Survives the Audit

Two laws govern the review side. First, **the reviewer's own comment is an artifact subject to the same protocol**. Second, an ungrounded comment is indistinguishable from a preference — and preferences lose to deadlines, which is why unmeasured concerns get resolved by "we'll watch it in prod".

**Reviewer cold-read procedure:**

1. Read the **title and summary only**. Write out answers to Q1–Q5 from that text alone. Every question answered from the diff is a finding: the summary is not self-sufficient.
2. Sort the gaps: **blocking** (decision-critical — correctness, scope, safety) vs. **non-blocking** (curiosity, style, future work).
3. Locate the *minimal artifact* for each gap: for a suspicion, a failing test or a query count; for a claim, the benchmark shape. Do not ask for the author's mental state; ask for the artifact that would settle it.
4. Post comments as **claim + artifact + consequence + falsifiable check**.
5. Never resolve a blocking question off-artifact. If it was answered verbally, ask for the one-line patch to the PR body.

**Before — fails its own audit:**

> This looks wrong — we normally keep tenant scoping in the repo layer. Also this will be slow.

Diagnostics: *"looks wrong"* is an unsourced judgment; *"we normally"* is tribal knowledge with no defined referent; *"will be slow"* is an unmeasured performance claim.

**After — grounded comment:**

```markdown
**Tenant scoping (blocking)**
- Artifact: `src/orders/handler.ts:88` queries `orders` directly; every other read goes through
  `TenantRepo` (`src/repo/tenant.ts:12`).
- Consequence: tenant filtering now lives in two places; the next divergence drops it silently —
  the shape of INC-4412 (2026-02-11).
- Falsifiable check: `grep -rn "from orders" src/orders/` → 1 hit outside the repo layer today.
- Minimal fix: route through `TenantRepo.findOrders()`. One commit, no behavior change.

**Possible extra round trip (non-blocking, unverified)**
- I have no measurement. Requesting the shape rather than asserting the outcome:
  `make bench-orders` before/after, p95, median of 5 runs, same runner.
```

**Acceptance rule.** A blocking comment must contain artifact + consequence + falsifiable check. A suspicion must be labeled a question, never issued as a verdict. A comment that cannot be grounded is either dropped or converted into a request for evidence — never left standing as authority.

### 3.2 PR Descriptions — Claims Causally Bound to Benchmark Evidence

The PR body is the durability layer of the change: the diff records mechanics, the body records **intent and evidence**. Six months later the diff is still readable and the intent is not.

```markdown
## What changed
Reconciliation issues one batched query per batch instead of one query per invoice.

## Why
Reconcile p95 was 480 ms at 100k invoices (metric `reconcile_p95`, 7-day window) and 91% of
that was the N+1 fetch (`artifacts/reconcile-pprof.txt`). On-call logged 4 timeouts in 3 weeks
(INC-4501, INC-4522).

## Evidence — claim ↔ mechanism ↔ measurement
| Claim | Mechanism | Artifact |
|---|---|---|
| p95 480 ms → 210 ms | N+1 fetch → single batch (`handler.go:88`) | `make bench-reconcile WORKLOAD=prod-sample-100k RUNS=5` |
| queries 1,240 → 41 | same | query-count assertion `reconcile_test.go:214` |
| payload unchanged | same assembly path | `scripts/diff_reconcile.py` → 0 row diffs on 100k |

Baseline `main @ 4f9c1ab` · PR head `3a77e02` · runner `gha:ubuntu-22.04`, 4 vCPU ·
median of 5 runs, p95 spread ±6 ms

| Metric | Baseline | PR | Δ |
|---|---|---|---|
| p50 | 120 ms | 88 ms | −27% |
| p95 | 480 ms | 210 ms | −56% |
| queries / reconcile | 1,240 | 41 | −97% |

## Risk, rollback, and what I did not do
- Blast radius: read path only; writes, refunds, payouts untouched.
- Rollback: revert this commit; no schema, config, or flag migration.
- Not included: response cache (deferred, PR #830).
- Unverified: whether `ReconcileAPI` accepts >10k IDs. Owner: author, before merge.
```

**Binding rules that make the table auditable:**

- Every number must be reachable by a command that exists in the repository. If the command is missing, add it or delete the number.
- One baseline, named as a ref. "Before" without a SHA, branch, or tag is not a baseline; it is a memory.
- Report sample size and dispersion. A single run must be labeled *single run*; unlabeled it reads as a measurement.
- Never switch runners, hardware, or workloads silently. If the environment changed, state it and state why the comparison still holds.
- The mechanism goes in prose; the stack trace, profile, or log goes in the artifact. A pasted profiler dump is not a mechanism.
- **Round-trip consistency:** prose must agree with the table. "2× faster" over a −27% p50 row is a defect that destroys trust in every other number in the PR.
- Absence of evidence is a labeled gap with an owner and a date. Silence is read as "verified".
- Reviewer side, the **30-second cold read**: title → evidence table → risk block. Any number without a ref or a command triggers one request — for the ref — not a re-run of the benchmark.

### 3.3 Architecture RFCs / ADRs — Auditing Unsourced Premises

An RFC outlives its authors, its meeting, and often the team. Tribal knowledge hides here most effectively, because a premise reads as common sense: *"all writes go through the `orders` service"* is accepted by everyone in the room and false for one stray write site.

**The Assumption Ledger.** Every premise in the Context section is tagged `FACT`, `JUDGMENT`, or `UNKNOWN` and bound to an artifact and an owner:

| # | Premise as written | Class | Grounding | Owner / date |
|---|---|---|---|---|
| A1 | "All writes go through the `orders` service." | `FACT` (partial) | 5 write sites in `src/orders/**`; 1 stray write at `src/admin/orders.ts:64` | author, 2026-04-02 |
| A2 | "Traffic grows ~15% QoQ." | `FACT` (bounded) | metrics `grafana/traffic-qoq` — 3 quarters only, not seasonally adjusted | author |
| A3 | "The team can absorb a 3-week migration." | `JUDGMENT` | Criterion: staffing now; no artifact of future capacity | EM, decision on 2026-04-10 |
| A4 | "No other consumer reads the v1 field." | `UNKNOWN` | Consumer audit incomplete: 2 repos unscanned | author, **blocking** |

**Rules for the ledger:**

- A `FACT` that cannot be resolved by opening an artifact is demoted to `JUDGMENT` or `UNKNOWN` until grounded.
- A `JUDGMENT` keeps its label; presenting it as a fact is the single most expensive documentation defect, because downstream decisions inherit it silently.
- An `UNKNOWN` names an owner and a date, or the RFC is marked blocking. A named unknown costs one line; an unnamed unknown costs an incident.
- Pin versions and environments in Context. *"In production"* is not an environment; region, cluster, runtime version, and measurement source are.
- The **future-reader clause**: the ADR declares what it assumes about the outside world and what would invalidate it, so a reader in twelve months knows when the decision has expired rather than discovering it in an incident.
- Rejected alternatives must be evidence-backed, not rhetorical: state the cost axis and the measurement that decided it. A verbal decision that never reached the ADR is reconstructed in the ADR — decision, date, deciders, evidence, and the condition that would reverse it.
- An RFC that cannot be understood without its originating meeting is not an RFC. It is private notes with a title.

```markdown
# ADR-014 — Batch reconciliation queries

## Context
`billing-svc` pins `reconcile@2.4.1`; the read path issues one query per invoice
(`handler.go:88`). Reconcile p95 = 480 ms at 100k invoices; 91% of the profile is the N+1
fetch (`artifacts/reconcile-pprof.txt`). Assumption ledger A1–A4 above.

## Decision
Issue one batched query per batch, retaining the existing payload assembly path.

## Consequences
- Measured: p95 480 ms → 210 ms; queries 1,240 → 41 (`make bench-reconcile`, baseline
  `main @ 4f9c1ab`, same runner, median of 5).
- Cost of inaction: 4 timeouts in 3 weeks (INC-4501, INC-4522); query volume grows linearly
  with invoice count, so p95 worsens with traffic, not with load.
- Reverses if: DB batch-size limits below 10k IDs in a future Postgres upgrade (open question
  A4). Rollback: revert this commit; no schema or config migration.
```

---

## 4. Verification Checklist

- [ ] **Zero-context test passes.** A reader with no repository history, no channel access, and no memory of the author's session can answer all five cold-reader questions (what changed, why now, how I know, what breaks / how to undo, what am I asked to do) from the artifact alone; every sentence survives the "I was there" test.
- [ ] **Binding audit passes.** Every claim resolves to an artifact — path + line, commit SHA, command + output, metric, benchmark, or incident ID — and every measurement names workload, baseline ref, runner, and sample size or dispersion. Zero orphan numbers.
- [ ] **Tribal knowledge is purged.** No instance of *as discussed*, *per Slack*, *the usual*, *legacy*, *obviously*, *should be fine*, *the known issue*, *etc.* survives; each was either resolved into a named artifact or deleted.
- [ ] **Causal link is explicit for every measurement.** Each performance or correctness claim states claim → mechanism → measurement, and every number in prose matches the evidence table (no "2× faster" above a −27% row).
- [ ] **Safety block and gap ledger are present.** Blast radius, single-operation rollback, an explicit "not included" list, and every `JUDGMENT` / `UNKNOWN` labeled with an owner and a date — no unlabeled gaps read as verified.