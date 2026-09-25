# The Serventis Specification — Design Rationale

**Companion to SPEC.md Version 3.6.0** **Copyright © 2025–2026 William David Louth / Humainary**

---

This document is **non-normative**. It explains why the Serventis Specification is shaped as it is,
and records the alternatives that were considered and rejected. Where this document and SPEC.md
disagree, SPEC.md governs.

The specification's theoretical foundation — the semiotic argument, its relation to Peircean sign
theory, Lotman's semiosphere, and the Viable System Model — is developed at length in `SERVENTIS.md`
in the reference projection. This document addresses the engineering consequences.

---

## 1. Why the Specification Stops Where It Does

The first question a reader of SPEC.md is likely to ask is why it says so little about the thing
Serventis is *for*. It defines signs and vocabularies at length, then declines to say what
`Locks.GRANT` indicates, declines to require the property maps the Java projection publishes, and
puts its two ascent operators in a non-normative appendix. That restraint is the most consequential
decision in the document.

### Three Layers, One Specification

A working Serventis system involves three separable things (SPEC.md §1.2).

The **language** is structural: what a sign is, what a dimension is, that vocabularies are finite
and fixed, that a signal is a qualified sign, that translation is partial and abstention means
something. These are facts about the model. Two independent implementations that disagree about any
of them are not implementing the same thing.

The **policy** is interpretive: that `Locks.GRANT` reads `STABLE`, that `Tasks.SUBMIT` opens an
episode, that `Breakers.HALF_OPEN` reports an outcome rather than an act. These are judgments.
Careful people can disagree about them, and a consumer that reads `GRANT` differently is not
misunderstanding the language — it is holding a different opinion within it.

The **affordances** are executional: ordinal-addressed arrays, precomputed signal spaces,
constant-time lookup, mapping functions applied once at construction. These are how one runtime
makes the model cheap. The Java projection depends on them; another runtime might find them
actively wrong.

Only the first is specified normatively.

### Why Not Standardize the Policy

The case for standardizing the readings is real: a consumer that could rely on `GRANT → STABLE`
everywhere could interpret a vocabulary it had never seen. That is genuinely valuable, and it is why
the readings are documented rather than omitted.

It was rejected for now on evidence, not principle. Every reading in the registry was authored
against a single projection, and there is no second implementation to have disagreed with any of
them. Standardizing them would freeze one set of opinions at exactly the moment when the least is
known about which of them generalize — and unfreezing a normative reading later is far harder than
promoting a documented one.

There is a structural argument too. A reading is a claim about what a sign *indicates*, and what a
sign indicates depends on what the consumer is deciding. `Caches.MISS` reads `DEGRADED` for a
consumer watching hit rate and reads nothing at all for one watching correctness. A specification
that fixes the reading has quietly fixed the question, and the questions are not the specification's
to fix.

### Why Not Standardize the Affordances

`SignMap` is a good design. It is also a design whose whole value is that Java enums have dense
ordinals and arrays are cheap. A Rust projection would likely use a `match`; a Go projection might
use a generated switch; a projection over a vocabulary of three signs should probably use neither. A
normative requirement for constant-time precomputed lookup would impose Java's cost model on
languages with different ones, in exchange for nothing a consumer can observe.

The observable requirement is that a sign's reading is stable and cheap enough to consult per
observation. That is a quality-of-implementation matter, and SPEC.md leaves it as one.

### What Was Actually Lost

Very little, and it is recoverable. Everything demoted is in Appendix B, described in enough detail
to implement — the property maps, both sequencers, the tally, the recurrence operator, and the
constants the reference projection tunes them with. A projection is free to adopt all of it, and
SPEC.md §B.7 says what promotion into the core would require: two independent projections, agreement
on observable behavior rather than surface, and a demonstrated need for identical results.

The gain is that SPEC.md now says only things that are true of Serventis rather than things that are
true of its first implementation. A specification that conflates the two teaches every subsequent
projection to copy accidents.

---

## 2. Why Bounded Vocabularies

### The Problem with Open Ontologies

The dominant observability model is open: a producer emits a name and a value, and the set of names
is whatever anyone has ever emitted. This is maximally flexible and it is why the model is
universally adopted. It is also why nothing downstream can be decided cheaply.

