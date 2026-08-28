# Chapter 3 — Why Small

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified. `[R-TBD]` marks numbers
awaiting lab entries.)*

Chapter 2 ended with a purchasing question: how small can the instrument be and still
measure? This chapter answers it the way an engineer would want it answered — with a
sizing procedure rather than a slogan. The slogan version of this field says "bigger is
better" out of one side of its mouth and "small models are the future" out of the other.
Both are marketing. What you actually need is the thing this chapter builds: a ladder of
size classes, what each class can honestly hold, what each costs to run, and a selection
rule that starts from your task instead of from a leaderboard.

## The arithmetic that decides everything

Before capability, physics. A model's size is quoted in parameters — the count of learned
weights — and parameters are just numbers that must live somewhere. The arithmetic is
mercifully simple. Stored at full 16-bit precision, one parameter costs two bytes: an
8-billion-parameter model is roughly a 16-gigabyte file. Quantized carefully to around
four bits per weight — the mature end of the discipline Chapter 1 described — the same
model needs roughly five gigabytes, plus working memory for the context (the KV cache,
which grows with window length and can rival the weights for long contexts).

Run the same arithmetic down the ladder and the hardware map draws itself. A model in the
hundreds of millions of parameters fits in under a gigabyte — single-board-computer
territory. A 1-to-2-billion-parameter model quantizes into a couple of gigabytes —
comfortable on any modern industrial PC, even without a discrete GPU. The 7-to-8-billion
class wants a workstation GPU or a unified-memory machine and repays it with a real jump
in capability. The 30-billion class is the ceiling of "hardware a plant would actually
buy" — a serious GPU workstation — and above that you are building a server room, which
is a different book.

Speed follows memory. Generation is mostly a memory-bandwidth exercise: every token
requires streaming the active weights past the processor. Small models are fast on
modest hardware not because of any cleverness but because there is less to stream. When
a vendor quotes tokens-per-second, you now know what they are mostly measuring: the
memory system, at a given model size and precision `[R-TBD: tok/s by tier on reference
hardware]`.

## The ladder, with honest rungs

Our lab maintains a working ladder of size classes, each earning its place by measured
capability rather than by roadmap `[R-TBD: tier capability matrix]`. Names vary across
the industry; the classes do not.

**Sub-billion (the "pocket" class).** What it honestly does: classification, tagging,
field extraction from consistent formats, enum-constrained verdicts of the Chapter 2
kind — provided it was trained or tuned on text like yours. What it does not do:
open-ended reasoning, multi-step tool use, graceful handling of surprises. Used inside
its envelope, this class is a workhorse: it is cheap enough to run continuously against
a data stream, which changes what you can afford to monitor. One honesty note this book
will repeat, because the marketing around this class is the worst of any rung: sub-billion
still means a real computer. It does not mean a microcontroller. The gap between "runs on
a Raspberry-Pi-class board" and "runs on the 200-kilobyte microcontroller inside your
sensor" is measured in orders of magnitude, and no current language model of useful
ability crosses it.

**1-to-2 billion (the "line side" class).** The smallest class where instruction-following
becomes dependable enough to build on: it reads a prompt template it was not specifically
trained on and mostly does what the template says. Extraction quality rises; abstention
training (Chapter 5) starts genuinely working rather than being parroted `[R-TBD:
abstention-by-tier]`. This is the class we reach for first when a task must run on the
plant's own modest hardware with no GPU budget.

**7-to-8 billion (the "engineer's assistant" class).** The knee of the curve in our
measurements `[R-TBD]`. Multi-step behavior appears: read the fault table, then check
the historian excerpt, then produce the schema-constrained verdict, without the seams
showing. General knowledge is broad enough that the model degrades politely outside its
specialty instead of collapsing. If the plant can afford one GPU box, this class is
usually where it should live.

**~30 billion (the "department" class).** The largest rung this book takes seriously for
on-premises work. What you buy with the extra memory and money is mostly *robustness*:
fewer prompt-engineering hours per task, better recovery when inputs are messy, better
judgment about its own uncertainty. Whether that robustness is worth roughly four times
the hardware of the 8-billion class is a per-plant decision — Chapter 6's evaluation
harness exists precisely so you can answer it with your own data instead of ours.

## The specialist trap

There is a tempting shortcut at every rung: tune the model so hard on your domain that it
excels at your benchmark and nothing else. The trap is that "nothing else" includes the
connective tissue that makes a model usable — following slightly novel instructions,
handling a question phrased a way the training set never phrased it, writing a coherent
sentence about an adjacent topic. A specialist that has lost its general footing fails
strangely and often silently: it does not know that it has left its envelope, and neither
does its output.

