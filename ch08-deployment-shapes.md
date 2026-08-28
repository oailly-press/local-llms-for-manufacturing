# Chapter 8 — Deployment Shapes

*(draft v0, 2026-08-27 — written by Claude Fable 5, unverified. `[R-TBD]` marks numbers
awaiting lab entries.)*

The pipeline exists (Chapter 4), the model is honest (Chapter 5), the gate is armed
(Chapter 6), maybe the weights are yours (Chapter 7). What remains is the question a
plant asks about every system it adopts: what does this thing physically look like,
where does it sit on my network, and who restarts it at 2 AM? This chapter is the
shapes catalog — the handful of deployment topologies that actually occur, with the
configuration traps we have collected the honest way, by falling into them.

## The parts list, one more time

Chapter 1 previewed the stack; here it is as a deployment bill of materials. **The
weights file:** one large file, checksummed, versioned like firmware. **The inference
engine:** open-source serving software that loads the weights and speaks HTTP — the
open engines are mature, actively maintained, and run everything this book discusses;
our lab's production has run on one for its entire life `[R-TBD: engine/config
lineage]`. **The service wrapper:** an init-system unit with restart policy, Chapter
9's condition gates, and a log destination. **The gateway:** the small application
layer that owns Chapter 4's rendering and contracts — the only custom software in the
building. **The gate rig:** Chapter 6's runner, on the same box or beside it. Nothing
else. No orchestration cluster, no vendor appliance, no subscription.

## Shape one: the sidecar box

The starter shape, and for many floors the final one: a single industrial PC or
workstation beside the line, running one small model (Chapter 3's pocket or line-side
class), serving one or two applications — the work-order extractor, the fault-code
assistant. Everything on one box: engine, gateway, logs, gate rig.

Its virtues are the small-model virtues from Chapter 3 made physical: it restarts in
seconds, its spare is a copy (Chapter 9), and its blast radius is one line's
convenience features. Its limit is concurrency and ambition — one box serves a work
cell, not a plant. Configuration notes that recur at this shape: pin the model file
read-only; give the KV cache explicit headroom for your worst shift-change burst
rather than letting defaults decide (Chapter 3's cache-exhaustion story); and log
every request/response pair locally with rotation, because this box is also the corpus
collector (Chapter 7) and its disk fills on the schedule of its usefulness.

## Shape two: the department server

One GPU box (the engineer's-assistant class, sometimes two models resident — the
two-model pattern under one roof), serving a department over the plant network:
maintenance, reliability, and the control engineers all hitting one endpoint through
their own thin clients. This is the shape where serving stops being an appliance and
becomes a service, and three disciplines arrive with it.

**Admission and identity.** The endpoint answers only authenticated clients, and every
request carries who asked — not for surveillance (Chapter 7's warning stands) but
because dispositions, corpus clearance, and audit all key off it. A reverse proxy in
front of the engine, doing TLS and tokens, is an afternoon of setup and the difference
between a service and an open socket.

**Queueing honesty.** Batch serving (Chapter 3) means throughput scales well — until
the memory ceiling, where the honest failure is a fast "server busy, retry" rather
than a hung request. Configure explicit concurrency limits below the measured ceiling,
and surface queue depth as a metric; the day it trends upward is the day you size the
next shape `[R-TBD: concurrency ceiling measurements]`.

**Version discipline.** With many clients, "which model answered this?" must be
recorded, not remembered. The engine's version string, the weights checksum, the
contract version, and the schema version ride along in every response's metadata.
Chapter 6's gate reports already pin these; production echoes them.

## Shape three: the hierarchy

The full two-model pattern at plant scale: small always-on models at the line
(sidecar shape), escalating to the department server, escalating — where policy
allows — to a human queue or, in hybrid plants, an external frontier model for the
rare case that earns it. Each hop is Chapter 5's abstention machinery working as
routing; each hop is logged with its reason; and the traffic *down* the hierarchy is
Chapter 7's escalation-teacher loop returning trained improvements to the edge.

The hierarchy's design rule: **capability flows down, data flows up, and neither
crosses a boundary silently.** If an external model participates at the top, the
boundary is explicit — which renderings may leave the building (clearance flags,
again), stripped of what must not, logged as having left. A plant that cannot draw
that line cleanly should cap its hierarchy at the walls; the whole premise of this
book is that the local ceiling is high enough to be useful.

## Shape four: the air gap

Some floors do not negotiate: no route exists between the process network and anything
that touches the internet, and the model deployment lives entirely inside the wall.
The good news is structural — this book's whole stack was designed offline-first, so
the air-gapped shape is mostly the sidecar or department shape with its update path
made explicit rather than assumed.

The update path is the design problem. Weights, engine builds, and document-store
additions arrive by controlled media on a schedule: checksum manifest first, files
second, gate run third — nothing serves until the offline gate rig passes it, which
makes the gate the border checkpoint it always should have been. The gate set itself
updates by the same ceremony in the other direction: fresh cases go out (clearance
flags enforced at the boundary, per Chapter 7), labels come back. Plan the cadence by
the drift trigger of Chapter 6 — monthly is common — and resist emergency exceptions;
an air gap with a hurry-up path is a decorative air gap.

One capability deserves explicit celebration in this shape: everything still works.
No license server phones home, no model degrades for want of a subscription check, no
document leaves. The plants with the strictest walls are, not coincidentally, the
plants this book's local-first argument was written for.

## Sizing the box: the buyer's worksheet

Procurement wants a specification, not a philosophy. The worksheet, in order:

1. **Model memory:** parameters × bytes-per-weight at your chosen quantization, from
   Chapter 3's arithmetic, plus engine overhead.
2. **Cache memory:** worst-case concurrent contexts × per-token cache cost at your
   window size — the shift-change number, not the average. Cache headroom is the line
   item most sizing exercises omit and Chapter 3's ceiling story is what it costs.
3. **Disk:** weights (twice — current and previous version, because rollback is a
   copy), the document store, and the request logs at your retention policy. Logs
   dominate within a year on a busy box.
4. **Thermals and power:** sustained-load rating for the enclosure it will actually
   live in, per Chapter 9's heat section — the mezzanine in August, not the lab in
   October. A box that throttles is a box that silently fails its latency budget.
5. **The spare:** the same line again, at cold-standby price.

Sum it, round up one hardware notch — the marginal cost of headroom at purchase time
is a fraction of the cost of discovering its absence — and staple the worksheet to
the deployment record. When the numbers came from measurements (the gate's throughput
probes, the cache ceiling test), say so on the sheet; procurement respects an
instrumented number and audits remember one `[R-TBD: reference sizing worksheet]`.

## The security posture, stated plainly

A language-model service is one more networked application, and most of its security
is the boring kind you already practice: network segmentation (the service sits with
the other supervisory-level systems, never on the control network itself), least
privilege (the gateway reads the historian; nothing writes toward a controller —
Chapter 10 makes this a hard rule), authenticated clients, and logs that answer who
asked what and when.

Three items are model-specific enough to name. **The injection surface** — Chapter
4's adversarial note operationalized: plant text is untrusted input; fencing, quoting
contracts, and constrained outputs are the mitigations, and the residual risk is
bounded by the human between verdicts and actions. **The exfiltration surface** — in
any shape with an external hop, the prompt itself is an egress channel; clearance
flags and boundary logging are the controls, and the air-gapped shape is the plant's
statement that the channel does not exist. **The supply chain** — weights and engine
builds come from named sources, verified by checksum at download and again at load;
a model file is executable-adjacent content and deserves the same custody chain as
firmware. None of this is exotic; all of it belongs in the same security review the
plant already runs, using the same vocabulary, which is exactly how it will get
approved.

## The container question

Someone will ask why this chapter says "unit file" instead of "container," and the
answer is a default, not a doctrine. A floor deployment is one engine on one box with
one large file: the isolation containers buy solves problems this shape does not
have, while adding a runtime, an image registry, and a build pipeline to the parts
list. Files, checksums, and an init system are the plant's native idiom — Chapter 9's
recovery drills are written in it — and fewer layers is its own reliability feature.
Containers earn their place at the department shape and above when a team already
operates them fluently, or when one box must host several isolated gateways; even
then, the weights file stays a mounted artifact with its own checksum, never baked
into an image, so that model versioning remains Chapter 7's six-artifact discipline
rather than an image tag. Use what your crew can fix at 2 AM. That rule outranks
every architecture opinion in this section.

## The configuration traps, with the scars attached

Serving configuration looks like a page of harmless flags. Some of them are load-bearing
in ways the documentation undersells, and our lab's log is a small museum of the
failure modes. The exhibits:

**Precision of the attention cache.** Quantizing the KV cache saves real memory and is
usually safe — until a specific model dislikes it. Our production model, quantized to
an 8-bit cache, produced corrupted output; the cache went back to 16-bit and stayed
there, at measured memory cost, by standing rule `[LAB: CLAUDE.md serving traps —
KV-cache precision corrupting output]`. The general lesson: cache precision is a
*gated* change like any other, not a free flag. Flip it only with Chapter 6 watching.

**Memory-mapping vs. loading.** Engines can map the weights file from disk (fast
start, gentler on RAM, first-touch latency) or load it whole (slow start, no
surprises, but the file must fit). Forcing a full load of a file larger than memory
does not politely fail — it takes the host down with it, a lesson our lab's hardware
notes preserve in the imperative mood `[LAB: CLAUDE.md serving traps — no-mmap on
oversized files]`. Know which mode each unit file requests, and why.

**Layer placement on mixed hardware.** When a model splits across GPU and CPU memory,
*which* layers spill decides the speed — and some placement flags silently interact
with tensor-layout optimizations, producing configurations that run at a fraction of
their potential until one line in the load log explains why (Chapter 9's
read-the-load-log rule, which was earned on exactly this class of mystery). The
protocol: after any placement change, read the load log's placement summary in full,
then re-run the gate's throughput probe before calling it done `[R-TBD: placement
config matrix]`.

**The restart that isn't clean.** A serving process that dies mid-batch can leave the
GPU in a state where the successor process fails to allocate. The watchdog's restart
path must handle "restart the process" and "reset the device" as distinct rungs, and
Chapter 9's function-probing health check is what distinguishes them: process up but
no token back means climb the rung.

## The integration seams

A model service earns its keep only where people already work, which means three
unglamorous adapters, worth naming because their absence is the most common reason a
technically sound deployment goes unused.

**The HMI seam.** The operator-facing panel gets a read-only card: the watcher's
latest digest (Chapter 4), the current verdict for this line's active alarm, and
nothing type-in-able. Operators consume; the interaction surface lives with
maintenance.

**The CMMS seam.** The extractor writes *draft* work orders, flagged as
machine-drafted, into the normal approval queue — never directly into the record.
Chapter 5's confidence grades map onto the queue's priorities; the technician's
accept-or-correct is the disposition that feeds Chapter 7.

**The conversation seam.** The engineers' ad-hoc questions go through a chat surface
that is a thin skin over the same gateway — same contracts, same schemas underneath,
same logging. The moment a "quick chat tool" bypasses the gateway, everything this
book built — grounding, abstention arms, quote checks, corpus capture — silently
stops applying to the traffic that is fastest growing. One gateway, many skins, no
exceptions.

## The rollout path: shadow, advisory, assisted

Whatever the shape, it goes live in stages, and the stages are the same everywhere
because they are stages of *trust*, not of technology.

**Shadow mode first.** The full pipeline runs on live traffic — rendering, verdicts,
logging — and nobody sees the output but the gate. Two to four weeks of shadow
answers, scored against what the humans actually did, is the cheapest large-scale
evaluation you will ever run: production distribution, production load, zero
production risk. Shadow mode also burns in the operational layer — the watchdogs,
the log rotation, the thermal reality — while the stakes are nil. Most deployments
discover their first three surprises here, which is the point `[R-TBD: shadow-phase
findings from lab deployment]`.

**Advisory next.** Outputs become visible, clearly badged as machine drafts, with
Chapter 5's confidence grades and disposition buttons live. The crew's corrections
start flowing (Chapter 7's corpus), the usefulness metrics start accumulating
(Chapter 6's disposition stream), and the deployment earns or fails to earn its place
in the daily rhythm. Hold here until the dispositions say the tool is being *used* —
green gates with ignored output is Chapter 6's warning, not a graduation.

**Assisted, at most.** The ceiling of this book's ambition: machine drafts that
humans approve, machine routing that humans can override, machine summaries that
humans verify before acting. The stage that does not exist on this path is the one
where the model acts on the process unmediated — not because the models will never be
good enough, but because Chapter 10's checklist requires a person between a
probabilistic component and a physical consequence, and this book means it. The
rollout path's finish line is a tool the crew reaches for without being told to,
inside a boundary everyone can state from memory. That is what "deployed" means here.

## The deployment record

Every shape ships with a one-page record, kept where the runbooks live: the shape
name; the parts list with versions and checksums; the unit files and their conditions;
the health probes and their thresholds; the gate report that authorized this
configuration; the spare's location and last restore test; and the names — who owns
the service, who owns the gate, who signs the next change. One page. Chapter 9 already
argued that the difference between a ninety-minute recovery and a twenty-five-minute
one is a document somebody wrote; this is that document, written before the storm
instead of after. The next chapter — the last — compresses this book's whole argument
into the checklist that page belongs to.
Whatever shape you choose, choose it out loud: the record, the names, the boundary. A
deployment nobody can describe is a deployment nobody can defend at budget time.