An open vocabulary forecloses three things at once. Membership cannot be checked, so a typo becomes
a new metric rather than an error. Coverage cannot be checked, so nobody can say whether an
interpretation handles everything a producer might emit. And exhaustiveness cannot be enforced, so
adding a case to a producer silently leaves every consumer's interpretation incomplete.

### The Serventis Position

Serventis fixes the alphabet at publication. A sign set is finite, densely indexed, and closed
(SPEC.md §4.5). What that buys is the ability to ask total questions: does this interpretation
handle every sign? is this classification complete? has this consumer seen the whole alphabet? Each
has an answer, and none of them does under an open vocabulary.

It also makes cheap implementations *available* without requiring them. Because a vocabulary's size
and the size of its signal space are both known in advance, a projection may precompute lookup
tables and intern the signal space. SPEC.md §4.6 says the space is finite and that interning is
invisible to the model; it does not say to intern. The boundedness is the normative part, and the
optimization is what boundedness permits.

### What This Costs

A vocabulary cannot grow at runtime. A producer that discovers a new kind of event cannot invent a
sign for it; someone publishes a new version of the vocabulary (SPEC.md §8.3).

This was accepted deliberately. The alternative gives back every property above, permanently,
because a consumer can never again assume it has seen the whole alphabet. A vocabulary that changes
on a release cycle rather than at runtime is not a serious constraint on the subjects Serventis
describes: locks, caches, transactions, and services have had stable vocabularies for decades.

---

## 3. Why Implementer-Observable Signs

### The Inference Trap

The tempting design is to let producers emit conclusions: a cache that emits `THRASHING`, a service
that emits `OVERLOADED`, a lock that emits `CONTENDED`. These read well and they are almost always
wrong.

They are wrong because they require inference the producer is badly positioned to perform.
`THRASHING` is a statement about hit rate over a window, and the cache knows only about this lookup.
`OVERLOADED` is a comparison against capacity the service does not know. `CONTENDED` is a rate, and
the lock sees one acquisition. To emit these honestly a producer would need to accumulate history,
hold thresholds, and compare — so every producer carries a small, untested, uncoordinated analytics
engine, and the thresholds live in the least accessible place in the system.

They are also wrong because the inference is not recoverable. Once the cache emits `THRASHING`, the
evidence that produced it is gone. A consumer that would have drawn a different conclusion — because
it knows this is a warm-up cache, or because it is comparing against a peer — cannot, because it
never saw the lookups.

### The Split, and Its Exemption

Serventis splits the two jobs at the emission boundary (SPEC.md §8.1.1). The producer reports what
it directly observed: `LOOKUP`, `HIT`, `MISS`, `EVICT`. Recognition of pattern happens downstream,
where history, thresholds, and cross-subject comparison are available and where policy can change
without touching the producer.

The rule is mechanical enough to apply in review: if selecting any component of an observation
requires the producer to remember previous emissions, compare against a reference, or weigh
evidence, the vocabulary is *derived*. That is a classification, not a verdict — `Cycles` is derived
and perfectly well-formed. It is a defect only in a vocabulary claiming to be primary, which is
where the rule bites.

But the rule cannot apply to *every* vocabulary, and an earlier draft that said it did was
incoherent. `Statuses`, `Situations`, `Trends`, `Systems`, `Surveys`, and `Cycles` exist precisely
to carry conclusions; a `Sensor` compares against a configured reference by definition; a `Log`
severity is a classification. Requiring these to be directly observable would forbid the
vocabularies the architecture is built to ascend into.

So SPEC.md §8.1.1 splits vocabularies into **primary** and **derived**, and applies the rule only to
the first. The line is drawn from the vocabulary's own sets — whether selecting an observation, in
every component, needs anything beyond the moment of emission — rather than from who happens to hold
the instrument. Derived is the complement of primary, so the two are exhaustive: a vocabulary whose
signs are directly observed but whose dimension must be computed is derived, one component being
enough to decide it.

