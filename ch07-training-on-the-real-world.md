# Chapter 7 — Training on the Real World

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified. `[R-TBD]` marks numbers
awaiting lab entries.)*

Every chapter until now has treated the model as a purchased part: pick a size, wrap it
in plumbing, gate it. For many floors that is the whole story, and a good one. This
chapter is for the moment the gate tells you the purchased part has a ceiling — when
grounding is right, schemas are tight, the tokenizer question has been asked, and the
boundary cases still fail because the model has simply never lived in your world. The
fix is training, and training runs on the one asset nobody can buy: your data.

Read this chapter with Chapter 6 already installed, because its first commandment
governs everything here: **no training run without a gate to measure it.** Training
without an evaluation is spending money to change a system in an unknown direction.

## The ladder of intervention (climb it in order)

Training is the *last* rung of a ladder, and each lower rung is cheaper, faster, and
more reversible. The gate decides when to climb; impatience is not a reason.

**Rung 0 — better plumbing.** Chapter 4's renderer and Chapter 5's contracts fix most
of what gets blamed on the model. Our own failure-attribution habit exists because the
majority of "the model is too weak" complaints dissolve under inspection into rendering,
selection, or contract defects `[R-TBD: failure attribution tally]`.

**Rung 1 — examples in the prompt.** Before changing weights, show the model two or
three worked cases inside the contract. For format and tone this is often training's
equal at zero cost; its limit is capacity — examples spend context window, and their
influence fades on genuinely unfamiliar material.

**Rung 2 — supervised fine-tuning (SFT).** Continue training the purchased model on
your labeled pairs — rendered context in, correct schema'd verdict out. This is the
workhorse rung, the one this chapter mostly details, and on small models it is
genuinely accessible: hours on a workstation GPU, not weeks on a cluster `[R-TBD:
fine-tune wall-clock by tier]`.

**Rung 3 — continued pretraining.** Feed the model raw domain text — manuals,
procedures, standards — before any task tuning, so the vocabulary and idiom of your
world stop being foreign. Worth it when the domain gap is wide (the model has plainly
never read your industry) and you hold enough text to matter.

