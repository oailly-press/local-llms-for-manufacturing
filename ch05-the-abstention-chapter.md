# Chapter 5 — The Abstention Chapter

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified. `[R-TBD]` marks numbers
awaiting lab entries.)*

Every chapter so far has been building toward a single sentence, and this is the chapter
that gets to say it plainly: **on a plant floor, the most important thing a language
model can produce is a refusal to answer.**

That sentence sounds backwards everywhere else in the AI industry. Demos are scored on
answering; benchmarks reward the attempt; a chatbot that says "I don't know" three times
in a row feels broken. But you do not run a demo. You run machines that can hurt people
and processes that cost money by the minute, and in that world the failure modes are not
symmetric. A model that says "I don't know, and here is what's missing" costs you a
lookup or an escalation. A model that confidently names the wrong bearing costs you the
teardown of the good one — and worse, it costs you the crew's trust in every answer that
follows, including the correct ones. The entire economics of deploying language models
on a floor turns on the rate of confident wrong answers, and abstention is the machinery
that buys that rate down.

Chapter 2 explained why this machinery does not come built in: generation cannot stay
silent, and "I don't know" is just another sentence shape the model learned — produced
when the context makes it probable, not when the evidence makes it true. This chapter is
about closing that gap from both ends: **plumbing that gives the model somewhere honest
to stand, and training that teaches it to stand there.**

## The grades of no

"The model should abstain" is too coarse to engineer. In practice a floor deployment
needs about six distinguishable refusals, and conflating them wastes most of
abstention's value:

**Evidence-absent.** The rendered context does not contain the answer. "The fault table
excerpt does not cover code F-7288." This is the cleanest grade, the easiest to train,
and — thanks to Chapter 4's brownfield labels — often mechanically checkable after the
fact.

**Evidence-conflicting.** The context contains two answers. The manual says the limit is
90 °C; the alarm configuration says 85. A model that silently picks one is manufacturing
certainty; the correct output names the conflict and stops. Conflicts are gold for a
maintenance organization — each one is a documentation defect located for free — but
only if the model is trained to surface rather than resolve them.

**Under-specified question.** "Is the pump okay?" Which pump, okay for what, over what
window? The correct response is a clarifying question, and it is a *different skill*
from the other grades: the model must produce the missing parameters, not just decline.

**Out-of-competence.** The question is answerable but not by this system: a legal
question, a warranty judgment, an instruction to modify a setpoint. The refusal names
the right channel. This grade is mostly a policy statement wearing a model's voice, and
the policy belongs in the contract prompt where audit can read it.

**Stale-or-unreliable input.** Chapter 4's flatlined sensor, the historian gap, the
unmapped tag. The evidence exists but is flagged untrustworthy; the answer inherits the
flag. "The reading suggests X, but the sensor has been flat since 08-20 — verify at the
gauge."

**Escalate.** The model has an answer and the answer is alarming: evidence points to a
condition that should not wait for the normal workflow. Strictly this is the opposite of
refusing — but it belongs in the same taxonomy because it is the same trained judgment
about *the limits of the current interaction*, and because the plumbing that carries it
is identical: a first-class arm in the output schema.

Design your schemas so each grade is an explicit, selectable arm — Chapter 4's
constrained decoding makes the arms unmissable — and downstream handling can differ:
evidence-absent routes to a document lookup, conflicts file a documentation ticket,
escalations page someone. When all six collapse into one shrug, the organization learns
that the model's "no" means nothing in particular, and stops reading it.

## The plumbing half: give honesty somewhere to stand

Before any training, most of abstention is affordance. A model *cannot* honestly decline
if the pipeline hides what is missing, and it *will not* reliably decline if declining
requires composing an unusual sentence against the gradient of its instincts.

The affordances, most of them already built in earlier chapters:

**A first-class arm.** `INSUFFICIENT_DATA` as a schema value, not a phrase the model
must remember to write. With constrained decoding, abstention becomes one legal token
choice among a handful — the cheapest it can possibly be. Our measurements around
enum-constrained verdicts consistently show format failures vanishing and the remaining
errors becoming *judgment* errors `[R-TBD: enum-decode mechanics]` — which is exactly
the error type training can then address.

**Visible gaps.** Chapter 4's rule — every known data defect becomes a visible label —
is abstention's raw material. A model can only say "no data for the window in question"
if the rendering said `no data 02:12–02:31` instead of splicing the series seamlessly.

