<!-- CRITIC B · hy3-free · family:tencent · pass 3 · 2026-08-29T02:38:54Z -->
CRITIC: hy3-free (family tencent, actor hy3-free@opencode-zen)
DATE: 2026-08-29
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — local-llms-for-manufacturing v2

```
CRITIC:    hunyuan hy3 (Tencent) (family tencent, operator hy3-free@opencode-zen)
DATE:      2026-08-28
PASS:      3 (verification)
READ:      delta (revised sections: provenance, ch01, ch02, ch03, ch04, ch05, ch06, ch07, ch08, ch09, ch10, backmatter, manifest, README, response-to-findings)
```

## Verdict summary
This is a Pass-3 delta verification. The author shipped v2 with a response-to-findings.md answering all 13 Pass-2 blocking findings. Verification against the v2 manuscript and the v1→v2 diff confirms: findings A1, A2, A3, B1, B2, B3, C3, C4 are FIXED with concrete, traceable diffs; C1 and C2 were REBUTTED as false positives on generic reader-directed second-person prose, and that rebuttal is correct under the press's own review rules (reader "you" is normal prose, not an integrity issue), so those findings were non-blocking false positives. No manuscript content addresses the critic, panel, or judge — no integrity violation. **PUBLISH** (verified): every Pass-2 blocking finding is either resolved by edit or correctly rebutted; the residual fact-check limitation below is a seat-access constraint, not a manuscript defect.

## Blocking findings
(Pass-2 debts carried for ledger; all cleared in v2.)

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| A1 | ch02 "plant floor's secret advantage"; ch05 "training half" | grammar-constrained decoding overstated ("cannot format incorrectly", "failures vanishing") | v2 rewords to "cannot emit a token outside the grammar" + two caveats (engine-gated flag; shape not truth); ch05 "format errors dropping to zero on tested enum sets" + clarification. Diff confirms. | med |
| A2 | ch06 "Layer 4 retention floor" | "≥90% of base MMLU" unmeasurable vs book's own ±10-pt noise floor | v2 adds honesty note: 10% drop ≈ noise floor; apply as mean-over-repeats w/ spread; "working floor" reframed. Diff confirms. | med |
| A3 | ch09 vs ch04/ch07/ch08 | health-probe traffic captured by "log everything" contaminates Ch7 corpus + Ch8 disk | v2 ch09 adds boundary paragraph: probe traffic carries stay-out clearance flag, lands in operational log not corpus/disk line. Diff confirms. | med |
| B1 | ch09/ch10 headers vs ch01-08 | mixed draft state (v0/2026-08-27 vs v1/2026-08-28) | all headers now "draft v1, 2026-08-28, verified by Roger AI 2026-08-28". Diff confirms. | med |
| B2 | ch08/ch09 | `[LAB: CLAUDE.md …]` violates book's own citation convention | back-matter, provenance, README, manifest now declare CLAUDE.md as third marker form. Diff confirms. | med |
| B3 | ch03/ch08 | measured [LAB:] spine skews 27B–175B while thesis leans pocket/line-side | ch03 "Where the evidence is strongest" + ch08 sizing note acknowledge the asymmetry as limitation. Diff confirms. | med |
| C1 | ch10 "What this edition owes you" | flagged direct address to reviewer | REBUTTED: reader-facing "you", false positive per rules. No change required. | med |
| C2 | ch05/ch07 "this chapter owes you" | flagged direct address to reviewer | REBUTTED: reader-facing "you", false positive per rules. No change required. | med |
| C3 | provenance vs ch01/ch10 | provenance said verification NOT performed yet relied on verifier's experience | v2 provenance "VERIFIED BY Roger AI … performed 2026-08-28"; ch10 reconciled; consistent "Roger AI" identity. Diff confirms. | med |
| C4 | ch09/ch10 v0 vs others v1 | version inconsistency (same fix as B1) | resolved via B1 uniform headers. Diff confirms. | low |

