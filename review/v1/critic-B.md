<!-- CRITIC B · hy3-free · family:tencent · pass 2 · 2026-08-29T01:46:31Z -->
CRITIC: hy3-free (family tencent, actor hy3-free@opencode-zen)
DATE: 2026-08-29
PASS: 2
AUTO-TALLIED VERDICT: SALVAGEABLE

---

# Critic review — local-llms-for-manufacturing [v1]

```
CRITIC:    hunyuan hy3 (Tencent family) — operated by RogerAI Labs via OpenCode Zen
DATE:      2026-08-28
PASS:      2 (panel)
READ:      full manuscript
```

## Verdict summary
This is a coherent, unusually disciplined engineering book. Its argument — that small, local, grounded, abstention-trained language models can do useful plant-floor work, but only inside an evaluation-and-recovery instrument a plant builds and owns — holds together across all ten chapters, and its honesty posture (error bars, labeled-unmeasured claims, printed retractions, named verifier, published review trail) is correct and consistent. The central technical logic is sound: the context window as instrument of truth, grammar-constrained decoding as the plant's advantage, abstention as a trained threshold, and the gate as the floor's real authority are each argued from mechanism rather than from marketing. The manuscript is SALVAGEABLE; the blocking findings below are debts of internal consistency (mixed draft states and a citation-convention break) rather than flaws in the argument itself, and are cheaply fixable before verification.

## Blocking findings

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| 1 | ch09 / ch10 (header) vs ch01–ch08 (header) | Manuscript is internally inconsistent in revision state: Ch1–Ch8 are stamped "draft v1, 2026-08-28" while Ch9 and Ch10 are "draft v0, 2026-08-27". A panel Pass-2 expects one coherent draft state; the back half is a version behind the front half. | Front-matter provenance says "Nothing ships until a named human has verified the manuscript," and Ch9/Ch10 are explicitly the least mature — but they are presented inside the same "full manuscript," so a verifier receiving this packet would see mismatched drafts. | med |
| 2 | ch08 / §"The configuration traps" and §"Read the log" (and ch09 CLAUDE.md refs) | Citations to `CLAUDE.md serving traps` and `CLAUDE.md hardware notes` violate the book's own stated lab-citation convention. Back-matter "Lab citation convention" defines `[LAB:]` markers as resolving only into `RESULTS-MATRIX` sections or `PROJECT-LOG` dated entries; CLAUDE.md is a third, undeclared source. | Three in-text markers: `[LAB: CLAUDE.md serving traps — KV-cache precision corrupting output]`, `[LAB: CLAUDE.md serving traps — no-mmap on oversized files]`, `[LAB: CLAUDE.md hardware notes — pkill self-match trap]`. The convention section never lists CLAUDE.md, so the citation contract the book sets for itself is broken. | med |
| 3 | ch08 / §"Sizing the box" + §F references | §F is cited for fit recipes ("IQ3_XXS 102 GB loads; blobfish Q4 175 GB ... Q8-MTP 160 GB; Q3-MTP 143 GB") inside a chapter whose thesis is *small* on-prem models, but these are 100–175B-class footprints. Not a false claim, but the concrete empirical spine of the book is dominated by 27B–175B measurements while the pocket/line-side classes the thesis leans on carry no `[LAB:]` number at all (explicitly "not published"). The mismatch between the argued tier and the measured tier should be stated as a limitation in Ch3/Ch8, not left to the reader to infer. | Ch3: "We do not have a published 7–8B industrial walkthrough score"; sub-billion/1–2B claims are labeled unmeasured; the only small-model `[LAB:]` figures (FailureSensorIQ 44.7; 350M class floors ~random) are weak/negative. The book is honest about each
gap individually but never reconciles the aggregate: its proof is mostly about mid/large models. | med |

## Suggestions (non-blocking)
1. Normalize all chapter headers to a single draft version/date before verification; promote Ch9/Ch10 to v1 or demote the rest, but make them uniform.
2. Either add CLAUDE.md to the "Lab citation convention" list or re-tag those three markers as PROJECT-LOG/RESULTS-MATRIX entries, so the citation contract holds.
3. Add a one-paragraph "evidence base" caveat in Ch3 (and echo in Ch8) stating plainly that the measured `[LAB:]` results skew to 27B–175B and that small-tier claims are presently working rules — this strengthens rather than weakens the book's honesty brand.
4. The "518-page silence" book is never named or dated; give it a real (or clearly illustrative) citation so the opening framing is auditable like everything else.
5. Chapter 7's "escalation-loop gains per cycle are not a published series" and similar are honestly flagged, but a single consolidated "what we have NOT yet measured" index (already partly in Ch10's accounting) would help a verifier trace every unmeasured claim in one place.

## Fact-check sample
Pass 2: 5% of factual claims, chosen across chapters. All sampled claims resolve to their cited source or to the book's own stated arithmetic; no sampled claim was unsupported. (Internal `[LAB:]` records were treated as given per scoping rules; the sample tests internal logical support, not external fetchability.)

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "Stored at full 16-bit precision, one parameter costs two bytes: an 8-billion-parameter model is roughly a 16-gigabyte file." | ch03 / The arithmetic that decides everything | chapter's own parameter×bytes arithmetic | yes |
| "A model in the hundreds of millions of parameters fits in under a gigabyte — single-board-computer territory." | ch03 / The ladder, with honest rungs | derived from same arithmetic (0.5B×4bit≈0.25GB) | yes |
| "embedding 134M vs 621M params, 487M freed for layers" | ch03 / tokenizer is part of the size budget | [LAB: PROJECT-LOG 2026-08-03 + matrix §O.1] | yes (621−134=487, internally consistent) |
| "The first outage cost about an hour and a half of recovery work. The second… about twenty-five minutes" | ch09 / opening | [LAB: PROJECT-LOG 2026-08-22 and 2026-08-24] | yes (internally consistent with the 90-min→25-min delta narrative) |
| "a model handed raw register dumps is being asked to hallucinate the documentation; a model handed decoded values with names and units is being asked to read" | ch04 / What the plant actually says | chapter's own rendering-meaning principle (no external cite, presented as design rule) | yes |

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 4

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
(none — Pass 3 not run)
