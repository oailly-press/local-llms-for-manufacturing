<!-- CRITIC A · muse-spark-1.2-contributor-free · family:muse · pass 2 · 2026-08-29T01:45:58Z -->
CRITIC: muse-spark-1.2-contributor-free (family muse, actor muse-spark-1.2-contributor-free@opencode-zen)
DATE: 2026-08-29
PASS: 2
AUTO-TALLIED VERDICT: SALVAGEABLE

---

# Critic review — Local LLMs for Manufacturing [pocket v1] [REV 1.0 draft 2026-08-28]

```
CRITIC:    muse-spark-1.2 (Muse family) — operated by RogerAI Labs via OpenCode Zen
DATE:      2026-08-28
PASS:      2 (panel)
READ:      full manuscript
```

## Verdict summary
Full manuscript read cover-to-cover. This is a strong pocket-tier draft with a coherent physical thesis — small local models are useful precisely when wrapped in deterministic rendering, schema-constrained decoding, abstention-as-routing, and a noise-aware evaluation gate — argued consistently from mental model (Ch2) through sizing (Ch3), plumbing (Ch4), abstention (Ch5), gate (Ch6), training (Ch7), deployment and reality (Ch8-9) to checklist (Ch10). The honesty posture is load-bearing and correctly applied: every unmeasured claim that is marked as such is not counted as a defect per scoping, [LAB:] markers are treated as internal lab pointers not public citations, and the founder floor-voice is treated as in-scope experience. Technical narrative is clear, plant-grounded, and original in its integration of calibration, retention floors, and ops. Remaining debts are fixable within the draft's architecture — overstatements about constraint guarantees, an undetectable retention threshold against the book's own noise floor, and probe/corpus contamination — and do not require restructure. **SALVAGEABLE — findings below**

## Blocking findings
Debts, not advice. Author must fix-with-diff or rebut-with-evidence, every one.

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| 1 | ch04-reading-the-plant.md:The output side: schemas as guardrails | Claims grammar-constrained decoding "converts 'mostly formats correctly' into 'cannot format incorrectly'" and "format failures vanishing" as universal. Overstates guarantee: cited enum-decode result is enum-only (IEB-Signals), but book also uses JSON schemas with free-text `evidence`/`what's missing` fields where grammar can enforce field presence but not quote-truth or enum validity if engine falls back. Ch8 shows engine flags silently degrade behavior; constrained decoding is itself a gated engine flag that can fail open. | Ch4: "cannot format incorrectly" + "remaining errors becoming judgment errors" citing [LAB: R.158 — bit-exact 640 rows, 0 flips]; Ch8: KV-cache precision corrupting output, --no-repack/mmap, placement matrix — same class of silent flag interaction not qualified for constrained decode. Fix: qualify to deterministic enum decode and add fallback behavior + gate check for grammar support. | med |
| 2 | ch06-the-quality-gate.md:Anatomy Layer 4 / ch03-the-specialist-trap.md / ch06-noise first | Retention floor "retain ≥90% of base MMLU and IFEval" is unmeasurable against the book's own noise floor. Ch06 establishes ±10 pts across three identical runs on a 15-scenario suite at temperature 0 due to batch-packing nondeterminism, and requires publishing ranges and thresholds outside noise floor. For a 79.0 MMLU base (Qwen3.6-27B) 90% = 71.1, delta 7.9 pts is *inside* the ±10 pt noise floor; for small task suites the floor is even noisier. Gate will false-pass or false-fail retention non-deterministically. | Ch6 footnote [LAB: §C — ±10 pts, 15-scenario tool suite] + §Anatomy Layer 4 "[LAB: PROJECT-LOG — retain ≥90%]" + Ch3 specialist trap same citation. No noise-aware retention rule (mean over repeats, spread, or larger general suite) is defined. | med |
| 3 | ch09-surviving-reality.md:Watchdogs, and who watches them / ch04:corpus you are accidentally building / ch08:sizing | Health probe "send a real, tiny inference request on a timer; require a token back" will be captured by Ch4's standing rule "log every rendering and every answer" as trainable corpus and as disk growth, contaminating Chapter 7 training data with synthetic probes and invalidating Chapter 8 disk sizing (logs dominate within a year). No exclusion is stated; probe traffic also risks train/eval contamination boundary. | Ch9: "Log the boring successes, too... timestamped line"; Ch4: "log every rendering and every answer... building exact training set Chapter 7 needs"; Ch8: "logs dominate within a year on busy box"; Ch6/Ch7 contamination checks assume train/eval separation but not probe exclusion. Fix: mark probe requests with non-trainable flag and exclude from corpus and gate. | med |

## Suggestions (non-blocking)
Structure, ordering, missing topics, tone. Numbered list.

1. **Make the 90-day plan reference the noise floor explicitly.** Ch10 Weeks 1-2 promises "noise floor measured five times" — explicitly tie to Ch6 rule 3 (third run on disagreement + control re-bench) so first-time builders don't ship a 100-case gate whose thresholds sit inside noise.

2. **Add a one-paragraph tokenizer worked example to Ch3 ladder.** Ch2/Ch3 correctly argue tokenizer is part of size budget (+3.5–7.9% chars/token, 487M params freed). A before/after rendering of `LINE3_CONV_VFD2_AMPS` token count would make the sizing rule ("more capability would fix vs better grounding/tokenizer would fix") actionable for buyers comparing same-parameter models.

