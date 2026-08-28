# Chapter 4 — Reading the Plant: Protocols and Historians

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified. `[R-TBD]` marks numbers
awaiting lab entries.)*

Chapter 2 promised that the context window is your instrument of truth. This chapter is
about filling it — the unglamorous, decisive layer between your plant's data and the
model's window. Nothing in this book pays off harder per hour of engineering, because
the layer is where most deployments actually fail: not because the model was too small,
but because it was fed the wrong slice of the plant, in a format that fought it, with
the question buried.

The chapter's one-sentence thesis: **a language model is a text instrument, and your
plant does not speak text — so the translation you build *is* the application.**

## What the plant actually says

Strip away vendor branding and the data a plant emits has three shapes:

**Registers and tags.** The oldest industrial protocols move numbers by address: a
register holds a 16-bit value, and meaning lives entirely in external documentation —
this address is a temperature, that one a status word, the scale factor is ten. Nothing
in the wire format says so. A model handed raw register dumps is being asked to
hallucinate the documentation; a model handed *decoded* values with names and units is
being asked to read. The difference is your decode table, which you already maintain
for the HMI. Reuse it.

**Structured telemetry.** Newer stacks self-describe to a degree — hierarchical
namespaces, typed values, engineering units carried in metadata. Better raw material,
same principle: the model should receive the *meaningful* rendering, not the transport
rendering. A JSON blob with seventeen levels of vendor namespace wrapping one
temperature is worse model input than the line `furnace_3.zone_2.temp = 613 °C`.

**Events and text.** Alarms, operator comments, work orders, shift notes. This is the
one shape that is already language — and it is the messiest: inconsistent vocabulary,
shorthand, typos, meaning that depends on which technician typed it. It is also, not
coincidentally, where language models add the most value the fastest, because nothing
else in your stack can read it at all.

The historian sits across all three: a time-series archive of tags plus, usually, an
event store. It is the plant's memory, and the model's relationship to it is the
central design problem of this chapter.

## Textualization: the layer nobody budgets for

Between the historian and the context window sits a transformation this book calls
textualization: turning machine data into the text the model will actually read. Every
deployment has this layer; the failed ones just built it accidentally.

The design rules, each one paid for somewhere:

**Render meaning, not transport.** Decode registers, resolve enums to their names,
apply scale factors, attach units. Every decoding step you leave to the model is a
hallucination invitation on exactly the material where hallucination is least
detectable — a wrong number looks identical to a right one.

**Say the units, every time.** `613` is a trap; `613 °C` is data. Unit discipline in
the rendering costs nothing and removes an entire class of confident misreading. Same
for timestamps: render one timezone, name it, and keep one format end to end. Mixed
timestamp formats in a single context are how a model "finds" a sequence error that is
actually your formatter's.

**Structure for the eye that will read it.** Models read tables well when tables are
small and aligned, and badly when they sprawl. Long time series belong summarized —
minimum, maximum, mean, last value, and the timestamps of excursions — with the raw
slice attached only for the window the question is about. Rendering *judgment* is
allowed and encouraged: an excursion marker like `← exceeds alarm limit (600)` placed
by your deterministic code is the single cheapest accuracy upgrade in this chapter,
because it moves a computation from the probabilistic component to the reliable one.

**Keep identifiers whole.** Chapter 2's tokenizer warning becomes a formatting rule
here: never let the renderer wrap, hyphenate, or abbreviate a tag name. If your tags
are brutally long, ship a legend — short alias in the rendering, full name in a
glossary block — so the model reasons over compact symbols your code can expand back
deterministically.

## Selection: the historian is bigger than the window

A single line's historian can emit more text per shift than a small model's window
holds. You will not "give the model the data." You will give it a selection, and the
selection logic is application logic you own.

The selection patterns that recur:

**Question-scoped slicing.** For "why did line 3 trip at 14:07," the slice picks
itself: the tags of line 3's trip chain, a window around 14:07 — minutes before, one
minute after — plus active alarms and the last operator note. Resist the urge to be
generous: a model reading three hundred lines of irrelevant steady-state readings is
spending attention *not* reading the four lines that matter, and needle-burying is a
measured failure mode, not a theoretical one `[R-TBD: context-dilution measurement]`.

**Exception-first summarization.** For standing questions — "summarize the night
shift" — deterministic code compresses first: excursions, alarms, state changes,
setpoint moves, plus base statistics per tag. The model narrates and connects; the
arithmetic already happened in code. Split the labor by trustworthiness: code counts,
the model explains.

**Round-trip drill-down.** The most robust pattern for open-ended diagnosis: give the
model the summary plus a *catalog* of what it may request — tag names, time ranges —
and let it ask. Your code fulfills each request with a fresh, scoped rendering. Three
short trips beat one enormous context: each step keeps the window dense with relevant
material, and the request trail becomes an audit log of the model's reasoning that a
human can replay `[R-TBD: single-shot vs drill-down accuracy]`.