Our lab's rule, learned by measuring the failure `[R-TBD: retention gate]`: **every
specialized model must also hold a floor on general benchmarks, and that floor is a
shipping gate.** Specialization is supposed to be an addition, not an amputation. When a
vendor shows you a domain benchmark, the question that exposes the trap is one sentence:
"what did the general scores do while the domain scores went up?"

## The tokenizer is part of the size budget

Chapter 2 introduced the tokenizer as the model's alphabet. At small scales the alphabet
becomes a sizing decision, and it is the most under-discussed lever in the small-model
literature. A vocabulary tuned to your text means your tag names, codes, and units are
few tokens instead of many fragments. Every fragment saved is context window reclaimed,
attention un-wasted, and — in training — capacity spent on meaning instead of spelling.
In our from-scratch work, tokenizer choices behaved like a meaningful fraction of the
parameter budget: the same weight count went measurably further with an alphabet that
matched the corpus `[LAB: PROJECT-LOG 2026-08-03 + matrix §O.1 — 32k industrial tokenizer: +3.5–7.9% chars/token on industrial text at 1/4 the vocabulary; embedding 134M vs 621M params, 487M freed for layers]`.

The practical consequence for a buyer: two models of identical parameter count are not
the same size on *your* data. The one whose tokenizer shatters your identifiers is
effectively smaller — sometimes much smaller — for your purposes. Chapter 6's evaluation
design accounts for this by benchmarking on your text, never on generic text.

## Small as an operational property

The case for small models is usually argued on cost, and the cost case is real. But live
with a deployment for a while and different virtues dominate:

- **Small restarts fast.** A model that loads in seconds changes maintenance windows,
  crash recovery, and how casually you can ship an update. Chapter 9's recovery drills
  assume load times measured in seconds to low minutes — realistic in the small classes,
  fantasy above them `[R-TBD: load-time by tier]`.
- **Small runs redundant.** Two modest boxes running the same 2-billion model is a
  failover story a plant understands. One large shared model is a single point of
  failure with a queue in front of it.
- **Small stays cool.** Watts matter in a sealed cabinet on a hot mezzanine. The
  difference between a model that idles at single-digit watts on an edge box and one
  that needs 300-watt-class GPU cooling decides where the hardware can physically live.
- **Small is auditable.** When Chapter 6's gate flags a regression, a small model
  retrains or retunes on a timescale that keeps the fix inside the same week. Iteration
  speed is a quality property: the model you can afford to fix is the model that ends up
  correct.

## The two-model pattern

One more configuration belongs on the menu before costs, because it dissolves many
apparent dilemmas: run two tiers, not one. A small always-on model handles the
continuous stream — classifying, extracting, filing verdicts — and *escalates* the cases
it abstains on to a larger model that wakes only when called. The small model's
abstention training, which Chapter 5 builds anyway, becomes the routing signal for free:
"I don't know" stops being a dead end and becomes a transfer of custody.

The economics are lopsided in the pattern's favor. The stream is overwhelmingly routine,
so the expensive model runs a small fraction of the time on exactly the cases where its
extra judgment earns its electricity; the cheap model soaks the volume on hardware that
costs less than a valve. The operational story improves too: the always-on component is
the simple, fast-restarting, redundant one, while the complex component is allowed to be
slower and singular because nothing depends on it minute-to-minute. And the audit story
is cleaner than either model alone: every escalation is a logged decision with a stated
reason, which is more than most human triage produces. When later chapters seem to force
a choice between a model small enough to trust operationally and one large enough to
handle the ugly cases, remember that the fork is usually false — the answer is a
hierarchy, and the plant already runs everything else that way `[R-TBD: escalation-rate
and cost split from lab deployment]`.

## The cost table you actually need

Cloud pricing and local hardware are quoted in units designed not to be compared, so
build the comparison yourself; the arithmetic fits on an index card. A hosted model
charges per token in and out. A continuous plant workload is easy to estimate: suppose
one modest monitoring task reads a few thousand tokens of context and writes a few
hundred tokens of verdict, once a minute, around the clock. That is on the order of a
few billion tokens a year for a single task — before you add the second task, the second
line, or the engineer who starts asking the thing questions because it turns out to be
useful. Multiply by the per-token price of any competent hosted model and you get an
annual bill that recurs forever, grows with adoption, and buys you nothing you keep.

