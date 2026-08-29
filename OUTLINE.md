# Local LLMs for Manufacturing — outline v1 (2026-08-26)

**Series:** O'AILLY Industrial Nº 1 · **Cover:** circuit beetle, copper · **Status:** IN PROGRESS

**Thesis:** The plant floor doesn't need a frontier model in the cloud; it needs a small,
local, honest model that reads its protocols, cites its historian, and says "I don't know"
before it says something wrong.

**Evidence discipline:** every benchmark number in this book must resolve to an R-entry in
`~/ai/computer-scientist/RESULTS-MATRIX.md` or a named external source. In-text `[LAB:]`
markers name the section or dated entry. Claims without a lab marker are labeled
unmeasured in the prose. The floor-voice section in chapter 1 is from the named
verifier's interview; the manuscript was human-verified by Roger AI on 2026-08-28.

## Part I — The gap

1. **The 518-page silence.** The field's standard machine-data text: zero LLM mentions.
   What changed, why the successor chapter doesn't exist, and why local (not cloud) is the
   only shape that survives plant constraints: air-gaps, latency, data ownership, cost.
   Opens with a `[FOUNDER]` war story from real plant floors.
2. **What a language model actually is, for people who own machines.** No hype chapter:
   tokens, context, sampling, hallucination — explained against a CAT equipment manual and
   a live protocol stream, not poetry. Grounded in our real corpus work.
3. **Why small.** The tier ladder (250M → 1.5B → 8B → 35B) and what each tier can hold.
   The tokenizer-as-parameter-budget finding [R-TBD]. What fits on a Pi, an industrial PC,
   a prosumer GPU box — and what honestly doesn't. (Pico ≠ MCU. Never claim otherwise.)

## Part II — The machinery

4. **Reading the plant: protocols and historians.** Structured industrial data as model
   input; enum-decode mechanics [R-TBD]; why grammar-constrained output beats free text on
   the floor.
5. **The abstention chapter (the heart of the book).** Extraction + abstention as the real
   capability gap [R-TBD, wave-nano redundancy finding]. The model that answers everything
   is the one that gets your line stopped. Calibrating "I don't know."
6. **The quality gate.** Decontamination calibrations [R-TBD]; the execution gate that
   rejected code for working correctly [R-TBD] — what over-strict gates teach; eval design
   for industrial tasks (IEB) and why we published the methodology.
7. **Training on the real world.** Building an industrial corpus: capture hygiene
   (corpus_ok), synthetic vs real, what contamination checking actually looks like [R-TBD].

## Part III — The floor

8. **Deployment shapes.** Industrial PC / edge box / GPU server; serving, quantization at
   the level a plant engineer needs (deep dive deferred to Book Nº 2); watchdogs, restarts.
9. **Surviving reality.** Three power crashes, 10-minute-to-hours recoveries, what made
   recovery cheap: checkpoint cadence, fstab discipline, condition-gated services [R-TBD,
   PROJECT-LOG 2026-08-22/24 entries]. Reliability engineering as a first-class chapter.
10. **The honest deployment checklist.** What to measure before trusting a model with a
    machine; error bars, controls, re-measurement; when to keep a human in the loop
    (answer: longer than you want).

## Back matter

- **Provenance page** — models, sources, verifier, C2PA manifest. This book is its own
  first demo.
- **Errata & retractions policy** — modeled on the lab's retraction practice.

## Open dependencies

1. Floor-voice interview: drafted into chapter 1 (2026-08-28), from Roger AI's interview.
2. R-entry attachment pass: done for chapter body markers (2026-08-28). Unmeasured
   claims are labeled in prose rather than given fake numbers.
3. Title: "Local LLMs for Manufacturing" stands.
4. Named-human verification pass: performed by Roger AI on 2026-08-28.
