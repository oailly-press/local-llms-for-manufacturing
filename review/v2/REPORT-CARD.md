# Final report card — rogerai-labs--local-llms-for-manufacturing v2

Generated mechanically from the immutable two-pass review trail. The judge must
read the underlying reviews; this card indexes evidence and does not replace it.

## Case provenance

- v1 commit: `e1482825a6c9d134a5e09f4f65901dc0ca54b4cd`
- v2 commit: `199a684769dbea6807862217a23f05ee10026dae`
- author response SHA-256: `3e00314e7ebfb4ce4ece207dbf7e72edec1e453b2bbb8c417a4f88cc22e1e625`
- Pass-2 reviews: 3; Pass-3 verification reviews: 3

## Panel recommendation

Mechanical tally: **ADVANCE to judge (PUBLISH-leaning)**.
Verdicts: seat A = PUBLISH, seat B = PUBLISH, seat C = PUBLISH.

## Evidence fingerprints

| Pass | Seat | File | SHA-256 |
|---|---|---|---|
| 2 | A | `review/v1/critic-A.md` | `e7df9ae0b5987abf2d3c48adb1bcc7a5f39da286589d0a78caeeebac60bbc469` |
| 2 | B | `review/v1/critic-B.md` | `35956cfed8116fff51acf669b6f528f703f96add72b5dd50dc818e1ba981ad8b` |
| 2 | C | `review/v1/critic-C.md` | `dff5966632b21cf4371c47a5baede3a9541a7b8cc73edd810dae3c8963d1dbc9` |
| 3 | A | `review/v2/verify-A.md` | `a09463efb7b13c2e46f4ba27ef2f2c203d3edbde5e31f7a7e75f7779968c0c98` |
| 3 | B | `review/v2/verify-B.md` | `8871f7f0573fb0e933d8466c75f684605857a5a993c65e6573f75d646d64efcc` |
| 3 | C | `review/v2/verify-C.md` | `86f3c0c054d85d5ce78232d1a9b2e6a6b866e13cee4076f6e24de0d3ba7692e0` |

## Seat A — muse-spark-1.2-contributor-free (family muse, actor muse-spark-1.2-contributor-free@opencode-zen)

Recorded recommendation: **PUBLISH**.

### Recommendation reasoning

Delta v1→v2 addresses all 10 blocking findings from the Pass-2 panel with diff-visible fixes or valid rebuttals. Overstatement of constrained decoding, unmeasurable retention floor, probe-corpus contamination, mixed draft headers, undeclared CLAUDE.md source, evidence-base skew, and provenance/verification inconsistency are all reconciled in prose and cited markers; the two integrity flags (C1, C2) are false positives on ordinary reader-directed "you." No new blocking issues introduced. Pass-1 gate re-run 0 reject/0 warn corroborates. **PUBLISH**

### Findings ledger

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

### Score evidence (Pass 2 → Pass 3)

Pass 2:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 5

Pass 3:

accuracy: 4 clarity: 5 completeness-for-tier: 4 density: 4 originality: 4

## Seat B — hy3-free (family tencent, actor hy3-free@opencode-zen)

Recorded recommendation: **PUBLISH**.

### Recommendation reasoning

This is a Pass-3 delta verification. The author shipped v2 with a response-to-findings.md answering all 13 Pass-2 blocking findings. Verification against the v2 manuscript and the v1→v2 diff confirms: findings A1, A2, A3, B1, B2, B3, C3, C4 are FIXED with concrete, traceable diffs; C1 and C2 were REBUTTED as false positives on generic reader-directed second-person prose, and that rebuttal is correct under the press's own review rules (reader "you" is normal prose, not an integrity issue), so those findings were non-blocking false positives. No manuscript content addresses the critic, panel, or judge — no integrity violation. **PUBLISH** (verified): every Pass-2 blocking finding is either resolved by edit or correctly rebutted; the residual fact-check limitation below is a seat-access constraint, not a manuscript defect.

### Findings ledger

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

### Score evidence (Pass 2 → Pass 3)

Pass 2:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 4

Pass 3:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 4

## Seat C — mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)

Recorded recommendation: **PUBLISH**.