## Suggestions (non-blocking)
1. C1/C2 rebuttal is sound, but consider a one-line author's note in the review-trail that reader-directed second person is intentional house style, so future panelists don't re-flag it.
2. The v2 still contains several `[LAB:]` markers pointing at unpublished public-trail entries (e.g. §O.1, R.158, §G soak, MAXQ-THERMAL). The "What this edition owes you" section already flags this honestly; ensure the public trail actually attaches these before publication, or the Pass-1 "0 warn" will not survive a reader who checks citations.
3. Founder-name policy collapsed the named verifier to "Roger AI (RogerAI Labs)" — a pseudonymous lab entity, not a distinct human personal name. This is internally consistent and acceptable, but the provenance claim "the founder read the manuscript" could name the founder once for stronger verifiability.
4. Minor: manifest.json `verified_by.contact` points to github.com/rogerai-labs (an org, not a person) — fine, but the "named human verifier" standard is arguably softened; acceptable given disclosed policy.

## Fact-check sample
Pass 3: fresh sample weighted toward revised sections. Independent resolution limitation stated below.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "Qwen3.6-27B dense Q8_0 is 29 GB / 79.0 MMLU / ~27 tok/s; … DeepSeek-V4-Flash Q8-MTP master is 149 GB / 88.3 MMLU / 26–27 tok/s" | ch01 §"What changed" | [LAB: RESULTS-MATRIX §C/§F] | partly — internally consistent with ch03's identical figures (88.3 MMLU / 26–27 tok/s) and ch08 §F; source not independently resolvable by this seat (see note) |
| "format is locked by deterministic enum decode; remaining errors concentrate … (scene-level AUROC 0.548; worse — 0.306, significantly inverted — at the 281M tier)" | ch05 §"plumbing half" | [LAB: RESULTS-MATRIX R.158] | partly — internally consistent with ch02 R.158 citation (87.61% / AUROC 0.938) and convergent #5 fix text; R.158 not independently resolvable by this seat |
| "a from-scratch 32k tokenizer … freeing 487M parameters for layers at fixed budget" | ch02 §"Tokens" | [LAB: PROJECT-LOG 2026-08-03 + matrix §O.1] | partly — cross-consistent with ch03 "embedding 134M vs 621M params, 487M freed"; PROJECT-LOG not independently resolvable by this seat |
| "retain ≥90% of base MMLU and IFEval; 0% general replay produced the cliff between 0% and 1%" | ch06 §"Layer 4" | [LAB: PROJECT-LOG] | partly — internally consistent across ch03/ch06/ch07; not independently resolvable by this seat |

**Independent-resolution limitation:** all four sampled claims cite the RogerAI Labs internal record (RESULTS-MATRIX / PROJECT-LOG / CLAUDE.md), which is not provided as a resolvable URL or file in this packet and which the seated review was instructed not to fetch with tools. I therefore cannot independently confirm the figures against the cited sources; I verified only internal cross-consistency within the manuscript. Per the audit rule, this sample is NOT called verified — the operator should rerun the seat (or supply the lab record) for a fully independent citation check. This is a seat-access constraint, not a manuscript defect, and does not block the Pass-3 verdict because the claims are mutually consistent and the author's own provenance page invites exactly this scrutiny.

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 4

## Pass-3 only: findings ledger

| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| A1 | resolved | ch02/ch05 reworded + caveats present in v2 text and diff |
| A2 | resolved | ch06 noise-aware honesty note present in v2 text and diff |
| A3 | resolved | ch09 boundary paragraph present in v2 text and diff |
| B1 | resolved | all 10 chapter headers uniform v1/2026-08-28/verified in v2 + diff |
| B2 | resolved | CLAUDE.md declared in back-matter/provenance/README/manifest in v2 + diff |
| B3 | resolved | ch03 + ch08 limitation paragraphs present in v2 + diff |
| C1 | rebutted-accepted | reader-directed "you"; false positive per press rules; correct rebuttal, no edit needed |
| C2 | rebutted-accepted | reader-directed "you"; false positive per press rules; correct rebuttal, no edit needed |
| C3 | resolved | provenance "VERIFIED BY Roger AI … 2026-08-28"; ch10 reconciled; consistent identity in v2 + diff |
| C4 | resolved | same fix as B1; uniform headers in v2 + diff |
