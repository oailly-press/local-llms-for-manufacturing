# Chapter 6 — The Quality Gate

*(draft v1, 2026-08-28 — written by Claude Fable 5, unverified. Lab citations attached where the record exists; remaining claims are labeled as unmeasured.)*

Every chapter so far has ended by deferring to this one. Chapter 3's sizing rule said
"climb only on measured failure." Chapter 5's abstention metrics needed somewhere to
live. This is the chapter where measurement stops being a virtue and becomes a machine:
the evaluation gate — the apparatus that decides, with error bars, whether any model,
prompt, quantization, or pipeline change is allowed near your floor.

The gate's job description is one sentence: **it would rather reject a good change than
pass a bad one.** Everything in this chapter is engineering toward that asymmetry,
because the floor's costs are asymmetric in exactly that direction. A rejected good
change costs you a week of investigation. A passed bad change costs you a wrong verdict
in production, discovered by the person acting on it.

> **From the floor.** The most dangerous result on a plant floor is not a failure — it is a
> pass that is too clean. I have watched test equipment hand back a perfect result because a
> person *made* it hand back a perfect result: bypassed the fixture, forced the reading, moved
> the probe off the fault. Not out of malice, usually — a bad number that day was simply more
> expensive to them than an honest one. You do not catch that by trusting the number. You catch
> it by keeping enough of the surrounding data to ask whether the number could even be true:
> was the cycle time physically possible, did the other signals move the way a real pass moves,
> is this station suddenly, suspiciously flawless? A gate that reads only the verdict can be
> gamed by anyone whose incentive points at *clean* instead of *correct.* Build the gate — and
> train the model — to read the evidence, not the conclusion. This is the same discipline the
> rest of this chapter applies to the models themselves: a score that looks too good is a
> finding, not a victory.

## Why vendor numbers cannot be your gate

Start with the uncomfortable fact that makes this chapter necessary: the published
benchmark numbers that models arrive wrapped in — general knowledge scores, reasoning
suites, leaderboard ranks — are nearly useless for your decision. Not because they are
dishonest, but because they measure the wrong distribution. Your floor's questions are
your fault tables, your historian renderings, your technicians' shorthand, your
schemas. A model's rank on graduate-level reasoning problems tells you approximately
nothing about whether it correctly reads your VFD fault table, and the correlation
between general rank and your-task performance is weak enough at the small end of the
ladder that ranking by it is closer to superstition than diligence `[LAB: RESULTS-MATRIX §C — Qwen3.6-35B-A3B 71.0 MMLU / 87 tool hardmode versus DeepSeek Q8-MTP 88.3 MMLU / mean 55 tools; the two scores do not rank the same models]`.

The gate you need is built from your own material. The good news, and a running theme
of this chapter: building it is cheaper than it sounds, and the expensive part — the
labeled cases — is Chapter 4's logging habit already paying out.

## Noise first: the lesson we keep paying for

Before designing anything, absorb the single most important empirical fact about
evaluating language models, because our lab has now paid for it several times: **small
evaluation suites are far noisier than they look, and the noise wears a convincing
costume.**

The canonical incident from our own record: a fifteen-scenario tool-use suite, run
three times against the *identical* model, configuration, and temperature, returned
scores spanning ten points `[LAB: RESULTS-MATRIX §C footnote — ±10 pts across three
identical runs at temperature 0]`. Ten points is the size of a headline improvement.
An engineer who ran the suite once before a change and once after could "measure" a
breakthrough or a catastrophe that consisted entirely of batch-packing nondeterminism.
Nothing about the model had changed. The dice had.

The discipline that follows, written as rules because we follow them as rules:

1. **One surprising number is a hypothesis, not a result.** Re-run it.
2. **Two runs that disagree get a third, and a control** — the unchanged configuration,
   re-benchmarked, to measure the noise floor itself.
3. **Publish ranges, not lucky draws.** A gate that compares single runs is comparing
   noise. Every score that matters is a mean over repeats with its spread attached, and
   the gate's thresholds are set wider than the measured noise floor.