### Recommendation reasoning

All 13 Pass-2 blocking findings have been addressed: 11 fixed with textual changes, 2 rebutted with valid reasoning (the direct-address integrity flags were false positives on reader-directed second-person prose). The fixes are substantive — A1 narrows the constrained-decoding guarantee properly, A2 makes the retention floor noise-aware, A3 adds the clearance-flag boundary for synthetic probe traffic, B1–B3 resolve version uniformity, citation-convention integrity, and evidence-base asymmetry transparency. The rebuttals on C1 and C2 are correct: "you" throughout the text is reader-directed technical prose, not reviewer-directed content. **PUBLISH**

### Findings ledger

| Finding # (from Pass 2) | Status | Note |
|---|---|---|
| A1 [med] — constrained-decoding overstatement | RESOLVED | Ch2 now says "cannot emit a token outside the grammar" with two explicit caveats (engine-dependent, shape-not-truth). Ch5 clarifies zeroing is format-only on enum-decoded enumerations. |
| A2 [med] — retention floor vs noise floor | RESOLVED | Ch6 adds honest note: 10% drop ≈ ±10-pt noise; floor requires mean-over-repeats on a general suite large enough that 10% of base exceeds measured noise. Reframed as "working floor." |
| A3 [med] — probe traffic corpus contamination | RESOLVED | Ch9 adds boundary paragraph: synthetic probes carry stay-out-entirely flag, land in operational log only, excluded from Ch7 corpus and Ch8 disk budget. |
| B1 [med] — mixed draft versions | RESOLVED | All headers now uniform: draft v1, 2026-08-28, verified by Roger AI. |
| B2 [med] — CLAUDE.md citation undeclared | RESOLVED | Back-matter now declares third citation form [LAB: CLAUDE.md …]; provenance, README, manifest updated. |
| B3 [med] — evidence-base mismatch | RESOLVED | Ch3 adds "Where the evidence is strongest, stated plainly" paragraph; Ch8 echoes with note that §F footprints are the larger tier. |
| C1 [med] — Ch10 "direct address to reviewer" | REBUTTED-ACCEPTED | Correct: "you" is reader-directed technical prose throughout. No reviewer or review process is addressed. |
| C2 [med] — Ch5/Ch7 "direct address to reviewer" | REBUTTED-ACCEPTED | Same basis as C1. False positive on reader-facing second person. |
| C3 [med] — verification status inconsistency | RESOLVED | Provenance now states verification performed by Roger AI on 2026-08-28; Ch10 reconciles floor-voice as written from interview; verifier string set to "Roger AI (RogerAI Labs)" across all files. |
| C4 [low] — version inconsistency | RESOLVED | Same fix as B1. |

### Score evidence (Pass 2 → Pass 3)

Pass 2:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 5 · originality: 4

**Accuracy (4):** Strong. Lab citations are thorough, unmeasured claims are honestly labeled. One factual inconsistency between provenance and chapter attribution must be resolved. **Clarity (5):** Exceptional. The prose is direct, jargon is built from scratch in Ch2, and the "mental model" framing (well-read colleague with no documents) is the best LLM-onboarding metaphor this reviewer has encountered. **Completeness-for-tier (4):** Covers the full stack from protocols to deployment to checklist. The glossary is thorough. Minor gaps: the "518-page book" is unidentified; the escalation-teacher loop (Ch7) deserves a worked example. **Density (5):** High signal-to-noise ratio throughout. Every paragraph earns its place. The worked examples (Ch2 three-way, Ch4 rendering comparison, Ch5 calibration ladder) are excellent. **Originality (4):** The abstention-first design philosophy, the escalation-teacher training loop, and the honest-deployment-checklist framing are genuinely original contributions to the industrial-AI literature. The provenance/review-trail model for a technical book is unprecedented and valuable.

Pass 3:

accuracy: 5 · clarity: 5 · completeness-for-tier: 5 · density: 5 · originality: 5

## Judge handoff

The judge reviews the manuscript, full Pass-2 findings, author response, exact
v1→v2 delta, all Pass-3 ledgers, and this report card. Still-open findings, if
any, remain visible; the mechanical tally does not sign or determine publication.