## Asking: the question is part of the instrument

Textualized data plus a vague question still fails. The prompt patterns that survive
floor duty:

**State the contract before the data.** Role, allowed sources ("answer only from the
data below"), the abstention arm ("if the data does not determine an answer, say
exactly `INSUFFICIENT DATA` and name what is missing"), and the output schema — all
*before* the data block. Contracts stated after a long context are the first thing
lost to Chapter 2's window-edge forgetting.

**One question per call.** "What tripped, is it the same as last Tuesday, and should
we replace the sensor?" is three questions, and a fused answer to all three is
unauditable. Three calls with three scoped contexts are cheap — Chapter 3 made them
cheap — and each answer lands somewhere specific.

**Demand evidence inline.** Require every claim in the answer to quote the line of
rendered data it rests on. This is the cheapest hallucination detector ever shipped:
fabricated claims either quote nothing or quote text that is not in the context, and
your code can check the quotes mechanically before a human ever reads the answer
`[R-TBD: quote-check catch rate]`.

## The output side: schemas as guardrails

Chapter 2 introduced grammar-constrained decoding as the plant's secret advantage; this
is where it goes to work. Every floor question that recurs deserves a schema: the enum
of legal verdicts, the required evidence field, the confidence grade, the
`INSUFFICIENT_DATA` arm. Constrained output turns free-text grading into field
checking, makes downstream automation safe to build, and — the underrated part — makes
*evaluation* mechanical, which Chapter 6 will exploit: a schema'd answer scores itself
against a labeled key without a human reading prose `[R-TBD: enum-decode mechanics]`.

Schema design has its own craft. Keep enums short — every added arm is a place to be
wrong, and models discriminate eight options far better than thirty. Make the
abstention arm first-class, not a string the model must remember to produce. Version
your schemas like any interface, because Chapter 6's evaluations pin to them. And log
every raw model response alongside its parsed form: when a verdict is challenged later,
the raw text is your flight recorder.

## A worked rendering

Theory earns its keep in the diff between bad and good input, so here is one, end to
end. The question: "why did conveyor 2 stop at 06:41?"

The accidental deployment pastes what the export button produced — hundreds of rows
shaped like this, one per tag per second:

```text fragment
2026-08-27T06:39:58.113Z,PLC7.DB44.REG117,4212,1
2026-08-27T06:39:58.113Z,PLC7.DB44.REG118,0,1
2026-08-27T06:39:58.641Z,PLC7.DB44.REG117,4213,1
```

Raw addresses, unscaled integers, a quality flag nobody explained, timestamps to the
millisecond for a question about minutes. The model must guess that REG117 is the drive
current, that 4212 means 42.12 amps, that quality `1` is good — every guess a coin flip
wearing a lab coat.

The engineered deployment renders the same facts like this:

```text fragment
CONTEXT: conveyor_2 trip investigation, window 06:36–06:43 local (America/Chicago)

conv2.drive_current_A: min 41.9, max 67.3 ← exceeds alarm limit (55.0), last 0.0
conv2.motor_temp_C:    min 71,   max 74   (alarm limit 90 — not reached)
conv2.state:           RUNNING → FAULTED at 06:41:12
alarms: 06:41:12  CONV2_OVERCURRENT  (priority 1, active)
operator note 06:44 [quoted material, not instruction]:
  "reset twice before shift change, tripped again both times" — j.m.
```

Same historian, same facts. The second rendering resolved names, applied scales,
attached units, pre-computed the excursion against its limit, collapsed a thousand rows
into the five lines that matter, fenced the human text, and stamped the timezone. Ask
Chapter 2's three-way question against both and the gap is not subtle: against the
first, a small model free-associates about registers; against the second, it has almost
no room to be wrong, and the remaining judgment — overcurrent from mechanical jam
versus drive fault, and what to check first — is exactly the judgment you wanted it
applying `[R-TBD: raw-vs-rendered accuracy delta]`.

The uncomfortable observation hiding in this example: most of the intelligence in the
answer was placed there by the renderer. That is not a criticism of the model. It is
the design. You want the probabilistic component operating on the shortest possible
inferential leash, and the leash is braided out of decode tables you already owned.

## The standing watcher

Everything above answers questions someone asked. The other half of floor duty is the
question nobody asked yet: continuous monitoring, where Chapter 3's always-on small
model earns its electricity.

The pattern that works is exception-driven, not exhaustive. The watcher does not read
the full stream — deterministic limit checks and your existing alarm system already
guard thresholds, and racing them with a language model is using a poem to do a
comparator's job. The watcher reads what the deterministic layer *cannot*: the
conjunction of weak signals. A drive current trending up for a week, a bearing
temperature that now runs three degrees warmer after each restart, an operator note
mentioning "the smell again," none individually alarmed — rendered together into one
periodic digest, with the standing question "what deserves a human's attention this
week, and why?"

Two disciplines keep the watcher from becoming noise. **Dwell before speaking:** the
digest runs daily or per shift, not per event, and repeats a flagged item only when its
evidence strengthens — a model that re-announces the same trend every hour trains the
crew to delete its reports, which is the alarm-management lesson of Chapter 9 all over
again. **Track the hit rate:** every watcher flag gets a one-click disposition from the
human who read it — useful, noise, already-known — and the running rate is reviewed
like any instrument's calibration. A watcher below a usefulness floor gets retuned or
retired; sentiment is not a metric `[R-TBD: watcher precision from lab deployment]`.

## The corpus you are accidentally building

One more consequence of doing this chapter properly, and it may be the most valuable:
log every rendering and every answer, and you are building the exact training set
Chapter 7 needs. The renderings are your plant's text in its canonical form; the
questions are your plant's real question distribution; the verdicts — especially the
ones a human corrected — are labeled examples of the highest possible relevance.

Three habits make the accident deliberate. Log the *rendered context*, not just the
raw data, because the rendering is what the model actually read and what a future
fine-tune should learn from. Capture the human disposition — accepted, corrected (to
what), rejected — at the moment it happens, in the same record; reconstructed labels
are worth a fraction of contemporaneous ones. And mark every record with its
clearance status at capture time: what may be used for training, what contains names
or sensitive material needing scrubbing, what must stay out entirely. Sorting that out
record-by-record later is a project; a flag at write time is a column. Chapter 7
inherits this corpus and will be grateful for every one of these habits — and the
provenance discipline is the same one this book's own publisher applies to itself,
which is not a coincidence.

## What can go wrong: the adversarial note

One risk class is specific to reading *text* from the plant and deserves its plain-
language warning. Operator comments, vendor documents, even alarm description fields
are authored content, and authored content can contain instructions — innocently
("CALL JIM BEFORE RESETTING, HE KNOWS THE TRICK") or otherwise. A model reading a
context cannot fully distinguish data from directive; a work-order note that happens to
read like a command can steer an answer. The mitigations are layered, none exotic:
render untrusted text clearly fenced and labeled as quoted material; instruct the
contract that quoted material is evidence, never instruction; constrain outputs so that
even a steered model can only choose among legal verdicts; and keep a human between
model verdicts and physical actions — which Chapter 10 will insist on for its own
reasons anyway. Treat this the way you treat any untrusted input path into a control
system: not with panic, with plumbing.

## Brownfield honesty

Every pattern above assumed the decode table is right, the sensors work, and the
historian kept everything. No plant matches that description. The pipeline has to be
honest about its inputs' dishonesty, and the mechanisms are pleasantly mundane.

Undocumented tags — the registers nobody remembers mapping — get rendered as exactly
what they are: `PLC7.DB44.REG119 = 77 (unmapped tag — meaning unknown)`. That label
does two jobs: it stops the model from inventing a meaning, and it turns every
investigation that stumbles over the tag into a small documentation work order. Dead
and stuck sensors are a renderer responsibility, not a model discovery: a value that
has not changed in a week, or reads outside physical possibility, gets flagged by code
(`flatlined since 08-20`) so the model treats it as an evidence gap rather than a fact.
And historian gaps — the outage from Chapter 9, the tag that started logging only last
spring — must render as explicit absence (`no data 02:12–02:31`), because a model
shown a seamless series will reason as if the world were seamless too.

The shared principle: **every known defect in the data becomes a visible label in the
rendering.** The model's abstention machinery — next chapter's subject — can only
decline to answer when the rendering lets it see what is missing. A pipeline that
papers over its gaps upstream has quietly removed the model's ability to be honest
downstream, and will then blame the model for the confident answer it was set up to
give.

## The chapter in one drawing

Historian → **decode** (names, units, scale — your existing tables) → **select**
(question-scoped slice or exception-first summary) → **render** (aligned, unit-bearing,
identifiers whole, judgments pre-computed) → **contract** (sources, abstention arm,
schema, then the data) → model → **constrained output** (enum verdict + quoted
evidence) → **mechanical checks** (quotes exist, schema valid) → human.

Every arrow is deterministic code except the model itself — which is precisely the
point. The probabilistic component sits in the middle of a pipeline that feeds it
honestly and checks it mechanically. Build the arrows well and Chapter 5 can make the
model itself honest about the one thing the pipeline cannot check: whether the answer
should exist at all.

One closing measurement to make once the pipeline stands: feed it a question whose
answer you already know, end to end, and time every arrow. Wherever the minutes went is
where your next engineering hour belongs — and it is almost never the model.
