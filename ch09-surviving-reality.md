# Chapter 9 — Surviving Reality

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified. This chapter's evidence
is unusually direct: our own lab's failure log, cited by date.)*

Every chapter before this one assumed the computer stays on. This one is about the week
that assumption failed twice.

Our lab runs language models the way this book proposes a plant should: continuously, on
owned hardware, with training jobs, serving processes, and data pipelines sharing a
building's worth of dependencies. In one August week that building lost power twice. The
first outage cost about an hour and a half of recovery work. The second, two days later,
cost about twenty-five minutes — not because it was gentler, but because the first crash
had been treated as an engineering deliverable instead of a bad day `[LAB: PROJECT-LOG
2026-08-22 and 2026-08-24 — power-loss recoveries #1 and #2]`. The delta between those
two numbers is this chapter. Everything in it was paid for.

A plant engineer will recognize the shape of what follows, because none of it is AI
engineering. It is the ordinary discipline of keeping industrial systems restartable —
applied to a stack that the AI industry, raised in data centers with generators, mostly
ships without it.

## What a power loss actually breaks

Walk the damage path of a hard power cut through a model-serving stack, because each
stage is a design decision you can make ahead of time:

**Storage is hurt first and lies about it.** Filesystems with journaling recover to a
consistent state by design; filesystems without it — including network-share formats
mounted through compatibility bridges — can be silently damaged and need offline repair
before they mount at all. Our first crash spent most of its ninety minutes here, on
exactly one lesson: know, for every volume your stack touches, what happens to it when
the power dies mid-write. The fix was boring and total — repairs to mount configuration
so every data volume came back automatically and consistently. In the second crash,
storage cost *zero* minutes `[LAB: PROJECT-LOG 2026-08-24 — "the crash-#1 fixes held;
zero filesystem work"]`. Boring and total is the grade to aim for.

**Services resurrect in the wrong order — or resurrect when told not to.** Modern init
systems restart what was running, which is what you want until it isn't. Our second
crash surfaced a subtle version: a service we had explicitly disabled came back anyway,
because three *other* services declared it as a dependency, and dependency pulls ignore
the disabled flag. It occupied most of a GPU's memory before anyone noticed, and the
training jobs that were supposed to own that GPU found it taken. The durable fix was not
a stronger "off switch" but a *condition*: the service's unit file now checks for a
hold-marker file on disk and refuses to start while the marker exists — an interlock,
in plant terms, rather than a request `[LAB: PROJECT-LOG 2026-08-24 — dependency pull
vs. disablement; condition-gated unit fix]`. The general rule: on shared hardware,
"stopped" enforced by intention decays; "stopped" enforced by a condition survives
reboots, dependency graphs, and colleagues.

**In-flight state is simply gone.** Whatever the model was generating, whatever batch
was mid-flight, whatever fine-tune step was between checkpoints — vanished. You do not
protect in-flight state; you bound it, which is the next section.

## Checkpoints are a cadence decision, not a feature

Long-running model work — a fine-tune, an index build, a corpus pack — survives power
loss exactly as well as its checkpoint cadence, no better. Our training runs checkpoint
every few thousand steps; when the first crash hit, the running jobs lost roughly three
to four hours of progress each — annoying, bounded, and resumable to the step, with
random state restored so the run continued as if unbroken. The week's two crashes cost
about nine and a half GPU-hours of redone work in total, and the log entry closes with
the sentence that matters: the cadence *bounds each loss* under four and a half hours
`[LAB: PROJECT-LOG 2026-08-24 — checkpoint cadence bounding recovery cost]`.

Translate the arithmetic to your floor. The cadence question is: how many hours of this
work am I willing to redo? Divide by two for safety margin, checkpoint at that interval,
and *verify a resume actually works* — a checkpoint you have never restored from is a
hope, not a checkpoint. The same logic covers the humbler state a serving deployment
accumulates: retrieval indexes, caches, configuration. If rebuilding it is fast, let it
rebuild; if it is slow, it is a checkpointing customer too.

For pure serving — the model answering questions — the news is better and it is a
genuine advantage of the small-model classes from Chapter 3: the "state" is a read-only
weights file plus a process. Recovery is: mount storage, start service, load weights,
health-check. Small models load in seconds to low minutes, which makes the whole
recovery a watchdog script rather than an operation.

## Verify by artifact, not by appearance

The second crash's log records a near-miss that deserves its own section, because the
class of error is universal.