3. **Promote the hierarchy escalation contract to a schema.** Ch3/Ch5/Ch8 describe abstain-as-routing and Ch7 escalation-teacher loop, but no example schema for the escalation envelope (context + question + named gap + clearance flags). Adding it would close the loop between Ch5's six grades of no and Ch8's "data flows up" boundary.

4. **Quantify the watcher dwell policy.** Ch4 watcher sets "dwell before speaking" and "track hit rate" qualitatively (correct per scoping). A worked disposition taxonomy (useful/noise/already-known) with a floor example (e.g., retire below X% useful) would help maintainers operationalize Ch6's "green gate with ignored output is failure."

5. **Consolidate thermal and sizing guidance.** Ch9 MAXQ-THERMAL and Ch8 worksheet both discuss sustained vs burst throughput. Cross-reference the commissioning baseline measurement procedure (season, load, duration) in both places so procurement's "summer conditions" line has a test to point to.

6. **Glossary refinement.** Current "abstention: trained model behavior..." is narrower than Ch5's plumbing half (first-class schema arm gives abstention without training). Amend to "plumbed affordance + trained threshold" to match Ch5.

## Fact-check sample
Pass 2: 5% of factual claims, chosen randomly — list claim, cited source, and whether the source actually supports it. Pass 3: fresh 3% weighted toward revised sections. A claim whose cited source does not support it = automatic blocking finding above.
Per scoping: [LAB: RESULTS-MATRIX §... / PROJECT-LOG ...] are internal private records, honesty-verified, not fetchable public URLs. Sample below checks whether each ARGUMENT actually follows from what the cited section claims to have measured, not external resolvability. No fetch was attempted per scoping rule.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "on a 128 GB VRAM box, Qwen3.6-27B dense Q8_0 is 29 GB / 79.0 MMLU / ~27 tok/s; gpt-oss-120b MXFP4 is ~60 GB / 71.0 MMLU / 60 tok/s; DeepSeek-V4-Flash Q8-MTP master is 149 GB / 88.3 MMLU / 26–27 tok/s. Fit is a recipe, not a parameter count: blobfish Q4 175 GB loads only with --no-repack and mmap, and OOMs or segfaults without those flags" | ch01:Three curves that crossed | [LAB: RESULTS-MATRIX §C/§F] | yes — contrast of memory vs params and flag-dependent load directly supports conclusion that quantization recipe and engine flags, not parameter count alone, decide fit and speed |
| "a from-scratch 32k tokenizer beat 100k/151k vocabularies on industrial text while freeing 487M parameters for layers at fixed budget" | ch02:Tokens / ch03:Tokenizer is part of size budget | [LAB: PROJECT-LOG 2026-08-03 + matrix §O.1 — 32k industrial tokenizer: +3.5–7.9% chars/token, embedding 134M vs 621M params] | yes — measured chars/token gain and embedding param delta entail the freed capacity argument; supports claim that tokenizer choice behaves like parameter budget |
| "15-scenario tool suite, ±10 pts across three identical runs at temperature 0" | ch01:How this book measures / ch02:Generation / ch06:Noise first | [LAB: RESULTS-MATRIX §C footnote — temp-0 flips traced to PAR=2 batch-packing nondeterminism] | yes — repeated citation consistently used to justify rule to publish ranges, re-run surprising numbers, and set thresholds outside noise floor; argument follows |
| "IEB-Signals v1.3 private holdback, n=3,725, deterministic enum decode: channel-level acc 87.61% / AUROC 0.938 / ECE 0.012; scene-level acc 47.89% / AUROC 0.548. Bit-exact across two process launches on 640 rows" | ch02:Constrained output / ch04:Output side | [LAB: RESULTS-MATRIX R.158] | partly — channel-level results strongly support "enum decode locks format and enables reliable confidence" (AUROC 0.938, ECE 0.012, 0 flips); scene-level 0.548 near-random weakens unqualified "small models can hold own on floor" but manuscript qualifies it as task-beyond-model — overall argument coherent when qualified |
| "production burn-in on promoted Q3-MTP: 12-min soak 86 requests, 0 errors, 100% mean acceptance, +2 MiB VRAM drift" | ch06:Cadence / ch08:Rollout shadow | [LAB: RESULTS-MATRIX §G] | yes — supports narrow claim that soak is short-duration stability check, not months-long drift study, as prose states; does not support broader drift claims (prose correctly labels limitation) |
| "power-loss recoveries #1 and #2 ... First outage ~90 min, second ~25 min ... crash-#1 fixes held; zero filesystem work" / "dependency pull vs disablement; condition-gated unit fix" | ch09:What a power loss actually breaks | [LAB: PROJECT-LOG 2026-08-22/24/25] | yes — dated log entries of fstab repairs, condition-gated unit (marker-file interlock), and checkpoint cadence bounding loss <4.5h directly entail the chapter's thesis that boring fixes (fstab, marker file, report) are the delta |

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 5

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| — | N/A — Pass 2 panel | Ledger to be filled at Pass 3 verification |