**A required "what's missing" field.** Pair every abstention arm with a mandatory
companion: name the evidence that would change the answer. This converts a dead-end
"I don't know" into a work item — pull this manual section, check that gauge — and it
also disciplines the model: fabricating a missing-evidence description is harder than
fabricating an answer, so the field acts as a natural brake on lazy abstention.

**Contract language that pre-authorizes refusal.** The prompt states, before the data:
"If the data below does not determine an answer, select INSUFFICIENT_DATA. That is a
correct and preferred outcome." The sentence matters more than it looks. Models arrive
tuned by their general training toward helpfulness; an explicit authorization measurably
shifts the threshold `[R-TBD: contract-authorization ablation]`, and it costs eleven
words.

## The training half: teaching the threshold

Plumbing gives the model somewhere to stand; training decides *when* it stands there.
The distinction that organizes everything: abstention is not a fact the model learns, it
is a **threshold** it learns — a decision boundary between "the evidence supports an
answer" and "it does not," running through every topic the model will ever touch.

What moves the threshold, in the order you should try:

**Demonstrations with the reasons visible.** Supervised examples where the context
genuinely lacks the answer and the target output is the right grade of no, *with the
tell named*: "the excerpt covers codes F-7000 through F-7199; the asked code is F-7221."
Symmetric examples where the evidence is present and the target answers. The pairing is
the point — a training set of only-refusals teaches refusal as a style, not a judgment.

**Near-miss mining.** The most valuable training examples live at the boundary:
contexts where the answer *almost* appears — the right manual but the wrong revision,
the right tag but the wrong week, a related fault code one digit off. Models fail at
the boundary, not in the obvious cases, and Chapter 4's logging habit produces exactly
these examples from your own traffic, pre-labeled by the humans who corrected them.

**Asymmetric penalties.** Wherever your tuning framework lets you weight errors, weight
them like the floor does: a confident wrong answer is several times worse than a missed
answerable question. The ratio is a policy decision your safety review should own —
this book's lab treats it as a first-class training parameter rather than a default
`[R-TBD: penalty-ratio sweep]`.

**Progressive evidence removal.** Build training sequences from a single case rendered
at several evidence levels — full manual page, partial page, table of contents only,
nothing — with the target flipping from answer to abstention at the level where a
careful human's would. This trains the *slope* of the threshold rather than isolated
points on it, and it doubles as the cleanest evaluation instrument this chapter has
(you will meet it again below as the calibration gym).

A warning from our own program, because it is the predictable failure of doing the
above with enthusiasm: **over-abstention is a real and measured failure mode, not a
hypothetical** `[R-TBD: over-abstention incident]`. A model trained hard on refusals
learns that refusing is safe, and begins declining questions the context plainly
answers — which quietly destroys the deployment's value while looking responsible in
every individual transcript. Abstention training without answerable controls in both
training and evaluation is how you build a very polite paperweight.

## Calibration: the honest middle

Between "answer" and "refuse" lives a third output worth engineering: the answer with
its confidence attached. Not decorative confidence — calibrated confidence, where of
the claims the model tags HIGH, nearly all are right, and of the claims it tags LOW,
you genuinely cannot count on much.

Calibration is measurable with nothing but a labeled evaluation set: bucket the model's
answers by its own stated grade, compute the accuracy within each bucket, and compare
the curve to the diagonal. A small model will not give you a philosopher's calibration,
but it does not need to: the floor needs three honest grades — act on it, verify first,
treat as a hunch — and holding a model to three grades it means is an achievable,
testable engineering target `[R-TBD: calibration curve by tier]`. Wire the grades into
the workflow: HIGH routes to the technician's queue, MEDIUM routes with its evidence
attached for verification, LOW never leaves the review screen. Now calibration is not a
model virtue; it is a routing rule with a measured error rate — which is a sentence a
plant manager can approve.

## Evaluating the skill of no

Chapter 6 builds the general evaluation machinery; abstention needs its specific
instruments stated here, because most published evaluations simply do not measure it.

**Score all four quadrants.** Answerable-and-answered, answerable-but-refused,
unanswerable-and-refused, unanswerable-but-answered. The last quadrant — the confident
fabrication — is the one the floor fears; the second is the paperweight tax. A single
"accuracy" number hides both. Report abstention precision and recall as first-class
metrics beside answer accuracy, and set gates on all of them: our own gate philosophy —
inherited from a benchmark program that would rather fail a good model than pass a
lucky one — is that **a model unable to say "I don't know" fails the industrial gate
regardless of its answer accuracy** `[R-TBD: IEB abstention gates]`.