4. **Change one thing at a time.** When two things changed together, bench the
   configuration that isolates each — the control run is what converts "the new build
   is faster and nothing broke" from a hope into a statement `[LAB: RESULTS-MATRIX §E —
   speculation-off control isolating quality from speed]`.

None of this is statistics beyond a maintenance department's comfort; it is the same
repeat-and-control instinct you apply to a suspicious vibration reading. The only
novelty is that the AI industry's demo culture has normalized skipping it.

## Anatomy of a floor gate

A working gate has four layers, each catching what the previous cannot:

**Layer 1 — format checks, free and total.** Does the output parse? Does it conform to
the schema? Are the enum values legal, the required fields present, the quoted evidence
actually present in the context? These checks are deterministic code, they run on every
single response in production as well as in evaluation, and with Chapter 4's
constrained decoding they should pass at essentially 100% — which is precisely why
they stay in the gate: a format regression is the loudest possible alarm that something
structural broke. Pre/post format-check pass rates are not a published pair; keeping format checks in the gate is the practice.

**Layer 2 — labeled-case scoring.** The heart: a few hundred real cases with known-good
answers, scored mechanically thanks to schema'd outputs. Include every case family the
floor actually produces: the routine, the boundary (Chapter 5's near-misses), the
unanswerable (all four abstention quadrants), the adversarial-answerable, and the
formerly-failed — every production mistake that got a human correction enters the gate
set permanently, so no failure has to be discovered twice. Curate for coverage, not
volume: two hundred cases that span the distribution beat two thousand that oversample
the easy middle.

**Layer 3 — behavioral probes.** Targeted mini-suites for properties that case-level
scoring misses: does the model still respect the contract when the context is at 90% of
window capacity? Does performance hold when tag names are swapped for unfamiliar ones
of the same shape — or was the model memorizing your identifiers? Does the abstention
threshold sit where Chapter 5's ladder left it? Each probe is a dozen cases aimed at
one failure mode you have reason to fear.

**Layer 4 — the retention floor.** Chapter 3's specialist trap, enforced: alongside
your task suites, a small general-capability suite with a hard minimum. A tuned model
that aces the plant work and collapses on general instruction-following has been
damaged in ways your task suite cannot see yet; the general floor catches the
amputation early `[LAB: PROJECT-LOG — retention gates: retain ≥90% of base MMLU and IFEval; 0% general replay produced the cliff between 0% and 1%]`.

## The gate that was too strict, and why we kept the story

A gate is code, and code has bugs. The failure mode you must design for is the gate
that is wrong in the *strict* direction — and our lab's most instructive example is the
execution gate that rejected generated code for working correctly: the checker's
sandbox judged legitimate solutions as failures because of an environmental assumption
the checker itself made `[LAB: PROJECT-LOG — the execution gate that rejected correct
code]`. For a while, the measured capability of every model under test was artificially
depressed, and the models were innocent.

Three lessons earned there. First: **gates need controls too.** Feed the gate known-good
answers on a schedule — human-written, verified solutions — and when the gate rejects
one, the gate goes under investigation, not the model. Second: a strict-side bug is
quieter than a lenient-side bug, because nothing bad ships; you simply lose true
capability to a phantom, and only the control run reveals it. Third — and this is why
the story is in a published book — an evaluation program that cannot admit its
instrument was broken will silently convert instrument error into "findings." Our
program's rule is that instrument failures get written up with the same rigor as
results, including the retraction of anything the broken instrument "found"
`[LAB: PROJECT-LOG — Finding 25 retraction: four instrument defects, not a finding]`.
Your plant's version of that rule: the gate's own defect log is part of the gate.

## Contamination: the quiet score inflater

One more instrument hazard, specific to language models: the model may have already
seen your test. Public benchmark questions leak into training corpora; more insidiously
for a floor deployment, *your own* evaluation cases can leak into your fine-tuning sets
through the very logging pipeline Chapter 4 recommended — the case you evaluated in
March becomes training data in May, and June's evaluation is partly a memory test.

The defenses are procedural, not clever. Keep the gate set physically separate from the
training corpus with a checked, one-way boundary — our lab treats train/eval
contamination checking as a standing pipeline step, not a one-time audit `[LAB: PROJECT-LOG — dataset ledger snapshots with SHA-256 + contamination checks are a standing qualification step; semantic/manual contamination remains an open residual]`. Rotate: retire a slice of the gate set to training
periodically and replace it from fresh production traffic, so the gate ages with the
plant instead of fossilizing. And run the memorization probe from Layer 3 — same case
shapes, fresh identifiers — whose divergence from the named-case scores is your
contamination gauge.

## The judge problem

Schema'd outputs made scoring mechanical, and you should fight to keep it that way. But
some floor tasks are irreducibly prose — the shift summary, the incident narrative, the
explanation field beside the verdict — and prose needs judgment to grade. The industry's
answer is to use another language model as the judge, and it works well enough to use
and badly enough to instrument.

Use it with its failure modes named. Judge models prefer longer answers, prefer fluent
answers, prefer answers that share their own phrasing habits, and drift toward
leniency when the rubric is vague — each a bias that will flatter exactly the failure
modes you built this gate to catch. The countermeasures mirror everything else in this
chapter: give the judge a rubric with binary checks rather than a 1–10 feeling
("does the summary mention the 06:41 trip: yes/no"); never let a judge model grade its
own family's outputs — independence matters for graders exactly as it does for
critics; and control the judge itself with planted cases — a known-excellent summary
and a known-flawed one salted into every batch, so a judge that fails the plants gets
investigated before its grades count. Judge-control agreement rates are not a published lab table; salting known-flawed items is the practice. Where a
prose task matters enough to gate a deployment, the final word stays with periodic
human scoring of a sample; the judge model's job is coverage between those samples,
not authority over them.

## Building the first gate in one week

The apparatus above sounds like a quarter's project. It is a week, if you spend the
week on the right things — and the week pays for itself the first time it blocks a bad
change.

**Day one:** pick the single highest-value recurring question on your floor and freeze
its schema. **Days two and three:** harvest cases — from Chapter 4's logs if you have
them, from two afternoons with a maintenance lead and a stack of real work orders if
you do not. Label fifty routine, twenty boundary, twenty unanswerable (spread across
Chapter 5's grades), ten adversarial-answerable. A hundred labeled cases is a real
gate; do not wait for five hundred. **Day four:** write the runner — a loop that
renders, asks, parses, and scores; with schemas it is an afternoon of code. Run it
five times against your current configuration and write down the spread: that number,
the noise floor, is the week's most valuable output, because every future comparison
is meaningless without it. **Day five:** set thresholds outside the noise floor, wire
the runner to your deployment checklist, and file the first report as the baseline.

From then on the gate grows by accretion: every production correction becomes a case,
every incident becomes a probe, every quarter retires stale cases to training and
pulls fresh ones from traffic. Our own benchmark program began as approximately this
week and grew into a named standard by exactly this accretion `[LAB: PROJECT-LOG 2026-08-02 — founder named the benchmark IEB (Industrial Edge Bench); collision sweep clean against IndustryBench/AssetOpsBench/FieldWorkArena; G1 audit: 879 items re-derived, 12 classes independently audited]`
— the gate you start crude this month beats the perfect gate you start next year, by
the width of a year.

## What the gate cannot see

Honesty about the instrument's edges, because a gate that claims totality teaches
people to stop thinking. The gate measures the model-and-pipeline's answers against a
frozen set of cases. It does not measure whether the technicians trust the tool,
whether the screen presents abstentions as next actions or dead ends, whether the
latency fits the rhythm of a shift, or whether the questions people actually ask are
drifting away from the questions the gate contains. Those live in usage metrics and
in Chapter 5's disposition tracking — accepted, corrected, ignored — which is the
gate's necessary complement: the gate says *the system answers correctly*; the
disposition stream says *the system is being used, and how*. A deployment green on the
first and red on the second is not a success with an adoption problem. It is a system
answering questions nobody is asking, and only the pair of instruments together can
tell you so.

## Cadence: when the gate runs

A gate that runs only when someone remembers is a gate that runs after the incident.
Wire it to triggers instead: every model version change, every quantization change,
every prompt-contract edit, every schema version, every tokenizer or renderer change —
the full list of things Chapter 4 taught you are part of the instrument. Plus one more
trigger the AI industry keeps relearning: **the calendar.** Drift is real even when
nothing "changed" — a vendor's silent update (Chapter 1's version-pinning argument), a
new product line's vocabulary entering the traffic, a season's different failure
distribution. A monthly full-gate run against production configuration, filed with its
ranges, is the deployment's routine bloodwork; the trend across months is as
informative as any single result `[LAB: RESULTS-MATRIX §G — 12-min soak on promoted Q3-MTP: 86 requests, 0 errors, 100% mean acceptance, +2 MiB VRAM drift. That is a soak, not a months-long drift study]`.

Keep every gate report. The chronological file of them is the deployment's medical
history: what was tried, what the noise floor was, what regressed and when, which
gate defects were found and fixed. When the auditor, the insurer, or the new plant
manager asks "how do you know this thing works?" — the answer is not a slide. It is
that file.

## The number the business actually needs

One translation duty falls on whoever owns the gate, because the gate's native outputs
— accuracy ranges, abstention quadrants, noise floors — are not the language the plant
runs on. The business runs on error budgets, and the gate is precisely the instrument
that lets you write one.

The translation reads like this: at the gated configuration, the verdict pipeline
produces at most N confident-wrong answers per thousand cases (upper bound of the
measured range, not the mean — the gate's pessimism is the budget's honesty), each
reaching a human reviewer before any action; the expected cost of a reviewed wrong
verdict is one technician-interruption; therefore the system's worst-case error cost
per month is a number, with a measurement behind it, revisited at every monthly gate
run. Set beside it the value column the disposition stream provides — questions
answered, lookups avoided, documentation debts surfaced — and the deployment stops
being a bet on a technology and becomes a line item with a maintenance schedule.
That sentence structure, more than any model capability, is what carries a pilot
through its first budget review — and every term in it came off this chapter's
instrument, which is the quietest argument for building the instrument well.

## The gate in one page

Build the set from your own logged, corrected traffic; span routine, boundary,
unanswerable, adversarial, and every past failure. Score mechanically through schemas.
Run repeats; publish ranges; set thresholds outside the noise floor. Keep a general-
capability floor beside the task scores. Feed the gate known-good controls and
investigate the gate when it rejects one. Guard the train/eval boundary and probe for
memorization. Trigger on every change and on the calendar. File everything.

None of it is exotic, and that is the point this book has been circling since Chapter
1: the difference between a plant that can trust its model and one that cannot is not
the model. It is the instrument the plant built around it — and unlike the model, the
instrument is entirely within your control.

And when the gate blocks something you wanted to ship — it will, and it should — write
the rejection down with the same care as a pass. The file of near-misses is the
instrument's proof that it earns its keep.
