# Chapter 2 — A Language Model, for People Who Own Machines

*(draft v1, 2026-08-28 — written by Claude Fable 5, verified by Roger AI 2026-08-28. Lab citations attached where the record exists; remaining claims are labeled as unmeasured.)*

You have a mental model for every machine on your floor. You know a pump moves fluid by
spinning an impeller, and that knowledge tells you what a cavitation noise means. You know
a PLC scans its ladder top to bottom, and that knowledge tells you why an interlock
races. You do not need to be able to build these machines; you need a model of them good
enough to predict how they fail.

This chapter gives you that mental model for a language model. Not the mathematics — the
failure-prediction model. By the end you should be able to look at an LLM's answer the way
you look at a gauge reading: knowing what the instrument actually measures, and therefore
knowing when to trust it.

## The wrong mental model, first

Almost everyone arrives with the same picture: the model is a very large database with a
very good search box. Ask it a question, it looks up the answer, and sometimes the lookup
fails.

That picture is wrong in a way that matters on a plant floor. There is no lookup. A
language model stores nothing the way a historian stores tags or a manual stores torque
specs. What it stores is a single, enormous set of numbers — the weights — tuned so that,
given a stretch of text, the model produces a good guess about what text comes next. That
is the whole machine. Everything else you have heard about it is a consequence of that one
sentence.

The correct mental model is closer to this: **an extremely well-read colleague with no
access to any documents, answering everything from memory, one word at a time, who is
physically incapable of staying silent.** Every strength and every failure mode in this
book falls out of that description.

## Tokens: the model's alphabet is not yours

The model does not read letters or words. Before your text reaches it, a *tokenizer*
chops the text into pieces called tokens — common words become one token, rare words
shatter into several, and the vocabulary of pieces is fixed when the model is built.

Why should you care? Because your plant speaks a vocabulary the tokenizer has never
prioritized. A sentence of ordinary English might cost one token per word. A fault code
like `E-4127-B`, a tag name like `LINE3_CONV_VFD2_AMPS`, or a protocol frame rendered in
hex will shatter into many single-character fragments. The model spends more of its
limited attention just holding your identifiers together, and it has weaker instincts
about pieces it rarely saw in training.

This is not a cosmetic detail. In our lab work on small industrial models, the tokenizer
turned out to behave like part of the parameter budget: a vocabulary that matches the
data means the model spends its capacity on meaning instead of on spelling `[LAB: PROJECT-LOG 2026-08-03 + matrix §O.1 — a from-scratch 32k tokenizer beat 100k/151k vocabularies on industrial text while freeing 487M parameters for layers at fixed budget]`. When Part II discusses reading protocols and
historians, tokenization is the first thing we will fix, not the last.

The practical instinct to build now: when a model mishandles an identifier — drops a
digit, merges two tag names, "corrects" a fault code to a more common one — suspect the
alphabet before you suspect the intelligence. The failure often begins before the model
proper ever runs.

## Weights: what "knowing" means when there is no database

Training a language model means showing it a mountain of text and adjusting the weights,
billions of times, so its next-piece guesses improve. What survives training is not the
text. It is a *compression* of the text's regularities: grammar, idiom, the shape of a
service procedure, the fact that "cavitation" appears near "suction" and "NPSH" far more
often than near "firmware."

Compression explains both the magic and the danger:

- The magic: the model can answer questions no single training document answered, because
  regularities generalize. It has read a thousand pump manuals; it has a *shape* for pump
  manuals, and it can fill that shape with your pump's specifics — if you supply them.
- The danger: compression is lossy. The regularities survive; the exact torque value, the
  exact register address, the exact revision date often do not. The model retains what
  things *sound like* more reliably than what things *are*.

This is why asking a bare model for a torque spec is malpractice even when it answers
confidently. The confident tone is part of the compressed shape of manuals — manuals
never sound unsure — and it attaches to the answer whether or not the number survived
compression. On the floor, treat an LLM's tone the way you treat a salesman's:
information about style, not about truth.

## The context window: working memory, and why it changes everything

There is one place the model *does* read verbatim rather than remember: the context
window. Everything you paste into the conversation — your question, the manual excerpt,
the historian export, the model's own previous answers — sits in a buffer the model
attends to directly while generating. The buffer is real, exact, and bounded. Nothing
outside it exists.

Three floor-level consequences:

1. **The context is your instrument of truth.** A model that "knows" nothing reliable
   about your fault code will read a pasted manual page about that fault code perfectly
   well. The single highest-leverage practice in this entire book is: put the document in
   the window and instruct the model to answer *from the document*. Chapter 4 builds
   machinery for exactly this.