**Rung 4 — from scratch.** New tokenizer, new weights, your corpus at the center. The
rung where Chapter 3's tokenizer-as-budget finding came from `[LAB: PROJECT-LOG
2026-08-03 — from-scratch 32k industrial tokenizer]`. For a single plant this rung is
rarely rational; it exists for platform builders, and this book's own lab lives there
so that plants do not have to.

## The corpus is the product

Whatever rung you climb to, the work is nine parts data to one part training command.
The corpus disciplines, in the order they save you:

**Capture at the source, with consent flags attached.** Chapter 4's logging habit —
rendered context, model output, human disposition, clearance status, all in one record
— is the corpus assembling itself. The clearance flag at write time is the discipline
that keeps the lawyers calm later: what may train, what needs scrubbing, what never
leaves the historian's shadow. Our lab's standing rule marks every captured stream at
ingestion, because sorting a million records retroactively is a project with no
champion `[R-TBD: capture-hygiene protocol]`.

**Real beats synthetic; synthetic fills the gaps real cannot.** Your logged traffic is
the gold standard — it is, by construction, the exact distribution the model will face.
Its weakness is coverage: the faults that happen rarely, the abstention cases (Chapter
5's ladder), the adversarial-answerables. There, generated data earns its place: a
larger teacher model, given your real documents, can author boundary cases at volume —
distillation, in the field's vocabulary, and the engine of most small-model quality
today. Two cautions from our own distillation program. First, teacher outputs inherit
teacher errors, so generated cases pass through the same human-spot-check and gate
machinery as real ones — synthetic data is an ingredient, never a bypass `[R-TBD:
teacher-error rates in distilled sets]`. Second, license and terms: know what your
teacher's terms permit trained artifacts to do commercially, and record the answer in
the corpus manifest, because retrofitting provenance onto a trained model is
impossible. Write it down at generation time or lose it forever.

**Deduplicate and decontaminate, mechanically.** Two hygiene passes run before any
token reaches training. Near-duplicate removal, because repeated text teaches
repetition in exactly the way Chapter 2's compression story predicts. And gate-set
exclusion — the contamination boundary from Chapter 6 — enforced by tooling, not by
promise: our pipeline checks materialized training shards against evaluation sets as a
standing step, and the check has caught real leaks that manual diligence had already
signed off on `[R-TBD: contamination catches]`.

**Balance what you feed.** A corpus assembled from convenience oversamples the routine.
Weight by what the gate says the model gets wrong, not by what the logs happen to hold:
boundary cases up, greatest-hits down, and — Chapter 5's warning standing — answerable
controls always paired with abstention cases, so the training pressure pushes the
threshold rather than just the refusal reflex.

## Where the data actually comes from: a field inventory

"Your data" sounds like one thing; on a real floor it is six, each with its own effort
profile and its own trap.

**Service manuals and OEM documentation.** The densest value per page and the messiest
acquisition: much of it lives as scanned PDFs, and OCR quality decides everything
downstream. Budget real time for the ugly ones — a fault table whose columns OCR
scrambled is worse than no table, because it trains confident misreadings. Spot-check
extraction against the paper with the same sampling discipline the gate uses; a
manuals corpus is an instrument too.

**Work orders and maintenance history.** The richest task-shaped data you own:
symptom, diagnosis, action, outcome, in your technicians' own language. Also the most
sensitive — names, blame, the occasional colorful assessment of a vendor. The
clearance flag earns its keep here; so does a scrub pass that replaces names with
roles before anything reaches a training shard.

**Historian exports and alarm logs.** Unlimited volume, low per-line value — raw
telemetry teaches a language model surprisingly little. Its training value appears
only after Chapter 4's rendering: excursion summaries paired with the questions they
answer. Train on renderings, never on raw dumps.

**Shift notes and operator logs.** Idiom, shorthand, and the plant's real vocabulary
live here — this is where the tokenizer and the model learn how your people actually
write. Same scrub rules as work orders.

**Standards and regulations.** Public, clean, voluminous, and generic. Useful as
continued-pretraining ballast for domain vocabulary — our own industrial corpus draws
heavily on public regulatory text for exactly this role `[LAB: PROJECT-LOG 2026-08-03
— regulatory corpus across nine agencies]` — but do not mistake it for task data; no
regulation ever answered a work order.

**Vendor bulletins and field advisories.** Small, fresh, high-value: the documents
that correct the manuals. A capture habit for these pays twice — once in training,
once because Chapter 4's retrieval layer wants them anyway.

The inventory's summary line: effort concentrates where value does — manuals and work
orders first, rendered telemetry second, everything else as seasoning.

## The worked tune: the extractor, one year later

Pick up Chapter 3's work-order extractor where the sizing walkthrough left it: the
line-side class passed the gate with margin, shipped, and has now logged a year of
traffic. The gate's monthly runs show the ceiling: symptom-field accuracy plateaued,
and the residual errors cluster in exactly two shapes — new-product vocabulary the
base model never saw, and the technicians' compressed shorthand for intermittent
faults.

The tune that addresses it, by this chapter's numbers: the year yielded a few thousand
disposition-labeled records; roughly a fifth are corrections, the high-value minority.
Augment the thin spots with teacher-generated boundary cases built from the real
manuals; balance so corrections and abstention cases punch above their volume; split;
smoke-test on fifty; run the tune on the workstation GPU overnight. The after-gate
tells the story in ranges: symptom accuracy up meaningfully, the two error clusters
visibly compressed, abstention quadrants unmoved, retention floor intact `[R-TBD:
extractor tune before/after]`. Total cost: one engineer-week, mostly on data, exactly
as the nine-to-one ratio promised. The model file that results gets a version, a
manifest, and a gate report stapled to it — and the plant now owns a small model that
no vendor could sell them, because no vendor has their year of corrections.

## What you must be able to reproduce

A tune that cannot be reproduced is a lottery ticket that happened to win. The
artifact list that makes it engineering — versioned together, referenced by the model's
release record:

the **dataset manifest** (which records, which snapshot, which clearance flags, which
scrub pass); the **tokenizer** identity; the **base model** checksum; the **training
configuration** (every hyperparameter, including the seed); the **checkpoint lineage**
(which step shipped and why — the validation curve that chose it); and the **gate
reports**, before and after, with their noise floors. Six files, none large, and
together they answer the question every auditor and every future engineer will ask in
the same words: *what exactly is this model, and how would we make it again?* Our lab
treats a run missing any of the six as unshippable regardless of its scores — the
provenance page this book's own publisher demands of authors is the same discipline
pointed at weights `[R-TBD: run-manifest standard]`.

## Whose knowledge is this?

One more discipline, and it is not technical. A fine-tuned floor model is largely a
compression of your technicians' accumulated judgment — the corrections they typed,
the shorthand they invented, the diagnostic instincts their work orders encode. Treat
that fact with the respect it is owed. The labeling afternoons go better when the
people whose knowledge is being captured know what it is for and see the result: the
extractor that stops mangling their shorthand is *their* improvement, and saying so
costs nothing. The disposition buttons must never become a surveillance instrument —
the moment corrections feed performance reviews, the corrections stop, and with them
the corpus. And when the tuned model works, the plant has not replaced its
technicians' knowledge; it has given it a backup copy and a faster index. Said
plainly and meant, that sentence is the difference between a crew that feeds the
system and one that starves it — and the corpus, like every instrument in this book,
runs on what the crew decides to give it.

## Running the tune without fooling yourself

The mechanics of a floor-scale SFT run are almost anticlimactic — a config file, a
labeled dataset, hours of GPU time. The self-deception opportunities are where the
engineering lives:

**Split before you start.** Train, validation, and the gate's held-out set, separated
before the first step and never merged. The validation curve tells you when to stop;
the gate — untouched during training — tells you what you actually built.

**Overfit on purpose once.** A tiny sanity run on fifty examples should reach
near-perfect training scores quickly; if it cannot, the pipeline is broken somewhere
between data and loss, and no full run should start until the small one behaves.
Cheap, boring, and it has saved us real GPU-days `[R-TBD: pipeline smoke protocol]`.

**Checkpoint on the cadence Chapter 9 taught,** because training runs are exactly the
long-running state the power-loss chapter was about — ours resumed mid-run through two
building-wide outages on the strength of nothing but cadence and tested restores
`[LAB: PROJECT-LOG 2026-08-22/24 — training resumed at step through both crashes]`.

**Gate before and after, same suite, same repeats.** The before-run is the baseline
and the noise floor; the after-run is the claim. The difference, expressed as ranges,
is the entire truth of what the tune accomplished. Anything narrated beyond that —
"it feels sharper" — is Chapter 6's demo culture sneaking back in through the side
door.

**Check the retention floor last and always.** The specialist trap is sprung by
exactly this chapter's activity. A tune that lifts the plant suite and dents the
general floor has traded connective tissue for memorized competence; Chapter 3's rule
holds — that trade fails the gate no matter how good the domain delta looks
`[R-TBD: retention gate]`.

## The escalation teacher

Chapter 3's two-model pattern was introduced as an operations idea: the small always-on
model escalates its abstentions to a larger one. Notice what that architecture quietly
produces on the training side: a perfectly targeted teacher, running for free on
exactly the cases the small model cannot handle.

Every escalated case arrives pre-labeled as a small-model gap — that is *why* it
escalated — and departs with the larger model's answer attached, plus, for the cases a
human then reviewed, a disposition on that answer. Fold the reviewed set back into the
next tune and the loop closes: the small model's weakest distribution becomes its next
training set, authored by its own escalation partner, at the rate the floor actually
generates hard cases. This is distillation with the sampling problem solved — no need
to guess which boundary cases to synthesize when the deployment is harvesting the real
ones nightly `[R-TBD: escalation-loop gains per cycle]`.

The loop needs two governors or it eats itself. Only *human-dispositioned* escalations
train — the big model's unreviewed answers are teacher outputs like any others, and
recycling unchecked teacher errors compounds them with each cycle. And the gate's
held-out set stays outside the loop entirely, per Chapter 6's boundary — a
self-improving system that grades itself on the cases it trained on will report
asymptotic perfection while learning nothing. Governed, the loop is the closest thing
this book offers to a deployment that gets better by being used; ungoverned, it is a
photocopier of mistakes. The difference is two rules and the discipline to keep them.

## When the answer is: do not train

The honest close, because a chapter about training owes you the cases where the right
call is putting the tool down.

Do not train to fix what plumbing can fix — rung zero exists because it wins more
often than pride admits. Do not train on a corpus you would be uncomfortable showing
the auditor, the union, or the vendor whose manual you scanned; the corpus manifest is
a disclosure document, and this book's publisher applies the same rule to itself. Do
not train against a moving target — if the schemas, renderer, or tokenizer are still
changing weekly, tune after they settle, or you will be paying to specialize a model
to a pipeline that no longer exists. And do not train past the gate's ability to
measure: when your labeled cases number in the dozens, every one of them belongs in
evaluation, not training — gather first, tune later.

The thread through all four: training converts data into behavior, permanently and
somewhat opaquely. It is the least reversible thing this book teaches. The
disciplines around it — clearance flags, contamination checks, before-and-after
gates, retention floors — are not bureaucracy around a simple act. They are what
makes an irreversible act safe to take, which is a sentence a plant engineer has
heard before, in front of different machinery, and knew to respect.

And when a tune ships, close the loop the way every chapter here closes it: the run's
six artifacts filed, the gate report attached, a dated entry in the deployment log
saying what changed and why. The next engineer — possibly you, a year from now, at
2 AM — inherits a model with a paper trail instead of a mystery with a version number.