During recovery, an operator checked whether the data-repacking jobs had survived by
listing processes and pattern-matching the command names. Two matches came back; the
conclusion "both packers are running" was one keystroke from being recorded. Both
matches were the *listing command itself* — the search pattern matched its own
invocation. The jobs were dead, and had the appearance been trusted, a large data
pipeline would have sat idle indefinitely while dashboards showed green `[LAB:
PROJECT-LOG 2026-08-25 — self-matching process check near-miss]`. The catch came from
checking the *artifact* instead: the job's log file had not been written since before
the crash. Mtime does not pattern-match itself.

The same log family records the inverse trap: a cleanup command that kills processes by
name-pattern matched the operator's own shell — the pattern found itself — and cut the
session out from under the recovery `[LAB: CLAUDE.md hardware notes — pkill self-match
trap]`. Two incidents, one lesson, and it generalizes far beyond Linux: **status checks
must observe what the work produces, not what the process table appears to contain.**
A packer is alive if its output file grew recently. A serving model is alive if a
health-check request returns a token. On a plant floor you already know this — you trust
the flow meter, not the pump's power light — and the discipline transfers unchanged.

## Read the log before trusting the plan

One more lab scar with direct floor application. A production model once served at a
catastrophic fraction of its normal speed — the kind of number that triggers a tuning
spree: flags, batch sizes, memory splits, none of it moving anything. The answer was a
single line in the *load* log, printed at startup, showing one component had quietly
loaded onto the wrong processor `[LAB: PROJECT-LOG — the 2 tok/s mystery; one load-log
line]`. Minutes of reading would have saved the hours of tuning.

The rule we wrote afterward: **when a number is pathological — not merely low, but
absurd — stop adjusting and start reading.** Pathological numbers are almost never
tuning problems; they are configuration problems announcing themselves at boot in a
log nobody reads. Your deployment's startup log should be short enough to read and
read on every deployment; Chapter 8's serving recipes print the facts that matter —
device placement, precision, memory reserved — precisely so this rule is cheap to
follow.

## The UPS question, answered honestly

The reflex response to this chapter is "buy a battery." Do — and understand precisely
what it buys. An uninterruptible supply sized for a serving box is not there to ride out
the outage; plant outages outlast affordable batteries with ease. It is there to buy
*minutes*, and minutes are only valuable if something spends them: a shutdown hook that
sees the on-battery signal, stops accepting new requests, lets in-flight work drain,
forces a final checkpoint, and unmounts storage cleanly. A UPS without that integration
converts a hard crash at power loss into the identical hard crash twenty minutes later,
at a time nobody predicted, with a false sense of security billed on top.

Brownouts deserve more fear than blackouts. A clean cut is the *easy* case — everything
stops, everything restarts, this chapter's machinery handles it. A sag reboots some
devices and not others, corrupts the states of equipment that half-survived, and
produces the weird Tuesday where the model box is fine but the switch between it and the
historian silently power-cycled and dropped its config. When a deployment misbehaves
after "a power event," widen the suspect list to every box in the path before blaming
the one with the AI on it; in our experience the exotic component is presumed guilty and
is usually the innocent one.

And test the battery the way you test a checkpoint: by using it. A UPS that has never
carried the load through a rehearsed shutdown is, like the unrestored checkpoint,
folklore with a purchase order.

## Watchdogs, and who watches them

Restart-on-failure is one line of configuration and everyone sets it. The engineering is
in the layer above: deciding what "failure" means and noticing when restarting stops
helping.

A serving process can be up, listening, and useless — weights half-loaded, memory
exhausted by Chapter 3's KV-cache ceiling, or wedged in a state where every request
times out. A process-level watchdog sees a running process and stays quiet. The health
check that means something is end-to-end: send a real, tiny inference request on a
timer; require a token back within a deadline; restart on misses. That single design
choice — probe the *function*, not the process — catches the entire family of
alive-but-dead states, and it is the same artifact-not-appearance rule from earlier
wearing a uniform.

Then bound the restarts. A service that crashes on startup will crash-loop forever at
whatever cadence you allow, and a crash-looping service generates exactly the alert
storm that trains a crew to ignore alerts. Back off between attempts, cap the attempts,
and after the cap, *change the message*: "restarting" is routine noise; "restarted five
times and stopped trying" is a page. The worst outcome of a monitoring design is not a
missed failure — it is a crew that has learned the alarms are wallpaper. Plants know
this as alarm management; the AI box gets no exemption from it.

