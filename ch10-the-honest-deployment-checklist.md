# Chapter 10 — The Honest Deployment Checklist

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified.)*

Every discipline that keeps people safe around machines eventually compresses itself
into a checklist — not because the discipline is simple, but because the moment of
decision is busy, and busy moments need the discipline pre-decided. This chapter is
that compression for everything this book has argued. It is written to be printed,
argued over, adapted, and signed; the prose around each check explains what the
one-line version is load-bearing for, because a checklist whose reasons have been
forgotten decays into ritual within a year.

A word on what "honest" means in the title. Not honest as a virtue — honest as an
engineering property, the one this book has been assembling since Chapter 2: a system
whose answers carry their evidence, whose refusals name their gaps, whose numbers
carry their error bars, and whose failures are written down where the next engineer
will find them. Each check below defends one piece of that property. Strike any of
them and the system still runs; it just stops being able to tell you the truth about
itself, which on a plant floor is the beginning of every bad story.

## The ten checks

**One: the evaluation existed before the deployment did.** If the gate (Chapter 6)
was built after the model was chosen, the model chose the gate. The order of
construction is itself evidence: a deployment that can show a dated gate report
predating its go-live has proof it was measured into existence rather than demoed
into it. If you inherit a deployment without one, building the gate retroactively is
the first work order — and running the incumbent through it is often illuminating in
both directions.

**Two: the model was sized by measured failure, not by demo.** Chapter 3's selection
rule, auditable in one question: what did the smaller model fail at, specifically,
and is the failure in the gate set? If nobody can produce the failed cases, the size
was chosen by instinct, and instinct in this field is calibrated by cloud
demonstrations that have nothing to do with your fault tables.

**Three: the rendering is honest about its defects.** Chapter 4's rule — every known
data defect is a visible label — checked by inspection: pick a flatlined sensor, a
historian gap, an unmapped tag, and read what the model actually receives. If the
pipeline papers over any of them, every downstream honesty claim is decorative,
because the model cannot decline what it cannot see missing.

**Four: abstention has arms, training, and numbers.** The schema carries the grades
of no as first-class values; the training set held answerable controls beside the
refusals; and the gate reports all four quadrants, with the confident-fabrication
rate carrying the tightest threshold. Chapter 5 in one sentence: a model that cannot
say "I don't know" fails the gate regardless of accuracy — and a deployment that
cannot show its four quadrants does not actually know whether its model can.

**Five: every published number has an error bar and a date.** The ±10-point lesson,
standing. Any score quoted without its spread and its noise floor is a demo statistic
wearing engineering clothes, and any score without a date will be quoted long after
drift has invalidated it. This check extends to the vendor across the table: ask for
ranges, watch the reaction, learn more from the reaction than the ranges.

**Six: the corpus has clearance flags older than the training run.** Chapter 7's
capture hygiene, checked at the manifest: every record that trained carries a
clearance decision made at write time, the scrub pass ran before the shard was cut,
and the gate set is provably disjoint from training. A model whose corpus cannot
answer "whose text is this and who said we could" is a liability wearing a version
number.

**Seven: recovery is rehearsed, not planned.** Chapter 9 compressed: the cold-start
has been timed this quarter, the checkpoint restore has been executed (not reviewed —
executed), the spare has served a real request at the current version, and the
health checks probe function rather than process. The evidence is a dated drill log.
"We have a recovery plan" without drill dates is the sentence that precedes every
ninety-minute outage.

**Eight: the boundary is stated and physical.** The deployment's charter says, in
writing a technician can quote: verdicts advise, humans act, and nothing the model
emits reaches a controller, a setpoint, or a safety function through any path,
including the informal ones. Chapter 8's rollout ladder ends at assisted on purpose.
This is the check that does not bend for a good quarter or an impressive pilot — the
one line where this book trades ambition for the right to make every other claim.