"Derived" is therefore broader than "interpretive". A recurrence dimension is a plain fact about the
stream and involves no judgment, but the producer must remember what it already emitted, so a
vocabulary carrying one is derived all the same. An earlier draft defined it by the
producer-to-subject relationship, which could not survive contact with conformance: the same
`Outcomes` instrument may be held by a subject reporting itself or by an assessor watching something
else, so no reviewer could tell from a vocabulary's name which rule applied to it. Defined
intrinsically, it is decidable when the vocabulary is published, which is when it needs deciding.

The exemption stays deliberately narrow, because it is exactly the hole through which the inference
trap returns.

### The Consequence for Vocabulary Design

This is why the primary vocabularies look as they do. `Caches` has `HIT` and `MISS` rather than
`HIT_RATE_LOW`. `Locks` has `CONTEST` — a compare-and-swap that lost, which the lock genuinely
observes — rather than `CONTENTION_HIGH`. `Atomics` has `SPIN`, `YIELD`, `BACKOFF`, and `PARK`, four
things the implementation actually did, rather than one `SLOW` that would have required it to judge.

---

## 4. Why Partial Translation

### Totality Manufactures Meaning

The obvious design for a translation into `Statuses` is a total function: every sign gets a status.
It is simpler to specify, simpler to implement, and it removes the absent value from the model.

It is also a lie. What status does a lock `RELEASE` indicate? None: a release is neither good nor
bad, it is what happens after a grant. A total translation would have to answer `STABLE`, and a
consumer weighing statuses would then read a stream of releases as positive evidence of health — so
a component that acquires and releases a lock in a tight loop while failing at everything else reads
as healthy.

### Abstention as a First-Class Answer

So translation is partial (SPEC.md §5.1), and abstention is a real answer, distinct from every
reading in the target vocabulary. A consumer skips abstaining signs rather than defaulting them,
which means a stream of releases contributes no evidence at all — exactly right, because it *is* no
evidence.

This is why SPEC.md §5.1 is emphatic that abstention is not `UNKNOWN`. The two are easy to conflate
and mean opposite things. `UNKNOWN` is verdict-bearing: something was decided and the decision was
indeterminate — a lock contention that resolved neither way. Abstention is the absence of a decision
— a lock release, which is not the kind of thing that decides anything. Collapsing them makes "we
tried and could not tell" indistinguishable from "we were not asking", which destroys any confidence
measure computed downstream.

### Why the No-Default Rule Is Scoped

An earlier draft banned defaulting an abstaining sign outright, and that ban was too broad. It
contradicted the bracket sequencer, where an episode that closes on an abstaining sign still reads
`STABLE` — and it should, because the reading comes from the structural fact that an episode
*completed*, not from the closing sign's own meaning. Six registered vocabularies close on
abstaining signs, so this is routine rather than a corner case.

SPEC.md §5.1 therefore scopes the rule to reading a sign **in isolation**. Where the evidence is
structural — position in a sequence, the shape of a trace — the conclusion never came from the
translation, so the translation's abstention neither supplies nor forbids it. This is the same
distinction §5 draws between the two kinds of sign, seen from the consumer's side.

---

## 5. Why the Operation/Outcome Distinction Stayed

Of everything demoted to Appendix B, the operation/outcome distinction is the one that stayed in the
normative core, and the split is worth explaining: the **concept** is normative, the
**classification** is not.

### The Concept Is Structural

Outcomes and operations tend to carry evidence differently, and that tendency is what makes partial
translation intelligible.

An outcome is usually self-contained. One `DENY` supports a reading on its own: something was
refused. An operation usually is not. One `CALL` supports no reading — calls are what services do.
The information is in what happened *next*, or in what conspicuously did not: a `CALL` followed by
`SUCCESS` is unremarkable, while a `CALL` followed by another `CALL` on the same subject means the
first never completed. Counting `CALL`s tells you throughput and nothing about health.

Remove the concept and SPEC.md §5 cannot explain its own structure — partial translation would look
like an unexplained gap rather than the expected consequence of much of a vocabulary carrying no
isolated reading.

### But the Concept Does Not Route

An earlier draft went a step further and made the kind *select* the ascent mode: outcomes read
individually, operations read structurally. That was wrong, and the registry disproves it.

