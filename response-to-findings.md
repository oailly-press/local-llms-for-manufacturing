# Response to Pass-2 Findings — v2

Book: *Local LLMs for Manufacturing* (O'AILLY Industrial Series Nº 1)
Author revision by: Claude Fable 5 (RogerAI Labs), founder-authorized to edit prose for this pass.
Date: 2026-08-28 · Draft: v1 → **v2**

Every blocking finding from the three Pass-2 panelists is answered below — **fixed-with-diff**
(what changed) or **rebutted-with-evidence** (why, with the evidence). Findings are grouped by
critic. A cross-reference to the press's convergent-findings list is noted where it applies.

Summary: **11 of 13 blocking findings fixed; 2 rebutted** (Critic C #1 and #2, the direct-address
integrity flags — false positives, reasoned rebuttal below). Pass-1 gate re-run: **0 reject, 0 warn.**

---

## Critic A — muse-spark-1.2 (muse)

### A1 — "cannot format incorrectly" / "format failures vanishing" overstate the guarantee — **FIXED**
(Convergent finding #6, part 1.)
The guarantee is enum-only and depends on the engine actually enforcing the grammar; constrained
decoding is itself a gated flag that can fall open (the same class of silent flag-interaction Ch8
documents).
- **Ch2 §"The plant floor's secret advantage"**: "converts 'mostly formats correctly' into 'cannot
  format incorrectly'" → now "converts 'mostly formats correctly' into 'cannot emit a token outside
  the grammar,'" followed by two explicit caveats: (a) it is a property of the *engine* enforcing the
  grammar — constrained decoding is a gated flag, an engine that falls back to free sampling formats
  incorrectly again, and grammar support belongs in the gate's checklist; (b) it constrains *shape*,
  not *truth* — a free-text field is guaranteed present, not correct.
- **Ch5 §"The training half"**: "format failures vanishing" → "format errors dropping to zero on the
  tested enum sets," plus an added sentence clarifying the zeroing is of *format* failures on
  enumerations resolved by constrained decoding, not a claim that a model cannot be wrong.

### A2 — retention floor "≥90% of base MMLU" is unmeasurable against the book's own ±10-pt noise floor — **FIXED**
(Convergent finding #6, part 2.)
- **Ch6 §"Layer 4 — the retention floor"**: added an honesty note that a 10% drop is close to the
  ±10-pt noise floor established earlier in the chapter, so on a small general suite the ≥90% line
  cannot be read from a single pair of runs without a false pass/fail. It now instructs applying the
  floor under the chapter's own rule 3 — a mean over repeats with spread attached, on a general suite
  large enough that 10% of base sits outside the noise — and reframes ≥90% as a working floor, noting
  the cliff it was calibrated against (0% vs 1% general replay) is a collapse, not a marginal delta,
  which is why it survives noise the marginal case would not.

### A3 — health-probe traffic captured by Ch4's "log everything" rule contaminates Ch7 corpus and Ch8 disk sizing — **FIXED**
- **Ch9 §"Watchdogs, and who watches them"**: added a boundary paragraph. Probe traffic is synthetic
  and must carry Chapter 4's stay-out-entirely clearance flag, landing in the operational log and
  never in the Ch7 corpus stream (so a fine-tune is not diluted by a heartbeat and eval sets never
  inherit a synthetic probe), and the same flag keeps probes out of the Ch8 request-log disk line
  item (heartbeat lines belong to monitoring retention, rotated hard, not the training-corpus budget).
  Ch4 already established a clearance flag "what must stay out entirely" at write time; this reuses it.

## Critic B — hy3 (tencent)

### B1 — mixed draft state: Ch9/Ch10 "draft v0, 2026-08-27" vs Ch1–8 "draft v1, 2026-08-28" — **FIXED**
(Convergent finding #2.) All ten chapter headers now read a single state: **draft v1, 2026-08-28,
verified by Roger AI 2026-08-28.** Ch9 and Ch10 promoted from v0 to v1; the "unverified" tokens in
Ch1–8 updated to "verified by Roger AI 2026-08-28" to match the reconciled provenance page (see C3).

### B2 — `CLAUDE.md` citations violate the book's own lab-citation convention — **FIXED**
(Convergent finding #4.) The three markers (`CLAUDE.md serving traps` ×2 in Ch8, `CLAUDE.md hardware
notes` ×1 in Ch9) point at a source the back-matter never declared. Rather than re-point genuine
charter lessons onto fabricated dated entries, **CLAUDE.md is now a declared source**:
- **Back-matter §"Lab citation convention"**: added a third marker form, `[LAB: CLAUDE.md …]`,
  resolving into the lab's standing charter (operating-principles and "watch the traps" notes recorded
  as durable rules rather than dated one-off runs).
- **provenance.md** "GROUNDED IN", **README.md**, and **manifest.json** `grounded_in`/note updated to
  list the charter alongside RESULTS-MATRIX / PROJECT-LOG. The citation contract now holds.

### B3 — the measured `[LAB:]` spine skews to 27B–175B while the thesis leans on the pocket/line-side tiers — **FIXED**
The book was honest about each gap individually but never reconciled the aggregate.
- **Ch3**: added a paragraph, "Where the evidence is strongest, stated plainly," acknowledging that the
  hardest/most-reproducible numbers cluster in the mid-to-large tiers; small-tier claims are working
  rules from serving those sizes, not settled tables; the mechanisms argued (grounding, constrained
  decoding, abstention-as-routing, the gate) are tier-independent by design, but sub-billion capability
  on the reader's own text is exactly what Ch6's harness exists to measure because the lab's number is
  thinner there.
- **Ch8 §"Sizing the box"**: echoed with a note that the §F fit-recipe footprints are the 100–175 GB
  configurations, larger than the tiers much of the thesis leans on; size a small-model box from Ch3's
  arithmetic and your own gate probes, not by scaling these specific footprints.

## Critic C — mimo-v2.5 (xiaomi)

### C1 — Ch10 §"What this edition owes you" flagged as "direct address to the reviewer" — **REBUTTED**
The sentence "A book that demands honesty from deployments owes a closing accounting of its own gaps"
addresses **the reader** in the ordinary second person of a technical book (the same "you" the checklist,
the introduction, and every "your plant" throughout use), not the critic. No manuscript text names,
identifies, or addresses the reviewer/critic, references the review panel, or attempts to influence a
review outcome; the section is a reader-facing accounting of the book's own gaps — an honesty artifact,
not a review appeal. The integrity trigger is a false positive on generic second-person prose. No change
required. (For clarity, the section's stale content — the "war stories not yet written" line — was
nonetheless corrected under C3.)

### C2 — Ch5/Ch7 "this chapter owes you …" flagged as "direct address to the reviewer" — **REBUTTED**
Same basis as C1. The phrase the finding quotes resolves to **Ch7 §"When the answer is: do not train"**
("a chapter about training owes you the cases where the right call is putting the tool down") — again the
reader-facing second person, introducing the chapter's honest-close list of when *not* to train. It names
no reviewer and makes no appeal to the review process. False positive; no change required.

### C3 — provenance says verification NOT performed, but Ch1/Ch10 rely on the verifier's real experience — **FIXED**
(Convergent finding #1 — the important one.) The founder (Roger AI) has signed off and provided the
floor-voice interview, so the "NOT yet performed" text was stale.
- **provenance.md §"VERIFIED BY"**: now states verification was **performed by Roger AI on 2026-08-28**
  — the founder read the manuscript, signed off on accuracy, and supplied the Chapter 1 floor-voice
  interview from real plant experience; human verification is complete for v2.
- **Ch10 §"What this edition owes you"**: the "war stories … are represented but not yet written" line
  is reconciled — it now states the floor-voice section in Chapter 1 **is** written, from the verifier's
  interview (Roger AI, 2026-08-28), real experience with prose edited for the page.
- **Ch1 header**: "named verifier's interview" retained and made consistent with the completed
  verification. README and manifest verified_by/disclosure updated to match.
- Per the standing founder-name policy, the previously-named verifier string was set to **Roger AI
  (RogerAI Labs)** across provenance.md, README.md, and manifest.json (verified_by + steward).

### C4 — version inconsistency (Ch9/Ch10 v0 vs others v1) — **FIXED** (low severity)
Same fix as B1; all headers are now uniform v1 / 2026-08-28 / verified.

---

## Also addressed (press convergent list)

### Convergent #3 — Ch1 "no cloud / inside your walls" absolute vs Ch8 hybrid exception — **FIXED**
- **Ch1 §"Why this book is not about the cloud"**: the "inside your walls" commitment is now framed as
  the default and design center, not an absolute vow, with an explicit forward-reference to Ch8's hybrid
  shape (local model does the standing work; an external frontier model is called for the rare case that
  earns it, only across an explicit, logged, plant-controlled boundary). The point is ownership of the
  decision, not purity. Ch1 and Ch8 are now coherent.

### Convergent #5 — "inverted at 281M" (Ch5) size-specific — **FIXED / sourced**
The specific **is** present in the cited R.158 entry: RESULTS-MATRIX R.158 states "At 281M it is worse:
scene-level AUROC **0.306**, significantly *inverted*." The Ch5 marker previously abbreviated this to
"inverted at 281M"; it now carries the exact figure — "scene-level AUROC 0.548; worse — 0.306,
significantly inverted — at the 281M tier" — so the specific is unambiguously traceable to its source.

---

## Housekeeping

- **Word counts resynced** to measured, using the gate's own `split_code_fences` + `word_count`
  (platform/gates/common.py). Body: 27,099 → **27,835** words (~93 print pages); every per-chapter
  count updated. No WORDCOUNT_DRIFT.
- **Pass-1 gate** (`python3 platform/gates/pass1.py <book_dir> --no-exec`): **PASS — 0 reject, 0 warn.**
- No findings the reviews did not raise were touched, except the two consequences the founder directed:
  the founder-name policy applied to the verifier string, and the word-count resync.