Log the boring successes, too. When the health probe passes, a timestamped line lands
in the artifact trail — which means the *absence* of that line is itself detectable by
the next layer up. Silence, as our monitoring rules put it, must never be mistakable
for health.

## Heat: the failure that arrives on schedule

Power loss is dramatic; heat is patient. A GPU in a sealed cabinet on a mezzanine in
August does not crash — it *throttles*, quietly trading speed for temperature, and your
deployment's response times drift upward with no error message anywhere. Our lab
learned to treat thermal configuration as a first-class deployment parameter after
measuring how much performance a power-and-cooling ceiling actually costs on sustained
load — and, more usefully, that the right power cap costs far less than the wrong
airflow `[LAB: MAXQ-THERMAL 2026-08-06 — power-cap vs. throughput measurements]`.

The floor rules that fall out: record the box's sustained (not burst) throughput at
commissioning, in summer conditions if you can get them; alert on *sustained deviation
from that baseline*, not just on temperature thresholds; and treat a slow drift in
response times as a maintenance signal like any other vibration trend. The failure mode
is not "the AI got worse." It is dust on a filter, and the fix is a shop vacuum, not a
retraining run.

## The spare on the shelf

Plants keep spare drives, spare cards, spare pumps. A model deployment is the rare
computer system where the spare-parts mentality transfers almost perfectly, because
Chapter 3's operational virtues made the system *copyable*: the entire deployed
intelligence is a weights file with a checksum, a configuration directory, and a service
definition. A cold standby is therefore not a project — it is a second box with the
same three things on it, health-checked monthly by the same end-to-end probe, powered
off the rest of the time. When the primary dies, recovery is a cable and a DNS entry.

Notice what makes this possible: everything that matters is a *file*. No license server
to reactivate, no cloud enrollment to re-authenticate, no vendor account whose owner
left the company. The restore test is the same as the deployment test. Run it before
you need it — the spare that has never served a request is one more piece of folklore —
and version the spare's contents in lockstep with the primary, because a standby
carrying last quarter's model answers last quarter's questions. The gap between "we
have a spare" and "we have a *tested* spare at the current version" is exactly the gap
between this chapter's two crash durations, wearing different clothes.

## The morning-after report

The practice that converted our ninety-minute crash into a twenty-five-minute crash was
not a technology. It was the report written the same day: what failed, in what order,
what the recovery actually required minute by minute, which fixes would have prevented
each minute, and — the part most postmortems omit — which *checks gave misleading
answers* during the recovery. The self-matching process check earned its place in this
chapter because a same-day report recorded it as a near-miss instead of letting it
evaporate into "anyway, it worked out."

Two crashes make a trend line only if the first one produced a document. Keep the
reports blameless (the interesting failures are systemic, and a crew that fears the
report hides the timeline), keep them short enough to be written the same day, and end
each with work orders, not recommendations. "Consider improving mount reliability" fixed
nothing; the line item "repair fstab entries for all three data volumes" is why crash
two skipped storage entirely. The report is the mechanism by which an outage becomes an
asset; skip it, and you have simply paid for the same lesson twice at full price.

## The recovery drill

Assemble the sections into the procedure this chapter exists to leave behind:

1. **Enumerate volumes; know each one's power-loss behavior.** Journaled, auto-mounted,
   repair procedure written down. The right time to learn a filesystem's failure mode is
   never during the failure.
2. **Make every service's restart policy explicit** — including restart *order* and the
   conditions under which something must NOT start. Interlocks by marker file beat
   intentions by checklist.
3. **Set checkpoint cadence by redo-tolerance,** and rehearse one restore per cadence
   change. Unverified restores are folklore.
4. **Write artifact-based health checks** for everything that matters: output freshness,
   health endpoints, token generation — never process-table appearances.
5. **Time a full cold recovery, on purpose, quarterly.** Ours went from ninety minutes
   to twenty-five because the first crash's report became a work order. The drill is how
   you buy that improvement without needing the storm.

None of this is glamorous, which is the point. The AI parts of this book — grounding,
abstention, evaluation — decide whether the system is worth trusting. This chapter
decides whether it is *there* on the morning the substation hiccups. A model that is
brilliant but absent loses to one that is adequate and running; the plant floor has
always graded on attendance, and it is right to.
Build for the morning after the storm, and the storm becomes a line item in a report
instead of a story people tell about the time the plant tried AI — because the
difference between those two outcomes was never the model. It was the fstab, the marker
file, and the report somebody wrote the same day.