Nine registered vocabularies classify a sign as an operation and still give it a status reading.
`Leases.REVOKE` reads `DEFECTIVE`, `Valves.DROP` reads `DEFECTIVE`, and `Processes.KILL` reads
`DEGRADED`, because an act can itself be the verdict of something already gone wrong.
`Queues.ENQUEUE` reads `STABLE`, because an enqueue the queue accepted is itself the success. Five
classify a sign as an outcome and place it mid-episode, where its position is what matters:
`Locks.GRANT` and `Resources.GRANT` advance an acquisition rather than ending it, and `Atomics.FAIL`
will be retried. The reference tally reads any non-abstaining status without consulting the kind at
all.

So SPEC.md §5.2 keeps the grammatical distinction and explicitly denies that it determines
interpretation, and §5.3 describes the two ascent modes by what supplies their evidence — the sign
itself, or the sign's position among others — with either kind free to feed either mode. The
correlation is real and worth stating; it is a tendency, not a routing table, and writing it as one
would have contradicted nine vocabularies in the same repository.

### The Classification Is Opinion

Whether `Breakers.HALF_OPEN` is an act or a result, though, is a judgment. It is defensible either
way — the breaker observed its timer expire, or the breaker moved itself to a trial state — and
nothing breaks if a projection decides differently. So the per-vocabulary `KIND` maps went to
Appendix B.2 with the other policy.

### Why Kind and Status Stay Independent

The two classifications nearly coincide: outcomes carry status readings, operations abstain. The
near-miss is the interesting part, and it runs both ways. `Leases.REVOKE` is an operation that reads
`DEFECTIVE`, because a forced revocation is itself the verdict of something already gone wrong;
`Valves.DROP` is the same shape. Conversely `Breakers.HALF_OPEN` is an outcome that abstains,
because being in a trial state indicates nothing on its own.

So SPEC.md §5.2 forbids deriving either from the other. `KIND` asks what a sign *is*; a status
reading asks what it *indicates*; different questions, answered separately, and the seams between
them are where the informative signs live.

Keeping them separate also keeps the kind stable. Because it is grammatical rather than evaluative,
it does not change when someone revises their opinion about what a `RECOURSE` means for health.

### Set-Relativity

The kind belongs to a sign *within its set*, never to its textual name (its lexeme; SPEC.md §5.2).
`STOP` in `Processes` is an outcome — the process exited — while a `STOP` bracketing a shutdown
sequence would be an operation, the act of stopping. Both are correct. This is the same phenomenon
as SPEC.md §8.4's homographs seen from the property side, and it is why a projection must never
build a global lexeme-to-kind table.

---

## 6. Why Status Ordering Is Policy

The seven `Statuses` signs invite ordering: `STABLE` is better than `DEGRADED`, which is better than
`DOWN`. It would be convenient — reducing several readings would be a maximum.

But the set mixes three axes (SPEC.md §7.1). `CONVERGING` and `DIVERGING` describe *trajectory*;
`STABLE`, `DEGRADED`, `DEFECTIVE`, and `DOWN` describe *level*; `ERRATIC` describes *variance*.
There is no correct answer to whether `DIVERGING` outranks `DEGRADED` — a subject that is degraded
and holding steady and one that is fine but deteriorating are differently alarming, and which
matters more depends on what you are about to do.

Serventis keeps the three axes in one set because they are what an observer needs to say. The
vocabulary therefore defines no intrinsic ordering or combination rule. A projection may impose one
as policy, but it may not present that policy as an order inherent in the sign set.

### Reduction, and What Is Not Reduction

An earlier draft required *every* reduction of several readings to preserve severity, so that `DOWN`
could never be outvoted. That was wrong twice over.

It was wrong about the tally, which legitimately outvotes a minority. A `DOWN` among a majority of
`STABLE` loses, because "the evidence favors stable" is exactly what a tally is for. A consumer
that must not lose a severe reading should not be aggregating it in the first place.

And it was wrong about the sequencers, which a second draft then wrongly cast as the
severity-preserving alternative. They are not a reduction at all. The bracket sequencer holds one
bit — whether an episode is open — and maps the single `STATUS` value of the sign it is admitting
onto the reading it emits for that admission. It never combines readings, so there is nothing for it
to mask: a `DOWN` advance followed by a clean close emits `DEFECTIVE` and then `STABLE`, two
readings in sequence, and a consumer that needs the first to outlive the second must hold onto it.