**Run the gym.** The progressive-evidence-removal sequences from the training section,
held out, give you the threshold's location and sharpness: at what evidence level does
the model flip, and how consistently? A model that flips at different levels for
cosmetically different phrasings of the same case has a soft threshold, and soft
thresholds are where the confident fabrications leak through.

**Adversarial answerables.** Include questions that *look* unanswerable — obscure
phrasing, ugly rendering, a distractor gap label elsewhere in the context — but are
answerable from the given evidence. These catch over-abstention the way the fourth
quadrant catches fabrication, and a gate needs both jaws.

## A ladder, worked

Here is the calibration gym on one rung of real shape, because the pattern is easier to
copy than to describe. Take Chapter 2's drive fault, F-7221, and build five renderings
of the same investigation:

**Level 1 — full evidence.** The fault table row for F-7221, the drive's decel
parameters, the historian slice showing the bus voltage spike. Target output: the
answer, HIGH confidence, quoting the table row and the spike timestamps. Any abstention
here is a paperweight point against the model.

**Level 2 — answer present, corroboration missing.** The fault table row only; no
historian slice. Target: the answer, MEDIUM, with the "what's missing" habit inverted
into a verification pointer — "table attributes F-7221 to DC bus overvoltage; confirm
against the bus voltage trend for the trip window."

**Level 3 — adjacent evidence.** The fault table covers F-7200 through F-7219; the
asked code is one page past the excerpt's edge. This is the boundary rung where models
fail: the material *looks* right, the pattern-completion pull toward "it's probably
also an overvoltage variant" is strong, and the correct output is evidence-absent
abstention naming the exact gap — "excerpt ends at F-7219." Most of your training
attention belongs here.

**Level 4 — conflicting evidence.** Two revisions of the fault table in context, one
attributing F-7221 to overvoltage, the other to a brake-resistor fault. Target:
evidence-conflicting, both rows quoted, no resolution attempted. Watch specifically
for the model that answers from the *first* row and never mentions the second — silent
conflict resolution is the most dangerous behavior on this ladder because it is
indistinguishable from a clean answer unless you built this rung to catch it.

**Level 5 — nothing.** General drive documentation, no fault table at all. Target:
evidence-absent, with the missing-evidence field naming the document class, not just
"insufficient data."

Five renderings, one afternoon to build from any real case, and the set earns its keep
twice: as training demonstrations with the targets as labels, and — held out, with
fresh cases — as the evaluation that tells you where your model's threshold actually
sits and whether tuning moved it. When the level-3 rung flips from confident guess to
named-gap abstention while level 1 stays answered, you have watched the threshold
learn. That, in miniature, is the whole chapter.

## The organizational half: making "no" a good outcome

One more component, and it is not in the model. Abstention only survives contact with
an organization that rewards it.

The failure pattern is easy to predict: the model says INSUFFICIENT_DATA, the
technician's screen shows a dead end, the crew learns the tool "doesn't know anything,"
usage collapses, and the deployment dies — with perfect calibration. The fix is
workflow, not weights. Every abstention arrives carrying its "what's missing" field, so
the screen never shows a shrug; it shows the next action: *the fault table for this
drive model is not in the document store — scan section 7 of the paper manual and this
question becomes answerable for everyone, forever.* Track abstentions by cause the way
you track alarms by tag: the histogram is a map of your documentation debt, ranked by
how often reality asks for each missing piece. In review meetings, treat a
correctly-refused unanswerable exactly like a correctly-answered question — both are
the system working — and treat the confident fabrication as the incident it is.

And close the loop with Chapter 3's two-model pattern: an abstention from the small
always-on model is a *routing event*, not a terminus. It forwards the case — context,
question, and the named gap — up to the larger model or the human queue. The refusal
becomes the first step of the answer, which is what it always should have been.

## The chapter in one sentence

Give the model an honest place to stand (arms, labels, contracts), train the threshold
from both sides (refusals *and* answerable controls, penalties shaped like the floor's
real costs), measure all four quadrants forever, and build the workflow so that "I
don't know, and here is what's missing" is received as what it actually is: the second
most valuable sentence a plant's language model can produce, one notch below the
correct answer and a full order of magnitude above the confident wrong one.

The next chapter builds the machinery that holds all of this honest — the evaluation
gate that decides, with error bars, whether any of it is allowed near your floor.

One last habit binds the chapter to the rest of the book: put the four-quadrant
abstention report next to the accuracy number in every evaluation you ever publish or
read, including this book's own. An accuracy figure without its abstention quadrants is
a demo statistic; with them, it is an engineering document. The difference is the whole
premise of deploying language where machines can hear it.