**Nine: the paper trail exists and is current.** The deployment record (Chapter 8),
the run manifests (Chapter 7's six artifacts), the gate reports in chronological
file, the incident and drill logs (Chapter 9), the errata. One binder — physical or
not — that answers the auditor's question, the insurer's question, and the 2 AM
question with documents instead of memories. If assembling it would take more than an
hour, it does not exist yet.

**Ten: the crew owns it.** The dispositions flow because the crew believes the
corrections improve their tool, not their surveillance file. The abstention histogram
is read in the maintenance meeting as a documentation work-list. Someone whose name
is on the record can say what the system is for, what it must never do, and who to
call. Usage metrics live beside accuracy metrics, and a green gate with an ignored
tool is treated as the failure it is. Every previous check is machinery; this one is
whether the machinery is alive.

## The never list

Ten checks earn a deployment its floor; four sentences bound it permanently. Never
let model output reach a control action without a human decision in between. Never
deploy a model you cannot roll back with a file copy. Never train on data you could
not show its authors. Never publish a number about the system that the system's own
gate did not produce. These four are not policies to balance against throughput —
they are the definition of the practice this book teaches, and a deployment that
breaks one has left the book's coverage, whatever else it has achieved.

## The checklist as a procurement instrument

The ten checks were written for systems you build, but they convert directly into
diligence for systems you buy, and the conversion is worth spelling out because a
purchased deployment skips none of the obligations — it only relocates them.

For each check, the vendor question and the shape of a good answer. One: "show me the
evaluation you ran on data like ours, with its date" — a good vendor produces a
methodology and offers to run it on your cases; a poor one produces a leaderboard.
Two: "why this model size and not the one below it" — good answers cite failed cases;
poor ones cite roadmaps. Three: "what does your pipeline show the model when a sensor
is dead" — the vendor who understands the question is rare and worth shortlisting for
that alone. Four: "show me the four quadrants" — and watch whether abstention is a
concept they have metrics for or a feature they promise to look into. Five: "what is
the run-to-run spread on that number" — Chapter 6 already taught you what the
reaction means. Six: "whose data trained this, under what terms" — an answer that
starts with a pause is an answer. Seven and nine: "walk me through your last recovery
drill and show me the record you would hand our auditor." Eight: "describe the paths
by which your system's output could reach a control action" — the only acceptable
answer enumerates them and shows the human in each. Ten is yours to keep, not theirs
to sell: no vendor can supply a crew that trusts the tool.

A vendor who survives all ten exists and deserves the business. A vendor who bristles
at them has told you, at proposal stage and free of charge, exactly what the
relationship will be like at incident stage.

## The first ninety days

The checklist compresses the book; the calendar decompresses it into a plan a
maintenance department can actually run.

**Weeks one and two** belong to the gate and nothing else: the single question chosen,
the schema frozen, the hundred cases labeled in the two afternoons Chapter 6
promised, the runner built, the noise floor measured five times. Resist the demo
urge; a deployment that starts with its instrument never has to retrofit its honesty.

**Weeks three through six** are Chapter 4's plumbing against the gate's baseline:
decode tables reused, renderings built and defect-labeled, contracts written with the
abstention arms in place, and the whole pipeline in shadow mode against live traffic
— scored nightly, shown to nobody. This is where the surprises surface: the timestamp
formats, the tokenizer-shattered tag names, the historian gap nobody mentioned. Each
one becomes a gate case the moment it is understood.

**Weeks seven through ten** open the advisory stage: badged drafts, disposition
buttons, the crew briefed on what the tool is for and — with equal clarity — what it
is never allowed to do (check eight, said out loud, early, by someone whose name is
on it). The dispositions begin accumulating into Chapter 7's corpus; the abstention
histogram gets its first review in the maintenance meeting.

**Weeks eleven through thirteen** buy down the operational risk: the recovery drill
run and timed, the spare restored and exercised, the deployment record assembled
while everything is fresh, the signature page signed. Somewhere in this window the
gate runs its first monthly cycle on schedule rather than on demand — the moment the
deployment stops being a project and starts being a system.

Ninety days, one line, one question, no heroics. The plants that fail at this do not
fail for lack of talent; they fail by attempting month three's ambitions in week two,
on the strength of a demo, without an instrument. The calendar is the checklist's
enforcement mechanism: nothing on it requires believing anyone — only measuring.

## Keeping the checklist alive

Checklists die two deaths, and both are preventable. The first is ritual decay: the
checks get initialed without being performed, because the system has been fine and
the meeting is long. The antidote is the same one aviation found — tie each check to
an artifact that cannot be initialed into existence. A drill has a timestamp; a gate
report has ranges; a signature page has a date. Auditing the artifacts quarterly
takes an hour and converts the checklist from a promise into a record.

The second death is the waiver: the one check suspended "temporarily" for a good
reason — the pilot that would close a loop just this once, the training run on data
whose flags were almost sorted. The never list exists precisely because the waived
check is how disciplined systems degrade; each item on it was chosen because its
first violation looks reasonable at the time. The rule that keeps waivers honest is
borrowed from Chapter 9's morning-after report: a waiver can only be granted in
writing, with a name, a scope, and an expiry — and an unexpired waiver appears in
every gate report until it closes. Waivers that must be signed and republished
monthly have a way of not being requested.

## Signing it

A checklist nobody signs is a poster. This one takes three signatures, renewed on a
cadence: the engineer who owns the gate, attesting the numbers; the operations owner,
attesting the drills and the record; and the plant's responsible manager, attesting
the boundary. Re-signature triggers are the same as the gate's: model change, schema
change, pipeline change, and the calendar. The signature page lives in the front of
the binder from check nine, which means the binder's first page is three people's
names — and that is the correct first page, because every instrument in this book
ultimately reduces to people willing to put their names beside its claims.

That is also, the reader may have noticed, how this book itself is built. Its models
are named on the cover; its claims carry citations into a public lab record; its
verification is a named human's signature; its review trail publishes with it; and
where its instruments failed, the retractions are printed in the text. The checklist
you just read is the one its publisher runs on its own books. We wrote the successor
chapter to those 518 silent pages the only way that would have been worth doing —
under the same discipline we are asking of you.

## What this edition owes you

A book that demands honesty from deployments owes a closing accounting of its own
gaps, so here is this edition's, in plain text. A number of claims in these chapters
still carry their bracketed markers pointing at lab entries not yet attached; none
ships in a verified edition, and the markers are visible in this draft precisely so
that reviewers can hold us to each one. The war stories from real plant floors — the
voice this book's verifier brings from years among the machines this book is about —
are represented but not yet written; they arrive by interview, not invention, because
a fabricated anecdote in a book about honest instruments would be a foundation crack,
and we would rather show you the empty section than fill it wrong. The evaluation
benchmark this book references has its own public history, including its retractions,
and readers are owed the link rather than a summary flattering to us. And the field
itself is moving: the tier capabilities in Chapter 3 are dated measurements of a
moving frontier, which is why they carry dates and why the gate — not this book — is
your standing authority on what your model can do this quarter.

Those debts are recorded in the same spirit as check nine's binder: visibly, dated,
with names attached, in the review trail that publishes alongside this book. If you
find a claim here that its citation does not support, the errata process on the
provenance page is not a formality — it is the mechanism by which this book remains
the thing it claims to be, and we ask you to use it.

## Where this leaves you

Start smaller than feels ambitious: one question, one schema, one hundred labeled
cases, one box beside one line. Run shadow mode until the surprises stop. Let the
gate, not the vendor, tell you when to climb. Feed the corpus, mind the flags, drill
the recovery, read the load log. And when a colleague from another plant asks what
you are running, hand them the deployment record and the gate report instead of a
demo — because the demo is the genre this whole field needs to grow out of, and the
record is what growing out of it looks like.

The machines have been describing themselves to computers for forty years. The
computers can finally read. What happens next on your floor depends less on the
models than on the honesty of the instruments you build around them — and that part,
every line of it, is in your hands.
Good luck out there — and write your own log entries the same day you earn them.
