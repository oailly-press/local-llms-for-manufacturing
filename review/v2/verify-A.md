<!-- CRITIC A · muse-spark-1.2-contributor-free · family:muse · pass 3 · 2026-08-29T02:38:41Z -->
CRITIC: muse-spark-1.2-contributor-free (family muse, actor muse-spark-1.2-contributor-free@opencode-zen)
DATE: 2026-08-29
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — local-llms-for-manufacturing v2

```
CRITIC:    muse-spark-1.2 (Muse) (family muse, operator muse-spark-1.2-contributor-free@opencode-zen)
DATE:      2026-08-28
PASS:      3 (verification)
READ:      delta (revised sections: provenance, ch01, ch02, ch04, ch05, ch06, ch07, ch08, ch09, ch10, backmatter, manifest, README)
```

## Verdict summary
Delta v1→v2 addresses all 10 blocking findings from the Pass-2 panel with diff-visible fixes or valid rebuttals. Overstatement of constrained decoding, unmeasurable retention floor, probe-corpus contamination, mixed draft headers, undeclared CLAUDE.md source, evidence-base skew, and provenance/verification inconsistency are all reconciled in prose and cited markers; the two integrity flags (C1, C2) are false positives on ordinary reader-directed "you." No new blocking issues introduced. Pass-1 gate re-run 0 reject/0 warn corroborates. **PUBLISH**

## Blocking findings
Debts, not advice. Author must fix-with-diff or rebut-with-evidence, every one.

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| — | — | No new blocking findings in delta; all Pass-2 debts verified resolved/rebutted below | — | — |

## Suggestions (non-blocking)
1. Consider adding a one-line pointer in Ch4's "log everything" rule to the Ch9 exclusion to prevent future reader misapplication.
2. Ch3's new limitation paragraph is strong; echoing its exact wording in the Introduction's claim-boundary section would aid skimmers.
3. Retain word-count resync method in publisher ops notes to prevent WORDCOUNT_DRIFT on next edit pass.

## Fact-check sample
Pass 2: 5% of factual claims, chosen randomly — list claim, cited source, and whether the source actually supports it. Pass 3: fresh 3% weighted toward revised sections.
A claim whose cited source does not support it = automatic blocking finding above.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "converts 'mostly formats correctly' into 'cannot emit a token outside the grammar'" | ch02: The plant floor's secret advantage | [LAB: RESULTS-MATRIX R.158 — deterministic enum decode] | partly — qualified in v2 as enum-only, engine-enforced, shape-not-truth; claim now matches the limited guarantee; full source text not independently resolvable in this offline seat — consistency judged on packet labels |
| "format errors dropping to zero on the tested enum sets" | ch05: The plumbing half | [LAB: RESULTS-MATRIX R.158 — format is locked by deterministic enum decode] | partly — v2 narrows to format failures on enumerations; supported as stated scope; offline seat cannot open lab record — noted limitation |
| "a 10% retention drop is close to the ±10-point noise floor" | ch06: Layer 4 — retention floor | [LAB: RESULTS-MATRIX §C footnote — ±10 pts across three identical runs at temp 0] + [LAB: PROJECT-LOG — 0% vs 1% general replay cliff] | yes — consistent with book's own noise measurement; applies rule 3 (mean over repeats) correctly |

Note: Per Pass-3 instructions (no tools), lab records (RESULTS-MATRIX/PROJECT-LOG/CLAUDE.md) were not independently resolved via tool access; sample judged against packet-provided citations and delta. Operator must rerun with tool access if independent source resolution is required.

## Scores (1–5)
accuracy: 4 clarity: 5 completeness-for-tier: 4 density: 4 originality: 4

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| A1 (constrained decoding overstatement) | resolved | Ch2 now "cannot emit a token outside the grammar" + 2 caveats (engine-enforced gated flag, shape-not-truth); Ch5 narrows to "tested enum sets" + explicit format-vs-judgment — diff verified |
| A2 (retention floor vs noise floor) | resolved | Ch6 adds honesty note: ≥90% cannot be read off single pair; requires mean over repeats, suite large enough that 10% sits outside noise; reframed as working floor with collapse vs marginal rationale — diff verified |
| A3 (probe log contamination) | resolved | Ch9 Watchdogs adds boundary paragraph: synthetic probe carries stay-out-entirely flag, to operational log not corpus, excluded from Ch8 disk budget — reuses Ch4 flag — diff verified |
| B1 (mixed draft v0/v1 headers) | resolved | All chapter headers now uniform "draft v1, 2026-08-28, verified by Roger AI 2026-08-28"; Ch9/10 promoted from v0 — diff verified |
| B2 (CLAUDE.md violates citation convention) | resolved | Backmatter adds third form [LAB: CLAUDE.md …] as standing charter; provenance/README/manifest grounded_in updated — contract now holds — diff verified |
| B3 (spine 27B–175B vs pocket thesis) | resolved | Ch3 adds "Where the evidence is strongest" limitation paragraph; Ch8 Sizing adds honest asymmetry note — aggregate limitation now stated — diff verified |
| C1 (Ch10 "owes you" direct address) | rebutted-accepted | Reader-directed "you" per review rules; no critic/panel/judge address; false positive — rebuttal accepted, stale war-story line also fixed under C3 |
| C2 (Ch05/Ch07 "owes you" direct address) | rebutted-accepted | Same as C1 — Ch07 "chapter owes you" is reader-facing when-not-to-train list; no reviewer appeal — false positive — rebuttal accepted |
| C3 (provenance verification inconsistency) | resolved | Provenance now "Verification performed by Roger AI on 2026-08-28"; Ch10 now says floor-voice written from verifier interview; Ch1 header/README/manifest aligned — diff verified |
| C4 (v0/v1 version inconsistency low) | resolved | Same fix as B1 — uniform v1 headers — diff verified |