So SPEC.md §7.1 requires an implementation not to treat the sign set as defining a lattice, total
order, or reduction rule, and to document the policy where it genuinely reduces several readings to
one.
Mapping a single value onto another is normalization, not reduction, and the requirement does not
reach it.

---

## 7. Why Dimensions Are Perspective, Not Location

### The Attribute Temptation

A dimension looks like a general-purpose attribute slot, and the pull to use it that way is strong:
which host, which region, which shard, which tenant. Resisting it is SPEC.md §8.1.5.

These are not properties of the *observation*; they are properties of the *subject*, and Substrates
already has a system for subject identity — hierarchical interned names attached to every component.
Putting location in a dimension duplicates that system badly: it multiplies the signal space by an
unbounded cardinality, it varies per deployment rather than per vocabulary (so the set is no longer
fixed), and it makes two emissions from the same subject look like different observations.

### What a Dimension Is For

A dimension changes what the sign *means*. `CALL × CALLER` and `CALL × CALLEE` are genuinely
different observations of the same event from its two ends. `FAIL × CORRECTNESS` and `FAIL × SAFETY`
are different failures. `STABLE × TENTATIVE` and `STABLE × CONFIRMED` are the same reading offered
with different warrant.

The compact test is perspective, not location. The longer form: if two subjects would emit the same
sign under different dimension values purely because of where they are, it is not a dimension. If
the *same* subject would emit the same sign under different dimension values depending on which
aspect it is reporting, it is.

### Why Most Vocabularies Have None

Most registered vocabularies have no dimension set at all. This is the expected result of applying
the test honestly: a queue enqueues, and there is no second viewpoint on that. The dimensioned
vocabularies are the ones with genuine dyadic structure (`CALLER`/`CALLEE`, `LESSOR`/`LESSEE`,
`PROMISER`/`PROMISEE`) or a genuine qualifying axis (Evals' criteria, Statuses' confidence).

---

## 8. Why Vocabularies Are Fixed but Optional

### The Tiering Problem

Two goals conflict. Interoperability wants a consumer to write one interpretation of `Locks` that
works against any projection. Practicality wants a projection to ship without implementing
thirty-six vocabularies.

Requiring everything makes the conformance bar absurd for a new-language projection. Requiring
nothing makes the names worthless — a consumer seeing `Locks` would have to ask which `Locks`.

### The Resolution

SPEC.md §8.2 tiers them. Universal vocabularies are required because they are the shared language
across domains and projections; a projection missing one cannot participate in ascent at all.
Registered domain vocabularies are optional to provide but fixed once provided — ship none, or ship
`Locks` with exactly its registered signs, but do not ship something called `Locks` with eight of
them.

The obligation covers membership, meanings, and the primary/derived classification. Classification
belongs there because it decides whether the implementer-observable requirement applies at all: a
projection that reclassified a registered vocabulary would be claiming a different obligation under
the same name. The obligation stops at interpretive policy — a projection that publishes different
readings for `Locks` is still shipping `Locks`, because the signs, what they report, and whether
they are observed or derived are the same; only the opinions about them differ. That is the tiering
of §1.2 applied to registration, and it is what lets the registry document policy without binding
it.

A minimal projection is therefore the observation model plus six small vocabularies and two
vocabulary templates. A projection that does ship `Locks` is automatically legible to every consumer
written against `Locks` anywhere.

---

## 9. Why Homographs Are Left Alone

The registry records lexemes appearing in several vocabularies meaning different things: `CONTRACT`
is agreement in `Exchanges` and shrinkage in `Pools`; `PROBE` is a trial request in `Breakers` and a
liveness check in `Leases`; `RELEASE` is a barrier opening in `Latches` and a holder relinquishing
in `Locks`.

The obvious tidying — rename until every lexeme is globally unique — was rejected. It would produce
worse vocabularies. `CONTRACT` is the right word in `Exchanges` and the right word in `Pools`;
forcing one to say `SHRINK` or `AGREE` makes that vocabulary read worse to serve a global property
nobody consuming a single vocabulary benefits from.

The cost of leaving them is bounded, because SPEC.md §4.2 already establishes that a sign has no
meaning independent of its set. A consumer resolves a sign against the vocabulary that produced it,
which it must do anyway. SPEC.md §8.4 makes the rule explicit; the registry lists the known cases so
nobody builds a cross-vocabulary lexeme table by accident.

