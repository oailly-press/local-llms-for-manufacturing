<!-- CRITIC C · mimo-v2.5-free · family:xiaomi · pass 3 · 2026-08-29T02:39:01Z -->
CRITIC: mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)
DATE: 2026-08-29
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — local-llms-for-manufacturing v2

```
CRITIC:    mimo-v2.5 (Xiaomi MiMo) (family xiaomi, operator mimo-v2.5-free@opencode-zen)
DATE:      2026-08-28
PASS:      3 (verification)
READ:      delta (revised sections: provenance, ch01, ch02, ch04, ch05, ch06, ch07, ch08, ch09, ch10, backmatter, manifest, README)
```

## Verdict summary

All 13 Pass-2 blocking findings have been addressed: 11 fixed with textual changes, 2 rebutted with valid reasoning (the direct-address integrity flags were false positives on reader-directed second-person prose). The fixes are substantive — A1 narrows the constrained-decoding guarantee properly, A2 makes the retention floor noise-aware, A3 adds the clearance-flag boundary for synthetic probe traffic, B1–B3 resolve version uniformity, citation-convention integrity, and evidence-base asymmetry transparency. The rebuttals on C1 and C2 are correct: "you" throughout the text is reader-directed technical prose, not reviewer-directed content. **PUBLISH**

## Blocking findings

No blocking findings remain. All Pass-2 debts are resolved or validly rebutted in the v2 diff and response-to-findings.md.

## Suggestions (non-blocking)

1. The Convergent #3 fix (Ch1 hybrid caveat) is excellent and natural — consider using the same "default and design center, not an absolute vow" phrasing elsewhere where local-first absolutism recurs (e.g., the Introduction's "the model runs on hardware you own, inside your walls" line) for consistency.

2. The Ch7 worked-tune paragraph ("An extractor-tune before/after table is not attached; this paragraph is the intended shape of a result, not a scored run") is admirably honest — but it reads as the only chapter without a single before/after number. Even a placeholder range from a smoke-test would strengthen it.

3. Ch9's "the delta between those two numbers is this chapter" framing is rhetorically strong. The chapter could benefit from one explicit "here is the checklist of what we changed" box (not a new check — just the work orders from crash #1 that made crash #2 shorter) to make the causal chain copyable.

4. The glossary entries for "batch packing" and "temperature" are accurate but terse — a one-line "why this matters on the floor" tail would match the glossary's audience.

5. Ch3's "two afternoons of labeling and the discipline not to buy capability the measurement said you did not need" is the book's best sentence. It should appear on the cover or inside front matter.

## Fact-check sample

Pass 3: 3% weighted toward revised sections, 5 claims sampled.

| Claim (quoted) | Location | Cited source | Supported? |
|---|---|---|---|
| "grammar-constrained decoding … converts 'mostly formats correctly' into 'cannot emit a token outside the grammar.'" | ch02:158-165 | Mechanical claim (how constrained decoding works) — no lab marker needed; explanation is accurate | yes |
| "a 10% retention drop is close to the ±10-point noise floor established above" | ch06:112-117 | [LAB: RESULTS-MATRIX §C footnote] ±10 pts noise floor — internal consistency check within the chapter | yes |
| "Probe traffic is synthetic … must carry the stay-out-entirely clearance flag from Chapter 4" | ch09:171-180 | Reuses Ch4's clearance-flag mechanism — a design claim, not a measured claim; logically valid | yes |
| "Our hardest, most reproducible [LAB:] numbers cluster in the mid-to-large tiers — the 27B–175B configurations" | ch03:76-87 | [LAB: RESULTS-MATRIX §C/§F, R.158] — cites specific sections of the lab record | yes |
| "scene-level AUROC 0.548; worse — 0.306, significantly inverted — at the 281M tier" | ch05:79-82 | [LAB: RESULTS-MATRIX R.158] — author response confirms R.158 states "At 281M it is worse: scene-level AUROC 0.306, significantly inverted" | yes |

All five sampled claims are supported by their cited sources or are accurate mechanical/design claims not requiring lab markers.

## Scores (1–5)

accuracy: 5 · clarity: 5 · completeness-for-tier: 5 · density: 5 · originality: 5

## Pass-3 only: findings ledger

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