Now price the local alternative. The industrial PC that runs the line-side class is a
one-time purchase in the low four figures; the GPU workstation that runs the assistant
class, mid four figures. Electricity for either is real money but small money. The
crossover point — where owning beats renting — arrives within the first year for any
workload that runs continuously, and the comparison only widens after that, because the
owned box serves the second task and the third at zero marginal cost. The cloud keeps
its advantage where usage is occasional, spiky, or exploratory: a monthly report, a
one-off analysis, a prototype you have not committed to. This book's subject is the
other kind of workload — the kind plants actually have — where something watches a
stream all day, every day. For that shape of demand, the rental arithmetic never wins
`[R-TBD: worked cost comparison at reference prices]`.

There is also a cost the table cannot hold: the meter changes behavior. Teams ration a
metered model — they ask it less, wire it into fewer places, and quietly stop
experimenting. An owned model gets used the way an owned oscilloscope gets used:
constantly, casually, and for questions nobody would have paid per-minute to ask. Some
of those questions turn out to be the valuable ones.

## Concurrency: the sizing axis everyone forgets

The ladder so far assumed one request at a time. Real deployments do not: five stations
ask questions during the same shift change; the monitoring task fires while an engineer
is mid-conversation. Two facts change the sizing picture.

First, serving engines batch. Multiple simultaneous requests share each pass through the
weights, so a box that produces some number of tokens per second for one user produces
far more *total* tokens per second for eight users — throughput scales much better than
intuition expects, at modest cost to each individual response `[R-TBD: throughput vs
concurrency on reference hardware]`. A single well-sized box genuinely can serve a
department.

Second, memory is the ceiling on that trick. Every concurrent conversation holds its own
context in the KV cache, and long contexts multiplied by many users can outgrow the
weights themselves. When a serving box that benchmarked beautifully starts refusing
requests at shift change, the diagnosis is almost always cache exhaustion, not model
weakness. The sizing rule of thumb: budget memory for the model *plus* your worst-case
simultaneous contexts, and prefer a smaller model with generous cache headroom over a
larger model wedged against its memory limit. The smaller model answers everyone; the
larger one answers nobody at exactly the moment demand peaks.

## A sizing walkthrough

Put the whole chapter to work on one concrete task: extracting structured fields —
machine, component, symptom, action taken — from free-text maintenance work orders, a
few hundred per day, into the CMMS.

Step one, the evaluation: two hundred real work orders, hand-labeled by a maintenance
lead over two afternoons, with a scoring rule per field and an explicit abstention arm
for illegible entries. Step two, start low: the pocket class, tuned on a few thousand
historical orders. Suppose it lands high on machine and component but noticeably lower
on symptom, where the prose gets idiosyncratic `[R-TBD: walkthrough numbers]`. Step
three, attribute before climbing: inspection shows half the symptom misses trace to
technicians' shorthand the tokenizer shatters, and a vocabulary adjustment plus a
grounding tweak recovers most of it. The pocket model now passes every field but
symptom, which sits just under gate. Climb one rung, not two: the line-side class
clears the gate with margin on unchanged data. Step four, stop. Record the margin, pin
the model version, and resist the voice suggesting the assistant class "to be safe" —
safety is the eval gate you just built, not spare parameters.

Total hardware: one GPU-less industrial PC. Total model cost: zero dollars of licensing.
The expensive ingredients were the two afternoons of labeling and the discipline not to
buy capability the measurement said you did not need. That ratio — labeling and
discipline over hardware and hype — is this book's cost structure in miniature, and it
recurs in every chapter ahead.

## The selection rule

Assemble the chapter into procedure form:

1. Write the task as an evaluation first — real documents, real faults, a scoring rule,
   an abstention arm. (Chapter 6 is the how.)
2. Start two rungs *below* where your instincts say. Instincts are calibrated by cloud
   demos; floors are cheaper than they suggest.
3. Climb only on measured failure: move up a rung when the smaller class fails your gate
   for reasons more capability would fix — not for reasons better grounding, a tighter
   schema, or a matched tokenizer would fix. In our experience the majority of "the model
   is too small" complaints dissolve at step three's inspection `[R-TBD: failure
   attribution tally]`.
4. Stop at the first rung that passes with margin, and record the margin: it is your
   early-warning gauge when the task drifts.

The smallest model that survives your gate is not a compromise. It is the correctly
sized instrument — and on a plant floor, correctly sized is what "professional" means.
The next chapter turns to what these instruments read all day: the protocols and
historians that speak for your machines.