The same argument covers the deeper variation: `FAIL` in `Atomics` is mid-episode — a
compare-and-swap that will be retried — while `FAIL` in `Processes` is terminal. That is not even a
naming collision; it is the same word, correctly used, with different structure behind it. No
renaming would fix it, because the difference is in the domain, not the word.

---

## 10. Why Time Stays Outside Sequence Recognition

Appendix B's sequencers recognize shape and refuse to recognize duration. "This episode has been
open for thirty seconds" is exactly what one wants from them, and they do not do it.

Three reasons. A recognizer that consults a clock is no longer deterministic — the same trace
replayed gives different readings, forfeiting the replay property Substrates exists to provide.
Timeout policy is deployment-specific and changes far more often than trace structure, so binding
them into one operator couples a stable thing to a volatile one. And it is unnecessary: a producer
that can observe a timeout already expresses it as a sign — `Locks.TIMEOUT`, `Tasks.TIMEOUT`,
`Latches.TIMEOUT` are all registered — so the temporal judgment enters the trace as ordinary
sequence shape.

Where no producer can observe the timeout, an external watchdog emits into the same stream and the
result is identical. The judgment stays visible, in a component whose job it is, rather than hidden
in a recognizer's internal timers.

Related reasoning explains why status-to-situation ascent is not specified (SPEC.md §7.2, §B.6).
Persistence is the obvious evidence for it, and an earlier draft went as far as requiring that
ascent to be time-based — which was itself a policy leak, since confidence, correlation across
subjects, or the cost of acting are all defensible bases and the specification has no standing to
pick among them. What the specification can say is that a situation reading depends on
considerations the observation stream does not carry, so it names the vocabulary and constrains
neither the evidence nor the mechanism.

---

## 11. Why Serventis Adds No Runtime

Serventis defines no queue, no thread, no ordering, no lifecycle. SPEC.md §1.3 makes the dependency
on Substrates total and one-directional, and the instrument contract (SPEC.md §6.5) forbids an
instrument from buffering, coalescing, reordering, or accumulating anything.

The pressure to add runtime is constant, because each addition looks locally reasonable: an
instrument that batches emissions, one that suppresses duplicates, one that keeps a count. Each
would silently break a Substrates guarantee — batching breaks ordering, suppression breaks the
run-length operators downstream, counting makes an instrument stateful and therefore unsafe to share
by name.

Keeping the layer thin is what makes it composable. Every Substrates operator works on a stream of
observations without knowing anything about the observation model, because an observation is just a
value. And a projection implementing Serventis has to implement vocabularies — not a scheduler.

---

## 12. Terminology

### Why These Names

**Sign** and **dimension** come from semiotics, where a sign stands for something to somebody and
its interpretation depends on context. That is the claim: `DENY` is not a fact but a classification
a producer applies, and what it means is settled by interpretation.

**Observation** and **signal** separate the general from the specific. Every emission is an
observation; a signal is the qualified case.

**Ascent** names the movement — from domain-specific to universal, from particular to general.
Movement, not conversion, because information is deliberately lost on the way up.

**Instrument** is borrowed from measurement: the thing you attach to a subject to make it legible.
It is deliberately not "logger", "emitter", or "reporter", each of which suggests a destination.

### What the Names Do Not Imply

**Signal** does not mean an operating-system signal, a Qt/GTK signal, or a reactive-programming
signal. It is a sign with a dimension, and nothing else.

**Status** is not HTTP status, process exit status, or a state machine's state. It is an operational
reading offered with a stated confidence.

**Situation** is not an incident, an alert, or a page. It is an urgency reading, and what to do
about it is somebody else's decision.

**Operation** in `Operations` is an episode role — `BEGIN`, `ADVANCE`, `END` — not a method call, an
RPC, or a unit of work. In §5's operation/outcome distinction it means an act rather than a result;
the two usages are related but not identical, and SPEC.md keeps them in separate sections for that
reason.

**Category** is one of the two dimension kinds (SPEC.md §4.3), unrelated to the operation/outcome
distinction and unrelated to any type-theoretic sense.