2. **The window is smaller than your data.** A shift's worth of historian output for one
   line can exceed the entire context budget of a small model. You do not "give the model
   the data"; you give it a *selection*, and the selection logic — what to include, what
   to summarize, what to drop — is engineering you own, not magic the model performs.
3. **Forgetting is architectural, not moody.** When a long troubleshooting session drifts
   past the window, the earliest turns fall out — including, often, the safety constraint
   you stated at the start. A model that "suddenly ignored" an instruction usually never
   received it: the instruction had already scrolled off the edge of the world.

## Generation: one piece at a time, dice in hand

The model produces output the same way it reads input: token by token. At each step it
computes, for every token in its vocabulary, a probability that this token comes next —
then one token is *chosen*, appended, and the whole process repeats with the window one
token longer.

Chosen how? That is the sampling policy, and it is a knob you control. At `temperature 0`
the policy is "always take the most probable token." Raise the temperature and lower-
probability tokens get a real chance — more varied prose, more creative connections, and
more ways off the rails. For plant work you will almost always run cold. Creativity is a
liability in a fault diagnosis.

Two honest footnotes from our benches, because they will bite you:

- Cold does not mean deterministic in practice. Identical prompts at temperature 0 can
  return different outputs across runs, because inference servers batch requests together
  and floating-point arithmetic is order-sensitive. On one of our tool-use suites the
  same configuration swung noticeably between identical runs from this effect alone
  `[LAB: RESULTS-MATRIX §C footnote — temp-0 flips traced to PAR=2 batch-packing nondeterminism; ±10 pts on a 15-scenario suite]`. If your acceptance test
  assumes bit-identical replays, it will fail for reasons that have nothing to do with
  the model's quality.
- The model cannot abstain by default. At every step, *some* token is chosen — the
  machinery has no built-in "say nothing." An untrained model's "I don't know" is just
  another sentence it learned the shape of, produced when the context makes that shape
  probable. Making abstention *reliable* — making the model prefer silence when evidence
  is thin — is trained behavior, and it is the heart of Chapter 5.

## Hallucination is not a bug report; it is the spec

Now assemble the pieces. A machine that (a) compresses its training text lossily,
(b) answers from memory unless the document is in its window, (c) must emit a next token
no matter what, and (d) learned that authoritative prose is the most probable shape of an
answer — that machine *will* sometimes produce fluent, specific, wrong statements. Not as
a malfunction: as the normal operation of exactly the mechanism described above.

The industry calls this hallucination. The name misleads, because it suggests a rare
pathological state. It is better to think of it the way you think of measurement noise:
always present, larger in some regimes than others, and manageable with the right
instrumentation. The regimes where it spikes are predictable — rare identifiers (see
tokens), specific numbers (see compression), questions just past the edge of what the
context contains, and long generations where early small errors compound.

The entire design of this book's middle chapters is instrumentation against exactly this:
grounding answers in windowed documents, constraining output formats, training abstention,
and gating everything through evaluation that would rather reject a right answer than
pass a wrong one.

## The plant floor's secret advantage: constrained output

Here is the part of the mental model that most general-purpose AI writing skips, and it
happens to be the part where industrial work *wins*.

Because generation is a per-token choice among alternatives, you can lawfully forbid
alternatives. If the answer must be one of `{RUNNING, IDLE, FAULTED, ESTOP}`, the
inference engine can zero out every token that could not begin a legal answer and choose
only among the legal ones. This is grammar-constrained decoding, and on structured
industrial questions — enumerations and fixed schemas, where the whole legal space is
small — it converts "mostly formats correctly" into "cannot emit a token outside the
grammar." Two caveats keep that honest. It is a property of the *engine* enforcing the
grammar: constrained decoding is itself a gated flag (Chapter 8), and an engine that
falls back to free sampling formats incorrectly again, so grammar support belongs in the
gate's checklist. And it constrains *shape*, not *truth* — where a schema carries a
free-text field, the grammar guarantees the field is present, not that its contents are
right. Inside those limits, the model's judgment still picks *which* legal answer, but the
space of expressible format mistakes collapses.

Free text is where language models are weakest; enumerations, schemas, and protocol
fields are where your domain lives. That asymmetry is a large part of why small local
models can hold their own on the floor `[LAB: RESULTS-MATRIX R.158 — IEB-Signals v1.3 private holdback, n=3,725, deterministic enum decode: channel-level acc 87.61% / AUROC 0.938 / ECE 0.012; scene-level acc 47.89% / AUROC 0.548. Bit-exact across two process launches on 640 rows]` — Chapter 4 makes
it concrete.

## What "small" changes, in behavior rather than benchmarks

Everything above is true of models from a quarter-billion to a trillion parameters. Size
changes the *reach* of each property, and the honest summary for a plant engineer is:

- Small models hold less compressed world. They lean harder on the context window — which
  is fine, because on the floor the context (your manual, your historian) is the part you
  actually trust.
- Small models generalize less far from what they were trained on. A general-purpose
  small model is mediocre at industrial text; the fix is training on industrial text,
  which is the subject of Chapter 7.
- Small models fail less mysteriously. There is less capability to be surprised by in
  either direction — a property that reads as a weakness in a demo and as a virtue in a
  safety review.

What size does *not* change: the mechanism. A trillion-parameter model also answers from
lossy memory, also cannot exceed its window, also emits tokens it cannot silently
withhold. The failure modes you learned in this chapter are not small-model failure
modes. They are language-model failure modes, and the cloud does not exempt anyone from
them.

## A worked example: one fault code, three ways

Make it concrete. A drive on a packaging line trips with fault `F-7221`. You have a
service manual PDF, a maintenance historian, and a small local model. Three ways to ask,
three different machines you are operating.

**Way one: the bare question.** "What does fault F-7221 mean on this drive?" The model
now answers from compressed memory. If that code is common across many manuals it read,
you may get a correct family-level answer. If the code is rare — and your plant's codes
mostly are — the mechanism from this chapter predicts what happens next: the tokenizer
shatters the identifier, memory holds no strong pattern for it, and the "shape of a
manual answer" fills the vacuum. You receive a fluent paragraph about overcurrent that
may belong to a different vendor's drive entirely. Nothing malfunctioned. You asked
memory for something memory never reliably held.

**Way two: the document in the window.** Same question, but you paste the manual's fault
table first and add one instruction: "Answer only from the excerpt above; if the excerpt
does not contain the answer, say so." Now the model is doing the thing it does best —
reading verbatim text in its window and reorganizing it. The answer cites the actual row:
DC bus overvoltage, check decel time and brake resistor. The quality jump between way one
and way two is larger than the jump between a small model and a frontier model on way
one. That comparison is the cheapest experiment you can run yourself, and it is the
single fact that reorganizes how teams use these tools. We have not yet published a head-to-head grounded-versus-bare industrial Q&A table; until that measurement exists, treat the claim as a design rule, not a score.

**Way three: the constrained verdict.** You are not writing an essay; you are deciding a
dispatch. So you ask for a structured verdict and constrain the output grammar:
`{"probable_cause": one of [DECEL_TOO_FAST, BRAKE_RESISTOR, BUS_SUPPLY, UNKNOWN],
"evidence": "<quote from excerpt>", "action": one of [DISPATCH_ELECTRICAL,
RESET_AND_MONITOR, ESCALATE]}`. The model can no longer produce an unparseable answer,
an out-of-vocabulary cause, or a missing evidence field — the decoder forbids it. Note
what the constraint also gives you: `UNKNOWN` is now a first-class, always-available
answer, which is half of abstention engineering before any training happens.

The three ways are one model with three different instruments wrapped around it. Reading
this book is largely learning to stop operating way one while believing you are
operating way two.

## Questions this chapter equips you to ask a vendor

A mental model is also armor. When someone demos an AI product for your plant, the
mechanism you now hold converts marketing into checkable claims:

1. *"What is in the context window at the moment of this answer?"* If the demo cannot
   say, the demo does not know where its answers come from — and neither will you.
2. *"Show me the same question with the document removed."* The gap between grounded and
   bare answers is the product's real value; a demo that refuses the comparison is
   selling the model's memory, which you now know is the unreliable part.
3. *"What happens on an identifier the model has never seen?"* Ask them to query a tag
   name you invent on the spot. Watch for the confident wrong answer this chapter
   predicts.
4. *"Can the output be grammar-constrained to our schema, including an UNKNOWN arm?"*
   If the answer is no, the product has declined the plant floor's best trick.
5. *"Is temperature zero, and do identical runs reproduce?"* Any hedging here means
   nobody has actually run the acceptance test twice.

None of these questions require mathematics. They require exactly the mechanism in this
chapter, applied with the same skepticism you would bring to a pump curve. A vendor
comfortable with all five is worth another meeting; a vendor irritated by them has told
you what the demo was hiding, and saved you a pilot's worth of budget in the process.

## The mental model, on one page

A language model is a next-token guesser: an extremely well-read colleague with no
documents, answering from lossy memory, one token at a time, incapable of silence unless
trained for it, reading verbatim only what fits in a bounded window, with a tone
calibrated to sound like documentation regardless of truth — and, uniquely useful to us,
willing to have its output constrained to a legal vocabulary you define.

Trust it the way you trust any instrument: within its measurement principle. The rest of
this book is the calibration procedure, and the next chapter begins it by answering the
question every purchase starts with: how small can the instrument be and still measure?
