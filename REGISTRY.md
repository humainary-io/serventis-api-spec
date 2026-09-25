# The Serventis Vocabulary Registry

**Companion to SPEC.md Version 3.6.0** **Copyright © 2025–2026 William David Louth / Humainary**

---

## 1. Purpose and Status

This document is the registry of Serventis vocabularies. It has two jobs.

**Part I** documents the **taxonomy**: the families of signs and dimensions that recur across
vocabularies, what each family means, and how a vocabulary specializes one. This part is
**non-normative**. It is a description of the vocabulary system as it stands, and the reference an
author consults when designing a new vocabulary under SPEC.md §8.1.

**Part II** records the **registered vocabularies** themselves. Its status is split, per SPEC.md
§8.2 and the layering of SPEC.md §1.2:

- The **membership and meanings** of each vocabulary — its sign set, its dimension set, and what
  those members mean — is **normative in content**, together with its primary/derived
  classification. A projection need not provide any registered vocabulary, but one that provides a
  vocabulary under a registered name MUST use exactly what is recorded here. This is what makes a
  registered name worth recognizing.

  That authority covers the **32 registered domain vocabularies** only. The eight universal entries
  are reproduced here for convenience; their authority is SPEC.md §7, and where a universal entry
  here and §7 disagree, §7 governs.
- The **property map columns** — `Kind`, `Status`, `Operation`, `Outcome` — are **non-normative**.
  They record the interpretive policy the Java reference projection publishes, and are documented
  here so that a second projection has a considered starting point. A projection that publishes
  different policy, or none, remains fully conformant.

**Part III** provides indexes over both.

The registry covers 40 entries: 8 universal (required by SPEC.md §7, of which `Surveys` and `Cycles`
are vocabulary *templates* that take their sign set from the caller) and 32 registered domain
vocabularies (optional, fixed membership). Together they publish 237 sign slots drawn from 161
distinct lexemes, and 51 dimension slots.

## 2. How to Read an Entry

Each **registered domain** entry is addressable as `registry:<name>`, where `<name>` is the entry's
name in lower case — `registry:locks`, `registry:caches`. That identifier is what a traceability
annotation or a conformance test cites to name the contract it is bound to (SPEC.md §A.2).
Identifiers are stable: an entry is never renamed, because a rename would be a different vocabulary
(SPEC.md §8.3).

The eight universal entries are **not** addressable that way. Their authority is a section of
SPEC.md §7, cited with a bare identifier such as `7.1`; each entry below records which applies.

Each entry records:

- **Instrument** — the vocabulary's emission surface and whether it is a Signer (bare signs) or a
  Signaler (qualified signals). See SPEC.md §6.
- **Shape** — the size of the sign set and dimension set. A vocabulary with no dimension set emits
  atomic observations; one with a dimension set emits signals over a signal space of `|S| × |D|`.
- **Properties** — which property maps the vocabulary publishes in the Java projection
  (non-normative; see SPEC.md §B.2).
- **Classification** — whether the vocabulary is *primary* or *derived* under SPEC.md §8.1.1, with
  the reason. For a registered domain vocabulary this is normative: it decides whether the
  implementer-observable rule applies.
- **Reference** — how a traceability annotation or conformance test addresses this entry.
- **Sign table** — one row per sign, with its meaning, and with the Java projection's property map
  values where published. The **meaning column is normative**; the property columns are not. A dash
  in a property column means that map abstains on that sign (SPEC.md §5.1); it does not mean the
  value is unknown or defaulted. `Kind` and `Operation` columns are total, so they never show a
  dash.
- **Dimension table** — where present, the dimension set and its kind (category or spectrum).

A vocabulary listing no properties is not deficient, and no vocabulary is required to publish any.
The universal vocabularies are ascent *targets*, so they have nothing above them to translate into.
Every registered domain vocabulary publishes `KIND`, but several are pure mechanism, with no
verdicts to read, and publish nothing else.

---

# Part I — The Taxonomy

## 3. Sign Families

A **sign family** is a group of textual names (lexemes) that answer the same kind of question about
a subject.
Families are an organizing lens over the registry, not a structure the specification defines: no
projection needs to know they exist, and nothing in SPEC.md refers to them. They exist because
vocabulary reuse (SPEC.md §8.1.6) needs something concrete to reuse *from*, and because the shape of
the reuse is itself informative.

Family membership is assigned per **slot** — per (vocabulary, sign) pair — rather than per lexeme.
This matters: the same word can belong to different families in different vocabularies, and §4
collects those cases. Every one of the 237 sign slots in the registry belongs to exactly one family.

### 3.1 Acquisition

Requesting, holding, and giving up a scarce thing. The most reused family in the registry, and the
one with the clearest canonical form: `Resources` publishes the six-sign core — `ATTEMPT`,
`ACQUIRE`, `GRANT`, `DENY`, `TIMEOUT`, `RELEASE` — and `Locks` and `Leases` extend it rather than
inventing their own.

The family's shape is a request that may be refused: an attempt or acquisition opens, a grant or
denial decides, a release closes. This is why every vocabulary in the family except `Pools`
publishes a full set of four property maps, and why the bracket sequencer (SPEC.md §B.4.1) works so
naturally on them. `Pools` has no refusal to read, only acts.

| Lexeme      | Vocabularies                      |
|-------------|-----------------------------------|
| `ABANDON`   | Latches, Locks                    |
| `ACQUIRE`   | Leases, Locks, Resources          |
| `ARRIVE`    | Latches                           |
| `ATTEMPT`   | Atomics, Locks, Resources         |
| `AWAIT`     | Latches                           |
| `BORROW`    | Pools                             |
| `DENY`      | Leases, Locks, Resources          |
| `DOWNGRADE` | Locks                             |
| `EXPIRE`    | Leases                            |
| `EXTEND`    | Leases                            |
| `GRANT`     | Leases, Locks, Resources          |
| `PROBE`     | Leases                            |
| `RECLAIM`   | Pools                             |
| `RELEASE`   | Latches, Leases, Locks, Resources |
| `RENEW`     | Leases                            |
| `REVOKE`    | Leases                            |
| `TIMEOUT`   | Latches, Locks, Resources         |
| `UPGRADE`   | Locks                             |

### 3.2 Lifecycle

The existence and running state of a long-lived subject: creation, running, stopping, destruction,
and the suspensions in between. Distinguished from Work (§3.3) by what it describes — Lifecycle is
about a thing that exists, Work about a job that completes.

`Processes` is the canonical member. `SUSPEND`/`RESUME` is the family's characteristic symmetric
pair.

| Lexeme    | Vocabularies               |
|-----------|----------------------------|
| `CRASH`   | Processes                  |
| `FAIL`    | Processes                  |
| `KILL`    | Processes                  |
| `RESTART` | Processes                  |
| `RESUME`  | Processes, Services, Tasks |
| `SPAWN`   | Processes                  |
| `START`   | Processes, Services        |
| `STOP`    | Processes, Services        |
| `SUSPEND` | Processes, Services, Tasks |

### 3.3 Work

A unit of work moving through its stages: submitted, scheduled, started, progressing, finished one
way or another. `Tasks` is the canonical member.

The family shares `START`, `SUSPEND`, and `RESUME` with Lifecycle and `TIMEOUT` and `EXPIRE` with
Acquisition, which is the ordinary consequence of a job being a thing that runs and can be waited
on. `EXPIRE` here is work outliving its deadline, the same event as `Tasks.TIMEOUT`.

| Lexeme     | Vocabularies           |
|------------|------------------------|
| `CALL`     | Services               |
| `CANCEL`   | Tasks                  |
| `COMPLETE` | Tasks                  |
| `EXPIRE`   | Services, Transactions |
| `FAIL`     | Tasks                  |
| `PROGRESS` | Changes, Tasks         |
| `REJECT`   | Tasks                  |
| `SCHEDULE` | Services, Tasks        |
| `START`    | Tasks                  |
| `SUBMIT`   | Tasks                  |
| `TIMEOUT`  | Tasks                  |

### 3.4 Episode

Bracketing structure as such: the universal `Operations` vocabulary and the domain signs that exist
purely to open or stage an episode. Small by design — most bracketing is expressed by projecting a
domain vocabulary through its `OPERATION` map (SPEC.md §B.2) rather than by emitting episode signs
directly.

| Lexeme    | Vocabularies          |
|-----------|-----------------------|
| `ADVANCE` | Operations            |
| `BEGIN`   | Operations            |
| `END`     | Operations            |
| `PREPARE` | Transactions          |
| `START`   | Changes, Transactions |

### 3.5 Verdict

Results. What an operation decided, observed, or produced. This is the family that ascends by tally
(SPEC.md §B.3), and every slot in it has kind `OUTCOME` wherever its vocabulary publishes a `KIND`
map.

The family is notable for how many domain-specific synonyms it carries for the same two verdicts:
`SUCCESS`, `COMPLETE`, `COMMIT`, `FULFILL`, `SUCCEED`, `PASS`, `HIT`, `MEET`, `ACK`, `APPLY`, and
`GRANT` are all the positive verdict in their own vocabulary's idiom. This is deliberate — a vocabulary reads better
when its verdict is named in its own terms — and it is exactly what the `OUTCOME` map exists to
normalize.

| Lexeme     | Vocabularies                                               |
|------------|------------------------------------------------------------|
| `ABORT`    | Transactions                                               |
| `ACK`      | Messages                                                   |
| `APPLY`    | Changes                                                    |
| `BREACH`   | Agents                                                     |
| `COMMIT`   | Transactions                                               |
| `ERROR`    | Evals                                                      |
| `FAIL`     | Atomics, Changes, Evals, Flows, Outcomes, Probes, Services |
| `FULFILL`  | Agents                                                     |
| `HIT`      | Caches                                                     |
| `MEET`     | Timers                                                     |
| `MISS`     | Caches, Timers                                             |
| `NACK`     | Messages                                                   |
| `PASS`     | Evals                                                      |
| `REVERT`   | Changes                                                    |
| `ROLLBACK` | Transactions                                               |
| `SKIP`     | Evals                                                      |
| `SUCCEED`  | Probes                                                     |
| `SUCCESS`  | Atomics, Flows, Outcomes, Services                         |
| `UNKNOWN`  | Evals, Outcomes                                            |

### 3.6 Admission

Letting something through, or not. Distinguished from Verdict by direction: an admission decision is
made *about incoming work* by a gatekeeper, where a verdict is made *about completed work* by
whoever did it.

`Valves` is the canonical member. The family overlaps Acquisition on `DENY`, which is the same
refusal seen from a resource rather than a gate. `Guards` reuses Acquisition's request-and-decision
lexemes `ATTEMPT`, `GRANT` and `DENY` for access decisions. A guard is a gate admitting a caller, so
its slots belong here.

| Lexeme      | Vocabularies      |
|-------------|-------------------|
| `ATTEMPT`   | Guards            |
| `CHALLENGE` | Guards            |
| `DELAY`     | Services          |
| `DENY`      | Guards, Valves    |
| `DISCARD`   | Services          |
| `DROP`      | Routers, Valves   |
| `FILTER`    | Pipelines         |
| `GRANT`     | Guards            |
| `PASS`      | Valves            |
| `REJECT`    | Changes, Services |
| `SKIP`      | Pipelines         |

### 3.7 Boundary

Capacity limits reached. The family's signature is a symmetric pair — `OVERFLOW`/`UNDERFLOW` —
appearing wherever a container has two ends it can run out of. `Queues` and `Stacks` carry both;
`Counters`, which only counts up, carries `OVERFLOW` alone.

| Lexeme         | Vocabularies                                |
|----------------|---------------------------------------------|
| `BACKPRESSURE` | Pipelines                                   |
| `EXHAUST`      | Atomics, Messages                           |
| `LAG`          | Pipelines                                   |
| `LIMIT`        | Systems                                     |
| `OVERFLOW`     | Counters, Gauges, Pipelines, Queues, Stacks |
| `UNDERFLOW`    | Gauges, Queues, Stacks                      |

### 3.8 Elasticity

Capacity changing size. `EXPAND`/`CONTRACT` is the pair; `DRAIN` is the emptying case. Note that
`CONTRACT` here is the antonym of `EXPAND` and unrelated to the `CONTRACT` of §3.16 — see §4.

| Lexeme     | Vocabularies  |
|------------|---------------|
| `CONTRACT` | Pools, Valves |
| `DRAIN`    | Valves        |
| `EXPAND`   | Pools, Valves |

### 3.9 Contention

Competing for progress. What a subject does while it cannot proceed: spinning, yielding, backing
off, parking. `Atomics` is the canonical member and the family is where the implementer-observable
principle (SPEC.md §8.1.1) shows most clearly — these are four things an implementation genuinely
did, not one inferred judgment that it was "slow".

| Lexeme     | Vocabularies |
|------------|--------------|
| `BACKOFF`  | Atomics      |
| `CONFLICT` | Transactions |
| `CONTEST`  | Locks        |
| `PARK`     | Atomics      |
| `SPIN`     | Atomics      |
| `YIELD`    | Atomics      |

### 3.10 Transport

Moving something from one place to another: sending, receiving, forwarding, routing, enqueueing,
pushing. The family covers both network transport and in-process container movement, because the
observable shape is the same.

| Lexeme       | Vocabularies      |
|--------------|-------------------|
| `CONNECT`    | Probes            |
| `DELIVER`    | Messages          |
| `DEQUEUE`    | Queues            |
| `DISCONNECT` | Probes, Services  |
| `ENQUEUE`    | Queues            |
| `FORWARD`    | Routers           |
| `FRAGMENT`   | Routers           |
| `INPUT`      | Pipelines         |
| `OUTPUT`     | Pipelines         |
| `POP`        | Stacks            |
| `PUBLISH`    | Messages          |
| `PUSH`       | Stacks            |
| `REASSEMBLE` | Routers           |
| `RECEIVE`    | Routers           |
| `ROUTE`      | Routers           |
| `SEND`       | Routers           |
| `TRANSFER`   | Exchanges, Probes |

### 3.11 Storage

Keeping something and finding it again. `Caches` is the canonical member; the family's
characteristic sequence is `LOOKUP` followed by `HIT` or `MISS`.

| Lexeme   | Vocabularies     |
|----------|------------------|
| `BUFFER` | Pipelines        |
| `EVICT`  | Caches           |
| `EXPIRE` | Caches, Messages |
| `LOOKUP` | Caches           |
| `REMOVE` | Caches           |
| `STORE`  | Caches           |

### 3.12 Recovery

Attempts to return to normal after something went wrong: retrying, resetting, re-routing, probing to
see whether it is safe yet. `Breakers` is the canonical member, and its `TRIP` → `OPEN` →
`HALF_OPEN` → `PROBE` → `CLOSE` cycle is the family's most complete expression.

| Lexeme       | Vocabularies      |
|--------------|-------------------|
| `CLOSE`      | Breakers          |
| `COMPENSATE` | Transactions      |
| `HALF_OPEN`  | Breakers          |
| `OPEN`       | Breakers          |
| `PROBE`      | Breakers          |
| `RECOURSE`   | Services          |
| `REDELIVER`  | Messages          |
| `REDIRECT`   | Services          |
| `RESET`      | Breakers, Latches |
| `RETRY`      | Services          |
| `TRIP`       | Breakers          |

### 3.13 Integrity

Damage and disorder in what was moved or stored: corruption, reordering, checkpointing and
watermarking against them. Small, and almost always the most severe reading in its vocabulary's
`STATUS` map.

| Lexeme       | Vocabularies |
|--------------|--------------|
| `CHECKPOINT` | Pipelines    |
| `CORRUPT`    | Routers      |
| `REORDER`    | Routers      |
| `WATERMARK`  | Pipelines    |

### 3.14 Counting

Numeric movement without a subject to speak of. `INCREMENT`/`DECREMENT` with a `RESET`. The
narrowest family, and the one closest to conventional metrics.

| Lexeme      | Vocabularies     |
|-------------|------------------|
| `DECREMENT` | Gauges           |
| `INCREMENT` | Counters, Gauges |
| `RESET`     | Counters, Gauges |

### 3.15 Assessment

Readings rather than events. This family is exceptional: its members do not report something that
happened, they report a conclusion about how things are. It is where the universal ascent targets
live — `Statuses`, `Situations`, `Systems`, `Trends` — together with the domain vocabularies whose
producers genuinely do assess: a log's severity, a sensor's band.

Because assessment signs are conclusions, a producer that emits them is doing interpretive work.
Every vocabulary in this family is therefore **derived** under SPEC.md §8.1.1, which exempts them
from the implementer-observable rule: inference is what produces their signs, so requiring them to
be directly observable would be incoherent. `Statuses`, `Situations`, `Systems`, and `Trends` are
the universal ascent targets; `Sensors` and `Logs` are the registered domain vocabularies here, a
sensor comparing against a reference and a log classifying a severity.

This family is **not** the derived inventory. Family membership and the primary/derived split are
independent questions, and vocabularies outside this family are derived too — `Evals` sits in the
Verdict family (§3.5) yet judges work against a criterion, `Timers` in Acquisition yet compares
elapsed time against a configured limit, `Routers` in Transport yet must remember send order for
`REORDER`, `Pipelines` in Transport yet must measure progress for `LAG`. Each entry in Part II
records its own classification, and those records are the inventory; this section explains only why
one family is derived in its entirety.

| Lexeme       | Vocabularies        |
|--------------|---------------------|
| `ABOVE`      | Sensors             |
| `ALARM`      | Systems             |
| `BELOW`      | Sensors             |
| `CHAOS`      | Trends              |
| `CONVERGING` | Statuses            |
| `CRITICAL`   | Situations          |
| `CYCLE`      | Trends              |
| `DEBUG`      | Logs                |
| `DEFECTIVE`  | Statuses            |
| `DEGRADED`   | Statuses            |
| `DIVERGING`  | Statuses            |
| `DOWN`       | Statuses            |
| `DRIFT`      | Trends              |
| `ERRATIC`    | Statuses            |
| `FAULT`      | Systems             |
| `INFO`       | Logs                |
| `NOMINAL`    | Sensors             |
| `NORMAL`     | Situations, Systems |
| `SEVERE`     | Logs                |
| `SPIKE`      | Trends              |
| `STABLE`     | Statuses, Trends    |
| `WARNING`    | Logs, Situations    |

### 3.16 Speech act

Communicative moves between parties: asking, disagreeing, promising, offering, accepting,
retracting. Both members of the family — `Actors` for conversation, `Agents` for promise-theoretic
coordination — describe interactions in which the utterance *is* the event.

| Lexeme        | Vocabularies   |
|---------------|----------------|
| `ACCEPT`      | Agents         |
| `ACKNOWLEDGE` | Actors         |
| `AFFIRM`      | Actors         |
| `ASK`         | Actors         |
| `CLARIFY`     | Actors         |
| `COMMAND`     | Actors         |
| `CONTRACT`    | Exchanges      |
| `DELIVER`     | Actors         |
| `DENY`        | Actors         |
| `DEPEND`      | Agents         |
| `EXPLAIN`     | Actors         |
| `INQUIRE`     | Agents         |
| `OBSERVE`     | Agents         |
| `OFFER`       | Agents         |
| `PROMISE`     | Actors, Agents |
| `REPORT`      | Actors         |
| `REQUEST`     | Actors         |
| `RETRACT`     | Agents         |
| `VALIDATE`    | Agents         |

### 3.17 Processing

Working on data already in hand: changing it, combining it, or handling it once it has arrived. The
family is distinguished by what happens to the data. Transport moves it, Storage keeps it, Admission
decides whether it passes, and Processing changes it. So in `Pipelines`, `FILTER` stays in Admission
and `BUFFER` in Storage, while `TRANSFORM` and `AGGREGATE` belong here.

`AGGREGATE` holds records until its window closes, but the sign reports the combining. Reporting the
holding is `BUFFER`'s job.

| Lexeme      | Vocabularies |
|-------------|--------------|
| `AGGREGATE` | Pipelines    |
| `PROCESS`   | Probes       |
| `TRANSFORM` | Pipelines    |

### 3.18 Membership

Belonging to a group and holding a role in it: joining and leaving, being suspected of failure and
refuting it or being removed, becoming leader and stepping down. `Members` is the only member. Its
signs report the membership protocol's decisions. Lifecycle covers a subject's own running state;
Membership covers the group's view of its members.

| Lexeme    | Vocabularies |
|-----------|--------------|
| `ELECT`   | Members      |
| `EVICT`   | Members      |
| `JOIN`    | Members      |
| `LEAVE`   | Members      |
| `REFUTE`  | Members      |
| `RESIGN`  | Members      |
| `SUSPECT` | Members      |

## 4. Cross-Family Lexemes and Homographs

Fifteen lexemes appear in more than one family. These are the concrete cases behind SPEC.md §8.4:
signs that share a name and are not the same sign.

| Lexeme     | Reading A                                                                                         | Reading B                                                                  | Reading C                                                              |
|------------|---------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------|
| `ATTEMPT`  | **Acquisition** — try to take a held thing (Atomics, Locks, Resources)                            | **Admission** — present a credential or request at a gate (Guards)         |                                                                        |
| `CONTRACT` | **Elasticity** — shrink capacity (Pools, Valves)                                                  | **Speech act** — commit to terms (Exchanges)                               |                                                                        |
| `DELIVER`  | **Speech act** — present completed work (Actors)                                                  | **Transport** — hand a message to a receiver (Messages)                    |                                                                        |
| `DENY`     | **Acquisition** — refuse a request for a held thing (Leases, Locks, Resources)                    | **Admission** — refuse entry at a gate (Guards, Valves)                    | **Speech act** — disagree with a proposition (Actors)                  |
| `EVICT`    | **Storage** — remove an entry for capacity or policy (Caches)                                     | **Membership** — remove a member declared failed (Members)                 |                                                                        |
| `EXPIRE`   | **Acquisition** — a lease's term ran out (Leases)                                                 | **Work** — the work outlived its deadline (Services, Transactions)         | **Storage** — a held item outlived its time to live (Caches, Messages) |
| `FAIL`     | **Verdict** — the result was failure (Atomics, Changes, Evals, Flows, Outcomes, Probes, Services) | **Lifecycle** — the process died (Processes)                               | **Work** — the job did not complete (Tasks)                            |
| `GRANT`    | **Acquisition** — hand over a held thing (Leases, Locks, Resources)                               | **Admission** — let a caller through a gate (Guards)                       |                                                                        |
| `PASS`     | **Admission** — allowed through (Valves)                                                          | **Verdict** — satisfied the criterion (Evals)                              |                                                                        |
| `PROBE`    | **Acquisition** — check a lease is still valid (Leases)                                           | **Recovery** — a trial request testing whether recovery is safe (Breakers) |                                                                        |
| `REJECT`   | **Admission** — refused at submission (Changes, Services)                                         | **Work** — the job was not accepted (Tasks)                                |                                                                        |
| `RESET`    | **Counting** — return a counter to zero (Counters, Gauges)                                        | **Recovery** — return a mechanism to its initial state (Breakers, Latches) |                                                                        |
| `SKIP`     | **Admission** — an element was passed over (Pipelines)                                            | **Verdict** — the criterion was deliberately not evaluated (Evals)         |                                                                        |
| `START`    | **Lifecycle** — the subject began running (Processes, Services)                                   | **Work** — the job began executing (Tasks)                                 | **Episode** — an episode opened (Changes, Transactions)                |
| `TIMEOUT`  | **Acquisition** — the wait for a held thing expired (Latches, Locks, Resources)                   | **Work** — the job exceeded its budget (Tasks)                             |                                                                        |

Two further variations do not show up as family differences but matter as much.

**`FAIL` differs in episode role.** In `Atomics`, `FAIL` maps to `ADVANCE` — a compare-and-swap that
lost will be retried, so the episode continues. Everywhere else it maps to `END`. Same word, same
family, different structure behind it.

**`RELEASE` differs in who acts.** In `Locks`, `Resources`, and `Leases` a holder gives up what it
held. In `Latches` the barrier itself opens and releases everyone waiting. Both are Acquisition, but
the actor is inverted.

The registry does not rename to eliminate these. Renaming would make the individual vocabularies
read worse to serve a global uniqueness property that no consumer of a single vocabulary benefits
from, and SPEC.md §4.2 already establishes that a sign has no meaning independent of its set. The
list exists so that nobody builds a cross-vocabulary lexeme table by accident.

## 5. Dimension Families

Of the 40 registry entries, 18 publish a dimension set. The others emit atomic observations.
This ratio is the expected result of applying SPEC.md §8.1.5 honestly: a dimension is warranted only
when the same sign genuinely means something different under each member.

The dimension sets fall into six category families and three spectra.

### 5.1 Perspective (category)

The largest family, and the one SPEC.md §8.1.5 has in mind. A perspective dimension names **which
party to an interaction** is reporting. The same event, observed from both ends, produces two
signals that differ only in dimension — and the difference between them is exactly the information a
consumer needs to detect a disagreement between the parties.

| Vocabulary   | Dimensions                       | The ends                               |
|--------------|----------------------------------|----------------------------------------|
| Services     | `CALLER`, `CALLEE`               | who invoked, who served                |
| Agents       | `PROMISER`, `PROMISEE`           | who promised, who was promised         |
| Transactions | `COORDINATOR`, `PARTICIPANT`     | who led, who took part                 |
| Leases       | `LESSOR`, `LESSEE`               | who granted, who held                  |
| Exchanges    | `PROVIDER`, `RECEIVER`           | who gave, who took                     |
| Probes       | `OUTBOUND`, `INBOUND`            | which direction the traffic went       |
| Messages     | `PRODUCER`, `BROKER`, `CONSUMER` | who sent, who relayed, who received    |
| Members      | `SELF`, `PEER`                   | the member concerned, any other member |

Every member but `Messages` is dyadic. That is not a rule: most interactions worth observing have two
ends, and a delivery through a broker has three, each able to report the same settlement.

### 5.2 Stage (category)

Where in a pipeline the observation was made.

| Vocabulary | Dimensions                     |
|------------|--------------------------------|
| Flows      | `INGRESS`, `TRANSIT`, `EGRESS` |

This is the family that comes closest to the location trap SPEC.md §8.1.5 warns against, and it
stays on the right side of the line because the stages are positions in a *processing* structure,
not deployment locations. A flow failing at ingress and one failing at egress are genuinely
different failures.

### 5.3 Constraint kind (category)

Which class of system limit a condition implicates.

| Vocabulary | Dimensions                      |
|------------|---------------------------------|
| Systems    | `SPACE`, `FLOW`, `LINK`, `TIME` |

The four are the exhaustive partition of what a system can run out of: room, throughput,
connectivity, and time.

### 5.4 Reference point (category)

What the observation was measured against.

| Vocabulary | Dimensions                        |
|------------|-----------------------------------|
| Timers     | `DEADLINE`, `THRESHOLD`           |
| Sensors    | `BASELINE`, `THRESHOLD`, `TARGET` |

`THRESHOLD` appears in both, and both intend the same thing — a configured limit — which makes it
the only dimension lexeme shared across vocabularies. They remain two distinct dimensions in two
distinct sets, and the coincidence of meaning is documentation rather than something an
implementation may rely on: SPEC.md §8.4 forbids unifying them, and forbids it precisely because
aligned meanings make the temptation strongest.

### 5.5 Criterion (category)

Which axis of judgment a verdict was reached on.

| Vocabulary | Dimensions                                                                                                |
|------------|-----------------------------------------------------------------------------------------------------------|
| Evals      | `CORRECTNESS`, `COMPLETENESS`, `GROUNDEDNESS`, `HELPFULNESS`, `ADHERENCE`, `TOOLING`, `ROUTING`, `SAFETY` |
| Guards     | `IDENTITY`, `PERMISSION`                                                                                  |

`Evals` is the largest dimension set in the registry, and the clearest case of a dimension carrying
real semantic weight: `FAIL × SAFETY` and `FAIL × HELPFULNESS` are different enough that conflating
them would be a serious error. `Guards` is the smallest: `DENY × IDENTITY` is a bad credential and
`DENY × PERMISSION` a known caller without the right.

### 5.6 Recurrence (category)

How the sign recurred in the stream. Produced by the recurrence operator (SPEC.md §B.5) rather than
chosen by a producer.

| Vocabulary | Dimensions                   |
|------------|------------------------------|
| Cycles     | `SINGLE`, `REPEAT`, `RETURN` |

### 5.7 The three spectra

A spectrum's members are ordered and the order carries meaning (SPEC.md §4.3), so a consumer may
compare them by rank.

| Vocabulary | Spectrum    | Low → High                             |
|------------|-------------|----------------------------------------|
| Statuses   | confidence  | `TENTATIVE` → `MEASURED` → `CONFIRMED` |
| Situations | variability | `CONSTANT` → `VARIABLE` → `VOLATILE`   |
| Surveys    | consensus   | `DIVIDED` → `MAJORITY` → `UNANIMOUS`   |

All three qualify an assessment rather than an event, and all three answer the same shape of
question: how much should this reading be trusted, and on what grounds? Confidence answers from
weight of evidence, consensus from agreement among observers, variability from the stability of what
is being assessed.

---

# Part II — Registered Vocabularies

In every entry below, the **meaning column is normative** and the property columns are not: those
record the reference projection's interpretive policy (SPEC.md §1.2, §8.2), and another projection
MAY publish different values without affecting its conformance. A dash marks abstention, not an
unknown value. §2 gives the full reading conventions.

### Universal vocabularies

#### Statuses

- **Instrument**: Status (Signaler)
- **Shape**: 7 signs × 3 dimensions
- **Properties**: none published
- **Classification**: derived — selecting a condition means weighing accumulated evidence (SPEC.md
  §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

| Sign         | Meaning                                                                                                                                         |
|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `CONVERGING` | The subject of observation is stabilizing towards reliable operation, typically following initialization, scaling, or recovery.                 |
| `STABLE`     | The subject of observation is operating within expected parameters, characterized by consistent response patterns and acceptable success rates. |
| `DIVERGING`  | The subject of observation is destabilizing, with increasing variations in response times, error rates, or other operational metrics.           |
| `ERRATIC`    | The subject of observation is exhibiting unpredictable behavior with irregular transitions between different operational conditions.            |
| `DEGRADED`   | The subject of observation is partially operational, with reduced performance, elevated error rates, or delayed responses.                      |
| `DEFECTIVE`  | The subject of observation is unreliable, with predominantly failed operations and significant inability to meet service level objectives.      |
| `DOWN`       | The subject of observation is entirely non-operational and unable to process any requests.                                                      |

**Dimension** (Spectrum)

| Dimension   | Meaning                                                                             |
|-------------|-------------------------------------------------------------------------------------|
| `TENTATIVE` | A preliminary assessment based on initial behavioral patterns.                      |
| `MEASURED`  | An established assessment with strong evidence for the condition classification.    |
| `CONFIRMED` | A definitive assessment with unambiguous evidence for the condition classification. |

#### Situations

- **Instrument**: Situation (Signaler)
- **Shape**: 3 signs × 3 dimensions
- **Properties**: none published
- **Classification**: derived — selecting an urgency means judging a condition (SPEC.md §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

| Sign       | Meaning                                                                |
|------------|------------------------------------------------------------------------|
| `NORMAL`   | The situation poses no immediate concern and requires no intervention. |
| `WARNING`  | The situation requires attention but is not yet critical.              |
| `CRITICAL` | The situation is serious and demands prompt intervention.              |

**Dimension** (Spectrum)

| Dimension  | Meaning                                                               |
|------------|-----------------------------------------------------------------------|
| `CONSTANT` | The situation exhibits no variation, maintaining an unchanging level. |
| `VARIABLE` | The situation exhibits moderate variation and fluctuation.            |
| `VOLATILE` | The situation exhibits high instability with rapid, chaotic changes.  |

#### Operations

- **Instrument**: Operation (Signer)
- **Shape**: 3 signs, no dimensions
- **Properties**: none published
- **Classification**: primary — an episode role is witnessed as it happens (SPEC.md §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

| Sign      | Meaning                                                                            |
|-----------|------------------------------------------------------------------------------------|
| `BEGIN`   | An action is starting.                                                             |
| `ADVANCE` | An action is progressing within an open episode — neither beginning nor ending it. |
| `END`     | An action is finishing.                                                            |

#### Outcomes

- **Instrument**: Outcome (Signer)
- **Shape**: 3 signs, no dimensions
- **Properties**: none published
- **Classification**: primary — a verdict is witnessed as it is reached (SPEC.md §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

| Sign      | Meaning                                                                        |
|-----------|--------------------------------------------------------------------------------|
| `SUCCESS` | The operation succeeded.                                                       |
| `FAIL`    | The operation failed.                                                          |
| `UNKNOWN` | The verdict is indeterminate — neither a clean success nor a definite failure. |

#### Systems

- **Instrument**: System (Signaler)
- **Shape**: 4 signs × 4 dimensions
- **Properties**: none published
- **Classification**: derived — selecting a constraint level means judging pressure against a limit
  (SPEC.md §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

| Sign     | Meaning                               |
|----------|---------------------------------------|
| `NORMAL` | Operating within standard parameters. |
| `LIMIT`  | At constraint boundary.               |
| `ALARM`  | Beyond constraint boundary.           |
| `FAULT`  | Constraint failed.                    |

**Dimension** (Category)

| Dimension | Meaning                                                    |
|-----------|------------------------------------------------------------|
| `SPACE`   | Container constraint — capacity, volume, room.             |
| `FLOW`    | Movement constraint — throughput, bandwidth, rate.         |
| `LINK`    | Structural constraint — connectivity, reachability, edges. |
| `TIME`    | Temporal constraint — latency, responsiveness, duration.   |

#### Trends

- **Instrument**: Trend (Signer)
- **Shape**: 5 signs, no dimensions
- **Properties**: none published
- **Classification**: derived — a series shape is read from history, not from one moment (SPEC.md
  §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

| Sign     | Meaning                                                            |
|----------|--------------------------------------------------------------------|
| `STABLE` | The process is in statistical control with no significant pattern. |
| `DRIFT`  | Sustained movement away from the baseline.                         |
| `SPIKE`  | A sudden extreme deviation (outlier).                              |
| `CYCLE`  | An oscillating pattern.                                            |
| `CHAOS`  | Erratic, chaotic variation.                                        |

#### Surveys

- **Instrument**: Survey (Signaler)
- **Shape**: vocabulary template — sign set supplied at materialization × 3 dimensions
- **Properties**: none published
- **Classification**: derived — agreement among observers must be gathered before it can be reported
  (SPEC.md §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

**Dimension** (Spectrum)

| Dimension   | Meaning                                    |
|-------------|--------------------------------------------|
| `DIVIDED`   | No clear majority among observers.         |
| `MAJORITY`  | Clear majority but not complete agreement. |
| `UNANIMOUS` | Complete agreement among all observers.    |

#### Cycles

- **Instrument**: Cycle (Signaler)
- **Shape**: vocabulary template — sign set supplied at materialization × 3 dimensions
- **Properties**: none published
- **Classification**: derived — recurrence requires remembering which signs were already emitted
  (SPEC.md §8.1.1)
- **Reference**: `7` — authority is SPEC.md §7, not the registry

**Dimension** (Category)

| Dimension | Meaning                                      |
|-----------|----------------------------------------------|
| `SINGLE`  | First occurrence of this sign in the stream. |
| `REPEAT`  | Same sign as immediately previous emission.  |
| `RETURN`  | Seen before, but not immediately previous.   |

### Execution

#### Tasks

- **Instrument**: Task (Signer)
- **Shape**: 11 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — each sign is a stage the executor itself performed or was told of
  (SPEC.md §8.1.1)
- **Reference**: `registry:tasks`

| Sign       | Kind        | Status     | Operation | Outcome   | Meaning                              |
|------------|-------------|------------|-----------|-----------|--------------------------------------|
| `SUBMIT`   | `OPERATION` | —          | `BEGIN`   | —         | Task submitted to executor or queue. |
| `REJECT`   | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | Task submission rejected.            |
| `SCHEDULE` | `OPERATION` | —          | `ADVANCE` | —         | Task scheduled for execution.        |
| `START`    | `OPERATION` | —          | `ADVANCE` | —         | Task execution begins.               |
| `PROGRESS` | `OPERATION` | —          | `ADVANCE` | —         | Task reports progress.               |
| `SUSPEND`  | `OPERATION` | —          | `ADVANCE` | —         | Task paused or suspended.            |
| `RESUME`   | `OPERATION` | —          | `ADVANCE` | —         | Task resumed after suspension.       |
| `COMPLETE` | `OUTCOME`   | `STABLE`   | `END`     | `SUCCESS` | Task completed successfully.         |
| `FAIL`     | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | Task failed with error.              |
| `CANCEL`   | `OPERATION` | —          | `END`     | —         | Task cancelled.                      |
| `TIMEOUT`  | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | Task exceeded execution time budget. |

#### Services

- **Instrument**: Service (Signaler)
- **Shape**: 16 signs × 2 dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — each sign is an interaction the participant performed or received
  (SPEC.md §8.1.1)
- **Reference**: `registry:services`

| Sign         | Kind        | Status     | Meaning                                                   |
|--------------|-------------|------------|-----------------------------------------------------------|
| `START`      | `OPERATION` | —          | The start of work execution.                              |
| `STOP`       | `OPERATION` | —          | The completion of work execution (regardless of outcome). |
| `CALL`       | `OPERATION` | —          | A request (call) for work to be done by another service.  |
| `SUCCESS`    | `OUTCOME`   | `STABLE`   | Successful completion of work.                            |
| `FAIL`       | `OUTCOME`   | `DEGRADED` | Failure to complete work.                                 |
| `RECOURSE`   | `OPERATION` | `DEGRADED` | Activation of a degraded operational mode after failure.  |
| `REDIRECT`   | `OPERATION` | —          | Forwarding of work to an alternative service or endpoint. |
| `EXPIRE`     | `OUTCOME`   | `DEGRADED` | Work exceeded its time budget.                            |
| `RETRY`      | `OPERATION` | —          | Automatic retry of work after failure.                    |
| `REJECT`     | `OUTCOME`   | `DEGRADED` | Refusal to accept work.                                   |
| `DISCARD`    | `OPERATION` | —          | Deliberate dropping of work.                              |
| `DELAY`      | `OPERATION` | —          | Postponement of work.                                     |
| `SCHEDULE`   | `OPERATION` | —          | Work queued for future execution.                         |
| `SUSPEND`    | `OPERATION` | —          | Work paused awaiting resumption.                          |
| `RESUME`     | `OPERATION` | —          | Previously suspended work restarting.                     |
| `DISCONNECT` | `OUTCOME`   | `DEGRADED` | Inability to reach or communicate with a service.         |

**Dimension** (Category)

| Dimension | Meaning                                                          |
|-----------|------------------------------------------------------------------|
| `CALLER`  | Emitted by the service in the caller role (initiating requests). |
| `CALLEE`  | Emitted by the service in the callee role (serving requests).    |

#### Processes

- **Instrument**: Process (Signer)
- **Shape**: 9 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — lifecycle transitions are witnessed by the supervisor (SPEC.md
  §8.1.1)
- **Reference**: `registry:processes`

| Sign      | Kind        | Status      | Operation | Outcome   | Meaning                           |
|-----------|-------------|-------------|-----------|-----------|-----------------------------------|
| `SPAWN`   | `OPERATION` | —           | `BEGIN`   | —         | New process created.              |
| `START`   | `OPERATION` | —           | `ADVANCE` | —         | Process execution begins.         |
| `STOP`    | `OUTCOME`   | `STABLE`    | `END`     | `SUCCESS` | Process stopped cleanly.          |
| `FAIL`    | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | Process failed.                   |
| `CRASH`   | `OUTCOME`   | `DEFECTIVE` | `END`     | `FAIL`    | Process crashed.                  |
| `KILL`    | `OPERATION` | `DEGRADED`  | `END`     | —         | Process forcibly terminated.      |
| `RESTART` | `OPERATION` | —           | `ADVANCE` | —         | Process being restarted.          |
| `SUSPEND` | `OPERATION` | —           | `ADVANCE` | —         | Process suspended.                |
| `RESUME`  | `OPERATION` | —           | `ADVANCE` | —         | Process resumed after suspension. |

#### Transactions

- **Instrument**: Transaction (Signaler)
- **Shape**: 8 signs × 2 dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — each sign is a coordination step performed or observed (SPEC.md
  §8.1.1)
- **Reference**: `registry:transactions`

| Sign         | Kind        | Status      | Operation | Outcome   | Meaning                                       |
|--------------|-------------|-------------|-----------|-----------|-----------------------------------------------|
| `START`      | `OPERATION` | —           | `BEGIN`   | —         | Transaction initiation.                       |
| `PREPARE`    | `OPERATION` | —           | `ADVANCE` | —         | The voting/prepare phase of two-phase commit. |
| `COMMIT`     | `OUTCOME`   | `STABLE`    | `END`     | `SUCCESS` | Transaction commitment.                       |
| `ROLLBACK`   | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | Transaction rollback.                         |
| `ABORT`      | `OUTCOME`   | `DEFECTIVE` | `END`     | `FAIL`    | Forced transaction termination.               |
| `EXPIRE`     | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | Transaction expiration.                       |
| `CONFLICT`   | `OUTCOME`   | `DEGRADED`  | `END`     | `UNKNOWN` | Write conflict or constraint violation.       |
| `COMPENSATE` | `OPERATION` | —           | `ADVANCE` | —         | Compensating action in saga pattern.          |

**Dimension** (Category)

| Dimension     | Meaning                                                                  |
|---------------|--------------------------------------------------------------------------|
| `COORDINATOR` | The emission of a transaction signal from the coordinator's perspective. |
| `PARTICIPANT` | The reception of a transaction signal from a participant's perspective.  |

#### Timers

- **Instrument**: Timer (Signaler)
- **Shape**: 2 signs × 2 dimensions
- **Properties**: STATUS, KIND
- **Classification**: derived — `MEET` vs `MISS` is chosen by comparing elapsed time against a
  configured deadline or threshold (SPEC.md §8.1.1)
- **Reference**: `registry:timers`

| Sign   | Kind      | Status     | Meaning                            |
|--------|-----------|------------|------------------------------------|
| `MEET` | `OUTCOME` | `STABLE`   | The time constraint was satisfied. |
| `MISS` | `OUTCOME` | `DEGRADED` | The time constraint was violated.  |

**Dimension** (Category)

| Dimension   | Meaning                         |
|-------------|---------------------------------|
| `DEADLINE`  | An absolute time constraint.    |
| `THRESHOLD` | A relative duration constraint. |

#### Changes

- **Instrument**: Change (Signer)
- **Shape**: 6 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — each sign is a step the change mechanism performed or a result it
  was given (SPEC.md §8.1.1)
- **Reference**: `registry:changes`

| Sign       | Kind        | Status     | Operation | Outcome   | Meaning                                                           |
|------------|-------------|------------|-----------|-----------|-------------------------------------------------------------------|
| `START`    | `OPERATION` | —          | `BEGIN`   | —         | A change to the subject began.                                    |
| `APPLY`    | `OUTCOME`   | `STABLE`   | `END`     | `SUCCESS` | The change took effect.                                           |
| `FAIL`     | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | The change could not be applied.                                  |
| `REJECT`   | `OUTCOME`   | —          | `END`     | `FAIL`    | The subject refused the change, by validation or policy.          |
| `REVERT`   | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | A change still rolling out was rolled back before it took effect. |
| `PROGRESS` | `OPERATION` | —          | `ADVANCE` | —         | The change advanced a stage, such as a canary step.               |

### Synchronization

#### Locks

- **Instrument**: Lock (Signer)
- **Shape**: 10 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — outcomes including `TIMEOUT` are returned by the acquisition itself
  (SPEC.md §8.1.1)
- **Reference**: `registry:locks`

| Sign        | Kind        | Status      | Operation | Outcome   | Meaning                                                |
|-------------|-------------|-------------|-----------|-----------|--------------------------------------------------------|
| `ATTEMPT`   | `OPERATION` | —           | `BEGIN`   | —         | A non-blocking lock acquisition attempt (try-lock).    |
| `ACQUIRE`   | `OPERATION` | —           | `BEGIN`   | —         | A blocking lock acquisition request (willing to wait). |
| `GRANT`     | `OUTCOME`   | `STABLE`    | `ADVANCE` | `SUCCESS` | Lock successfully obtained.                            |
| `DENY`      | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | Non-blocking acquisition denied (try-lock failed).     |
| `TIMEOUT`   | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | Blocking acquisition exceeded time limit.              |
| `RELEASE`   | `OPERATION` | —           | `END`     | —         | Lock voluntarily released by holder.                   |
| `UPGRADE`   | `OPERATION` | —           | `ADVANCE` | —         | Read lock upgraded to write lock (RW locks).           |
| `DOWNGRADE` | `OPERATION` | —           | `ADVANCE` | —         | Write lock downgraded to read lock (RW locks).         |
| `CONTEST`   | `OUTCOME`   | `DEGRADED`  | `ADVANCE` | `UNKNOWN` | CAS failure or contention detected.                    |
| `ABANDON`   | `OUTCOME`   | `DEFECTIVE` | `END`     | `FAIL`    | Lock holder terminated without releasing.              |

#### Latches

- **Instrument**: Latch (Signer)
- **Shape**: 6 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — arrival, release, and timeout are reported by the barrier (SPEC.md
  §8.1.1)
- **Reference**: `registry:latches`

| Sign      | Kind        | Status      | Operation | Outcome   | Meaning                                                                |
|-----------|-------------|-------------|-----------|-----------|------------------------------------------------------------------------|
| `AWAIT`   | `OPERATION` | —           | `BEGIN`   | —         | A thread is waiting at the barrier (blocking).                         |
| `ARRIVE`  | `OPERATION` | —           | `ADVANCE` | —         | A thread/participant has reached the barrier or decremented the count. |
| `RELEASE` | `OUTCOME`   | `STABLE`    | `END`     | `SUCCESS` | The barrier condition is satisfied and waiting threads are unblocked.  |
| `TIMEOUT` | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | A waiting thread's timeout was exceeded before barrier satisfaction.   |
| `RESET`   | `OPERATION` | —           | `ADVANCE` | —         | The barrier was reset for reuse in cyclic coordination.                |
| `ABANDON` | `OUTCOME`   | `DEFECTIVE` | `END`     | `FAIL`    | A participant terminated without arriving at the barrier.              |

#### Atomics

- **Instrument**: Atomic (Signer)
- **Shape**: 8 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — spin, backoff, and exhaustion are steps the retry loop itself takes
  (SPEC.md §8.1.1)
- **Reference**: `registry:atomics`

| Sign      | Kind        | Status      | Operation | Outcome   | Meaning                                     |
|-----------|-------------|-------------|-----------|-----------|---------------------------------------------|
| `ATTEMPT` | `OPERATION` | —           | `BEGIN`   | —         | A CAS operation has been initiated.         |
| `SUCCESS` | `OUTCOME`   | `STABLE`    | `END`     | `SUCCESS` | The CAS operation succeeded.                |
| `FAIL`    | `OUTCOME`   | `DEGRADED`  | `ADVANCE` | `UNKNOWN` | The CAS operation failed due to contention. |
| `SPIN`    | `OPERATION` | —           | `ADVANCE` | —         | Busy-wait retry in a spin loop.             |
| `YIELD`   | `OPERATION` | —           | `ADVANCE` | —         | Thread.yield() applied as scheduling hint.  |
| `BACKOFF` | `OPERATION` | —           | `ADVANCE` | —         | Deliberate delay applied between retries.   |
| `PARK`    | `OPERATION` | `DEGRADED`  | `ADVANCE` | —         | Transition from spinning to blocking.       |
| `EXHAUST` | `OUTCOME`   | `DEFECTIVE` | `END`     | `FAIL`    | Retry budget exceeded, operation abandoned. |

### Pooling

#### Resources

- **Instrument**: Resource (Signer)
- **Shape**: 6 signs, no dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — grant and denial are returned by the allocator (SPEC.md §8.1.1)
- **Reference**: `registry:resources`

| Sign      | Kind        | Status     | Operation | Outcome   | Meaning                                                                            |
|-----------|-------------|------------|-----------|-----------|------------------------------------------------------------------------------------|
| `ATTEMPT` | `OPERATION` | —          | `BEGIN`   | —         | A non-blocking request for units from a resource.                                  |
| `ACQUIRE` | `OPERATION` | —          | `BEGIN`   | —         | A blocking or wait-based request for units from a resource.                        |
| `GRANT`   | `OUTCOME`   | `STABLE`   | `ADVANCE` | `SUCCESS` | The successful granting of a request for units from a resource.                    |
| `DENY`    | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | The denial of a request due to insufficient resource capacity.                     |
| `TIMEOUT` | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | The expiration of a blocking request due to wait time exceeding configured limits. |
| `RELEASE` | `OPERATION` | —          | `END`     | —         | The return of previously granted resource units back to the resource pool.         |

#### Leases

- **Instrument**: Lease (Signaler)
- **Shape**: 9 signs × 2 dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — grant, expiry, and revocation are decided by the lease mechanism
  (SPEC.md §8.1.1)
- **Reference**: `registry:leases`

| Sign      | Kind        | Status      | Operation | Outcome   | Meaning                                                        |
|-----------|-------------|-------------|-----------|-----------|----------------------------------------------------------------|
| `ACQUIRE` | `OPERATION` | —           | `BEGIN`   | —         | A client is attempting to obtain a new lease.                  |
| `DENY`    | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | A lease request has been denied.                               |
| `EXTEND`  | `OUTCOME`   | `STABLE`    | `ADVANCE` | `SUCCESS` | The lease duration has been successfully extended.             |
| `EXPIRE`  | `OUTCOME`   | `DEGRADED`  | `END`     | `FAIL`    | The lease has automatically terminated due to TTL exhaustion.  |
| `GRANT`   | `OUTCOME`   | `STABLE`    | `ADVANCE` | `SUCCESS` | The lease has been successfully granted.                       |
| `PROBE`   | `OPERATION` | —           | `ADVANCE` | —         | A status check is being performed on a lease.                  |
| `RELEASE` | `OPERATION` | —           | `END`     | —         | The holder voluntarily terminated the lease before expiration. |
| `RENEW`   | `OPERATION` | —           | `ADVANCE` | —         | The holder is attempting to extend the lease duration.         |
| `REVOKE`  | `OPERATION` | `DEFECTIVE` | `END`     | —         | The lease authority has forcefully revoked a lease.            |

**Dimension** (Category)

| Dimension | Meaning                          |
|-----------|----------------------------------|
| `LESSOR`  | The lease authority perspective. |
| `LESSEE`  | The lease client perspective.    |

#### Pools

- **Instrument**: Pool (Signer)
- **Shape**: 4 signs, no dimensions
- **Properties**: KIND
- **Classification**: primary — expansion, borrowing, and reclamation are acts the pool performs
  (SPEC.md §8.1.1)
- **Reference**: `registry:pools`

| Sign       | Kind        | Meaning                                |
|------------|-------------|----------------------------------------|
| `EXPAND`   | `OPERATION` | The pool's capacity increased.         |
| `CONTRACT` | `OPERATION` | The pool's capacity decreased.         |
| `BORROW`   | `OPERATION` | A resource was borrowed from the pool. |
| `RECLAIM`  | `OPERATION` | A resource was reclaimed by the pool.  |

#### Exchanges

- **Instrument**: Exchange (Signaler)
- **Shape**: 2 signs × 2 dimensions
- **Properties**: KIND
- **Classification**: primary — contracting and transferring are acts a party performs (SPEC.md
  §8.1.1)
- **Reference**: `registry:exchanges`

| Sign       | Kind        | Meaning                               |
|------------|-------------|---------------------------------------|
| `CONTRACT` | `OPERATION` | Commit to participate in an exchange. |
| `TRANSFER` | `OPERATION` | Resource changes hands.               |

**Dimension** (Category)

| Dimension  | Meaning                                   |
|------------|-------------------------------------------|
| `PROVIDER` | The giving perspective in an exchange.    |
| `RECEIVER` | The receiving perspective in an exchange. |

### Data

#### Queues

- **Instrument**: Queue (Signer)
- **Shape**: 4 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — enqueue, dequeue, and the boundary conditions are witnessed (SPEC.md
  §8.1.1)
- **Reference**: `registry:queues`

| Sign        | Kind        | Status     | Meaning                                                        |
|-------------|-------------|------------|----------------------------------------------------------------|
| `ENQUEUE`   | `OPERATION` | `STABLE`   | An item was successfully added to the queue.                   |
| `DEQUEUE`   | `OPERATION` | —          | An item was successfully removed from the queue.               |
| `OVERFLOW`  | `OUTCOME`   | `DEGRADED` | The queue reached capacity and rejected an ENQUEUE operation.  |
| `UNDERFLOW` | `OUTCOME`   | —          | The queue was empty and could not satisfy a DEQUEUE operation. |

#### Stacks

- **Instrument**: Stack (Signer)
- **Shape**: 4 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — push, pop, and the boundary conditions are witnessed (SPEC.md
  §8.1.1)
- **Reference**: `registry:stacks`

| Sign        | Kind        | Status     | Meaning                                                    |
|-------------|-------------|------------|------------------------------------------------------------|
| `PUSH`      | `OPERATION` | `STABLE`   | An item was successfully added to the stack (top).         |
| `POP`       | `OPERATION` | —          | An item was successfully removed from the stack (top).     |
| `OVERFLOW`  | `OUTCOME`   | `DEGRADED` | The stack reached capacity and rejected a PUSH operation.  |
| `UNDERFLOW` | `OUTCOME`   | —          | The stack was empty and could not satisfy a POP operation. |

#### Caches

- **Instrument**: Cache (Signer)
- **Shape**: 7 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — hit, miss, eviction, and expiry are decided by the cache's own
  machinery (SPEC.md §8.1.1)
- **Reference**: `registry:caches`

| Sign     | Kind        | Status     | Meaning                                                         |
|----------|-------------|------------|-----------------------------------------------------------------|
| `LOOKUP` | `OPERATION` | —          | An attempt to retrieve an entry from the cache.                 |
| `HIT`    | `OUTCOME`   | `STABLE`   | A cache lookup succeeded - the requested entry was found.       |
| `MISS`   | `OUTCOME`   | `DEGRADED` | A cache lookup failed - the requested entry was not found.      |
| `STORE`  | `OPERATION` | —          | An entry was added to or updated in the cache.                  |
| `EVICT`  | `OUTCOME`   | —          | An entry was automatically removed due to capacity or policy.   |
| `EXPIRE` | `OUTCOME`   | —          | An entry was removed because it reached its time-to-live (TTL). |
| `REMOVE` | `OPERATION` | —          | An entry was explicitly removed or invalidated.                 |

#### Pipelines

- **Instrument**: Pipeline (Signer)
- **Shape**: 12 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: derived — `LAG` is chosen by measuring progress against an expectation
  (SPEC.md §8.1.1)
- **Reference**: `registry:pipelines`

| Sign           | Kind        | Status     | Meaning                                |
|----------------|-------------|------------|----------------------------------------|
| `INPUT`        | `OPERATION` | —          | Data was received from upstream stage. |
| `OUTPUT`       | `OPERATION` | `STABLE`   | Data was sent to downstream stage.     |
| `TRANSFORM`    | `OPERATION` | —          | Data was transformed.                  |
| `FILTER`       | `OPERATION` | —          | Data was filtered out.                 |
| `AGGREGATE`    | `OPERATION` | —          | Data was aggregated.                   |
| `BUFFER`       | `OPERATION` | —          | Data was buffered.                     |
| `BACKPRESSURE` | `OPERATION` | —          | Backpressure was applied.              |
| `OVERFLOW`     | `OUTCOME`   | `DEGRADED` | Buffer overflow occurred.              |
| `CHECKPOINT`   | `OPERATION` | —          | Processing checkpoint reached.         |
| `WATERMARK`    | `OPERATION` | —          | Event-time watermark advanced.         |
| `LAG`          | `OUTCOME`   | `DEGRADED` | Processing lag detected.               |
| `SKIP`         | `OUTCOME`   | —          | Data was skipped or dropped.           |

#### Messages

- **Instrument**: Message (Signaler)
- **Shape**: 7 signs × 3 dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — every sign is a delivery step or settlement the messaging system
  itself presents: an acknowledgement, a redelivery flag, a delivery limit reached, an expiry
  (SPEC.md §8.1.1)
- **Reference**: `registry:messages`

| Sign        | Kind        | Status      | Meaning                                                                                 |
|-------------|-------------|-------------|-----------------------------------------------------------------------------------------|
| `PUBLISH`   | `OPERATION` | —           | A message was handed on for delivery.                                                   |
| `DELIVER`   | `OPERATION` | —           | A message was handed to a receiver.                                                     |
| `ACK`       | `OUTCOME`   | `STABLE`    | The receiving end confirmed the message.                                                |
| `NACK`      | `OUTCOME`   | `DEGRADED`  | The receiving end refused or failed the message.                                        |
| `REDELIVER` | `OPERATION` | —           | A message was delivered again because an earlier delivery was refused or never settled. |
| `EXHAUST`   | `OUTCOME`   | `DEFECTIVE` | A message's delivery attempts ran out, and it was set aside or dropped.                 |
| `EXPIRE`    | `OUTCOME`   | `DEGRADED`  | A message's time to live ran out before delivery.                                       |

**Dimension** (Category)

| Dimension  | Meaning                                                            |
|------------|--------------------------------------------------------------------|
| `PRODUCER` | Emitted by the party that publishes messages.                      |
| `BROKER`   | Emitted by the intermediary that holds messages and delivers them. |
| `CONSUMER` | Emitted by the party that receives and processes messages.         |

### Flow control

#### Flows

- **Instrument**: Flow (Signaler)
- **Shape**: 2 signs × 3 dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — each stage reports the success or failure it observed (SPEC.md
  §8.1.1)
- **Reference**: `registry:flows`

| Sign      | Kind      | Status     | Meaning                                |
|-----------|-----------|------------|----------------------------------------|
| `SUCCESS` | `OUTCOME` | `STABLE`   | Successful transition through a stage. |
| `FAIL`    | `OUTCOME` | `DEGRADED` | Failed transition through a stage.     |

**Dimension** (Category)

| Dimension | Meaning                                       |
|-----------|-----------------------------------------------|
| `INGRESS` | Emitted at the entry point of a flow.         |
| `TRANSIT` | Emitted during processing/movement of a flow. |
| `EGRESS`  | Emitted at the exit point of a flow.          |

#### Routers

- **Instrument**: Router (Signer)
- **Shape**: 9 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: derived — `REORDER` requires remembering the order in which packets were sent
  (SPEC.md §8.1.1)
- **Reference**: `registry:routers`

| Sign         | Kind        | Status      | Meaning                                         |
|--------------|-------------|-------------|-------------------------------------------------|
| `SEND`       | `OPERATION` | —           | A packet was transmitted from this router.      |
| `RECEIVE`    | `OPERATION` | —           | A packet was received by this router.           |
| `FORWARD`    | `OPERATION` | `STABLE`    | A packet was forwarded to the next hop.         |
| `ROUTE`      | `OPERATION` | —           | A routing decision was made for a packet.       |
| `DROP`       | `OPERATION` | `DEGRADED`  | A packet was discarded.                         |
| `FRAGMENT`   | `OPERATION` | —           | A packet was fragmented due to MTU constraints. |
| `REASSEMBLE` | `OPERATION` | —           | Packet fragments were reassembled.              |
| `CORRUPT`    | `OUTCOME`   | `DEFECTIVE` | Packet corruption was detected.                 |
| `REORDER`    | `OUTCOME`   | —           | Out-of-order packet arrival was detected.       |

#### Valves

- **Instrument**: Valve (Signer)
- **Shape**: 6 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — admission decisions and capacity changes are acts the valve performs
  (SPEC.md §8.1.1)
- **Reference**: `registry:valves`

| Sign       | Kind        | Status      | Meaning                                        |
|------------|-------------|-------------|------------------------------------------------|
| `PASS`     | `OUTCOME`   | `STABLE`    | A request was allowed through the valve.       |
| `DENY`     | `OUTCOME`   | `DEGRADED`  | A request was blocked by the valve.            |
| `EXPAND`   | `OPERATION` | —           | The valve increased its capacity.              |
| `CONTRACT` | `OPERATION` | —           | The valve decreased its capacity.              |
| `DROP`     | `OPERATION` | `DEFECTIVE` | A request was dropped due to overload.         |
| `DRAIN`    | `OPERATION` | —           | The valve is clearing backlog during recovery. |

#### Breakers

- **Instrument**: Breaker (Signer)
- **Shape**: 6 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — trip, probe, and reset are transitions the breaker itself makes
  (SPEC.md §8.1.1)
- **Reference**: `registry:breakers`

| Sign        | Kind        | Status      | Meaning                                                       |
|-------------|-------------|-------------|---------------------------------------------------------------|
| `CLOSE`     | `OUTCOME`   | `STABLE`    | The circuit is closed and operating normally.                 |
| `OPEN`      | `OUTCOME`   | `DEFECTIVE` | The circuit is open and blocking requests.                    |
| `HALF_OPEN` | `OUTCOME`   | —           | The circuit is half-open and testing recovery.                |
| `TRIP`      | `OUTCOME`   | `DEGRADED`  | The failure threshold was exceeded, triggering circuit break. |
| `PROBE`     | `OPERATION` | —           | A test request is being sent in half-open state.              |
| `RESET`     | `OPERATION` | —           | The circuit was manually reset to closed state.               |

#### Guards

- **Instrument**: Guard (Signaler)
- **Shape**: 4 signs × 2 dimensions
- **Properties**: STATUS, KIND, OUTCOME
- **Classification**: primary — each sign is the guard's own decision or the request it was given
  (SPEC.md §8.1.1)
- **Reference**: `registry:guards`

| Sign        | Kind        | Status     | Outcome   | Meaning                                              |
|-------------|-------------|------------|-----------|------------------------------------------------------|
| `ATTEMPT`   | `OPERATION` | —          | —         | A credential or request was presented.               |
| `CHALLENGE` | `OPERATION` | —          | —         | Further proof was required, such as a second factor. |
| `GRANT`     | `OUTCOME`   | `STABLE`   | `SUCCESS` | Access was allowed.                                  |
| `DENY`      | `OUTCOME`   | `DEGRADED` | `FAIL`    | Access was refused.                                  |

**Dimension** (Category)

| Dimension    | Meaning                                             |
|--------------|-----------------------------------------------------|
| `IDENTITY`   | The check of who the caller is: authentication.     |
| `PERMISSION` | The check of what the caller may do: authorization. |

### Instrumentation

#### Counters

- **Instrument**: Counter (Signer)
- **Shape**: 3 signs, no dimensions
- **Properties**: KIND
- **Classification**: primary — increment, overflow, and reset are witnessed (SPEC.md §8.1.1)
- **Reference**: `registry:counters`

| Sign        | Kind        | Meaning                                             |
|-------------|-------------|-----------------------------------------------------|
| `INCREMENT` | `OPERATION` | The counter was incremented.                        |
| `OVERFLOW`  | `OUTCOME`   | The counter exceeded its maximum value and wrapped. |
| `RESET`     | `OPERATION` | The counter was explicitly reset to zero.           |

#### Gauges

- **Instrument**: Gauge (Signer)
- **Shape**: 5 signs, no dimensions
- **Properties**: KIND
- **Classification**: primary — increment, decrement, and the boundary conditions are witnessed
  (SPEC.md §8.1.1)
- **Reference**: `registry:gauges`

| Sign        | Kind        | Meaning                                     |
|-------------|-------------|---------------------------------------------|
| `INCREMENT` | `OPERATION` | The gauge was incremented.                  |
| `DECREMENT` | `OPERATION` | The gauge was decremented.                  |
| `OVERFLOW`  | `OUTCOME`   | The gauge exceeded its maximum value.       |
| `UNDERFLOW` | `OUTCOME`   | The gauge fell below its minimum value.     |
| `RESET`     | `OPERATION` | The gauge was explicitly reset to baseline. |

#### Probes

- **Instrument**: Probe (Signaler)
- **Shape**: 6 signs × 2 dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — connection, transfer, and their outcomes are witnessed (SPEC.md
  §8.1.1)
- **Reference**: `registry:probes`

| Sign         | Kind        | Status     | Operation | Outcome   | Meaning                                                                 |
|--------------|-------------|------------|-----------|-----------|-------------------------------------------------------------------------|
| `CONNECT`    | `OPERATION` | —          | `BEGIN`   | —         | Connection establishment.                                               |
| `DISCONNECT` | `OPERATION` | —          | `END`     | —         | Connection closure.                                                     |
| `TRANSFER`   | `OPERATION` | —          | `ADVANCE` | —         | Data transfer (sending or receiving, direction specified by dimension). |
| `PROCESS`    | `OPERATION` | —          | `ADVANCE` | —         | Data processing.                                                        |
| `SUCCEED`    | `OUTCOME`   | `STABLE`   | `ADVANCE` | `SUCCESS` | Successful completion.                                                  |
| `FAIL`       | `OUTCOME`   | `DEGRADED` | `ADVANCE` | `FAIL`    | Failed completion.                                                      |

**Dimension** (Category)

| Dimension  | Meaning                                                      |
|------------|--------------------------------------------------------------|
| `OUTBOUND` | Outbound communication (initiated by self, sending outward). |
| `INBOUND`  | Inbound communication (received from other, coming inward).  |

#### Sensors

- **Instrument**: Sensor (Signaler)
- **Shape**: 3 signs × 3 dimensions
- **Properties**: KIND
- **Classification**: derived — every sign is a comparison of a measured value against a configured
  reference (SPEC.md §8.1.1)
- **Reference**: `registry:sensors`

| Sign      | Kind      | Meaning                                                  |
|-----------|-----------|----------------------------------------------------------|
| `BELOW`   | `OUTCOME` | The measured value is below the setpoint reference.      |
| `NOMINAL` | `OUTCOME` | The measured value is at or near the setpoint reference. |
| `ABOVE`   | `OUTCOME` | The measured value is above the setpoint reference.      |

**Dimension** (Category)

| Dimension   | Meaning                                                                            |
|-------------|------------------------------------------------------------------------------------|
| `BASELINE`  | The baseline setpoint represents normal or expected operating level.               |
| `THRESHOLD` | The threshold setpoint represents a limit or boundary that should not be exceeded. |
| `TARGET`    | The target setpoint represents an ideal or optimal operating point.                |

#### Logs

- **Instrument**: Log (Signer)
- **Shape**: 4 signs, no dimensions
- **Properties**: STATUS, KIND
- **Classification**: derived — a severity is a judgment the producer forms about its own message
  (SPEC.md §8.1.1)
- **Reference**: `registry:logs`

| Sign      | Kind      | Status      | Meaning                                      |
|-----------|-----------|-------------|----------------------------------------------|
| `SEVERE`  | `OUTCOME` | `DEFECTIVE` | A serious failure or error condition.        |
| `WARNING` | `OUTCOME` | `DEGRADED`  | A potential problem or concerning condition. |
| `INFO`    | `OUTCOME` | `STABLE`    | Normal operational information.              |
| `DEBUG`   | `OUTCOME` | —           | Diagnostic or tracing information.           |

#### Evals

- **Instrument**: Eval (Signaler)
- **Shape**: 5 signs × 8 dimensions
- **Properties**: STATUS, KIND, OUTCOME
- **Classification**: derived — a criterion verdict judges work the producer did not itself perform
  (SPEC.md §8.1.1)
- **Reference**: `registry:evals`

| Sign      | Kind      | Status     | Outcome   | Meaning                                                                                |
|-----------|-----------|------------|-----------|----------------------------------------------------------------------------------------|
| `PASS`    | `OUTCOME` | `STABLE`   | `SUCCESS` | The subject satisfied the criterion.                                                   |
| `FAIL`    | `OUTCOME` | `DEGRADED` | `FAIL`    | The subject violated the criterion.                                                    |
| `UNKNOWN` | `OUTCOME` | —          | `UNKNOWN` | The criterion was evaluated but the verdict was indeterminate.                         |
| `SKIP`    | `OUTCOME` | —          | —         | The criterion was deliberately not evaluated.                                          |
| `ERROR`   | `OUTCOME` | —          | `UNKNOWN` | Evaluation of the criterion could not complete because the judge errored or timed out. |

**Dimension** (Category)

| Dimension      | Meaning                                                |
|----------------|--------------------------------------------------------|
| `CORRECTNESS`  | Factual or functional correctness.                     |
| `COMPLETENESS` | Coverage of the information or work required.          |
| `GROUNDEDNESS` | Support in authoritative evidence or supplied context. |
| `HELPFULNESS`  | Usefulness in accomplishing the request or objective.  |
| `ADHERENCE`    | Compliance with instructions, formats, and schemas.    |
| `TOOLING`      | Quality of tool selection, arguments, and use.         |
| `ROUTING`      | Quality of delegation, handoff, and context transfer.  |
| `SAFETY`       | Compliance with safety requirements.                   |

### Coordination

#### Agents

- **Instrument**: Agent (Signaler)
- **Shape**: 10 signs × 2 dimensions
- **Properties**: STATUS, KIND, OPERATION, OUTCOME
- **Classification**: primary — offers, promises, and their fulfilment or breach are acts a party
  performs or observes (SPEC.md §8.1.1)
- **Reference**: `registry:agents`

| Sign       | Kind        | Status     | Operation | Outcome   | Meaning                                                   |
|------------|-------------|------------|-----------|-----------|-----------------------------------------------------------|
| `OFFER`    | `OPERATION` | —          | `BEGIN`   | —         | An agent is advertising a capability (promise available). |
| `PROMISE`  | `OPERATION` | —          | `ADVANCE` | —         | An agent is making a promise about its own behavior.      |
| `ACCEPT`   | `OPERATION` | —          | `ADVANCE` | —         | An agent is accepting another agent's promise.            |
| `FULFILL`  | `OUTCOME`   | `STABLE`   | `END`     | `SUCCESS` | An agent kept its promise.                                |
| `RETRACT`  | `OPERATION` | —          | `END`     | —         | An agent is retracting a promise before fulfillment.      |
| `BREACH`   | `OUTCOME`   | `DEGRADED` | `END`     | `FAIL`    | An agent failed to keep its promise.                      |
| `INQUIRE`  | `OPERATION` | —          | `ADVANCE` | —         | An agent is asking about available capabilities.          |
| `OBSERVE`  | `OPERATION` | —          | `ADVANCE` | —         | An agent is monitoring another agent's promise state.     |
| `DEPEND`   | `OPERATION` | —          | `ADVANCE` | —         | An agent is declaring explicit dependency on a promise.   |
| `VALIDATE` | `OPERATION` | —          | `ADVANCE` | —         | An agent is confirming a promise is still held.           |

**Dimension** (Category)

| Dimension  | Meaning                                                             |
|------------|---------------------------------------------------------------------|
| `PROMISER` | The emission of a promise signal from the agent's own perspective.  |
| `PROMISEE` | The reception of a promise signal from another agent's perspective. |

#### Actors

- **Instrument**: Actor (Signer)
- **Shape**: 11 signs, no dimensions
- **Properties**: KIND
- **Classification**: primary — each sign is a communicative act the participant performs (SPEC.md
  §8.1.1)
- **Reference**: `registry:actors`

| Sign          | Kind        | Meaning                                               |
|---------------|-------------|-------------------------------------------------------|
| `ASK`         | `OPERATION` | Seeking information, clarification, or guidance.      |
| `AFFIRM`      | `OPERATION` | Making a claim or judgment with confidence.           |
| `EXPLAIN`     | `OPERATION` | Providing reasoning, elaboration, or rationale.       |
| `REPORT`      | `OPERATION` | Conveying factual observations or findings.           |
| `REQUEST`     | `OPERATION` | Asking another actor to perform action at peer level. |
| `COMMAND`     | `OPERATION` | Directing another actor to act with authority.        |
| `ACKNOWLEDGE` | `OPERATION` | Confirming receipt, understanding, or agreement.      |
| `DENY`        | `OPERATION` | Disagreeing with or correcting a proposition.         |
| `CLARIFY`     | `OPERATION` | Refining, specifying, or disambiguating intent.       |
| `PROMISE`     | `OPERATION` | Committing to perform future action or deliver work.  |
| `DELIVER`     | `OPERATION` | Presenting completed work or fulfilled commitment.    |

#### Members

- **Instrument**: Member (Signaler)
- **Shape**: 7 signs × 2 dimensions
- **Properties**: STATUS, KIND
- **Classification**: primary — every sign is the membership protocol's own result; `SUSPECT` and
  `EVICT` are its failure detector's decisions, never a producer's comparison of heartbeat gaps or
  timeouts (SPEC.md §8.1.1)
- **Reference**: `registry:members`

| Sign      | Kind        | Status     | Meaning                                       |
|-----------|-------------|------------|-----------------------------------------------|
| `JOIN`    | `OPERATION` | —          | A member joined the group.                    |
| `LEAVE`   | `OPERATION` | —          | A member left the group gracefully.           |
| `SUSPECT` | `OUTCOME`   | `DEGRADED` | The failure detector marked a member suspect. |
| `REFUTE`  | `OUTCOME`   | `STABLE`   | A suspected member proved it was alive.       |
| `EVICT`   | `OUTCOME`   | —          | A member was declared failed and removed.     |
| `ELECT`   | `OUTCOME`   | `STABLE`   | A member became leader.                       |
| `RESIGN`  | `OPERATION` | —          | A leader stepped down.                        |

**Dimension** (Category)

| Dimension | Meaning                                    |
|-----------|--------------------------------------------|
| `SELF`    | Reported by the member the event concerns. |
| `PEER`    | Reported by another member of the group.   |

---

# Part III — Indexes

## Family × Vocabulary Matrix

Which vocabularies draw on which sign family. A vocabulary appears in a family's row if at least one
of its signs belongs to that family.

| Family      | Lexemes | Slots | Vocabularies drawing on it                                                                                 |
|-------------|---------|-------|------------------------------------------------------------------------------------------------------------|
| Acquisition | 18      | 32    | Atomics, Latches, Leases, Locks, Pools, Resources                                                          |
| Lifecycle   | 9       | 15    | Processes, Services, Tasks                                                                                 |
| Work        | 11      | 14    | Changes, Services, Tasks, Transactions                                                                     |
| Episode     | 5       | 6     | Changes, Operations, Transactions                                                                          |
| Verdict     | 19      | 30    | Agents, Atomics, Caches, Changes, Evals, Flows, Messages, Outcomes, Probes, Services, Timers, Transactions |
| Admission   | 11      | 14    | Changes, Guards, Pipelines, Routers, Services, Valves                                                      |
| Boundary    | 6       | 13    | Atomics, Counters, Gauges, Messages, Pipelines, Queues, Stacks, Systems                                    |
| Elasticity  | 3       | 5     | Pools, Valves                                                                                              |
| Contention  | 6       | 6     | Atomics, Locks, Transactions                                                                               |
| Transport   | 17      | 19    | Exchanges, Messages, Pipelines, Probes, Queues, Routers, Services, Stacks                                  |
| Storage     | 6       | 7     | Caches, Messages, Pipelines                                                                                |
| Recovery    | 11      | 12    | Breakers, Latches, Messages, Services, Transactions                                                        |
| Integrity   | 4       | 4     | Pipelines, Routers                                                                                         |
| Counting    | 3       | 5     | Counters, Gauges                                                                                           |
| Assessment  | 22      | 25    | Logs, Sensors, Situations, Statuses, Systems, Trends                                                       |
| Speech act  | 19      | 20    | Actors, Agents, Exchanges                                                                                  |
| Processing  | 3       | 3     | Pipelines, Probes                                                                                          |
| Membership  | 7       | 7     | Members                                                                                                    |

## Shared Lexeme Index

The 39 lexemes appearing in more than one vocabulary: 38 signs and one dimension. Signs and
dimensions are separate alphabets (SPEC.md §8.4), so they are indexed separately below — a lexeme
occurring in both would be two unrelated symbols, not a shared one.

### Signs

Entries marked in §4 carry different meanings across their vocabularies; the rest are consistent
reuse.

| Lexeme       | Vocabularies                                                                 |
|--------------|------------------------------------------------------------------------------|
| `ABANDON`    | Latches, Locks                                                               |
| `ACQUIRE`    | Leases, Locks, Resources                                                     |
| `ATTEMPT`    | Atomics, Guards, Locks, Resources                                            |
| `CONTRACT`   | Exchanges, Pools, Valves                                                     |
| `DELIVER`    | Actors, Messages                                                             |
| `DENY`       | Actors, Guards, Leases, Locks, Resources, Valves                             |
| `DISCONNECT` | Probes, Services                                                             |
| `DROP`       | Routers, Valves                                                              |
| `EVICT`      | Caches, Members                                                              |
| `EXHAUST`    | Atomics, Messages                                                            |
| `EXPAND`     | Pools, Valves                                                                |
| `EXPIRE`     | Caches, Leases, Messages, Services, Transactions                             |
| `FAIL`       | Atomics, Changes, Evals, Flows, Outcomes, Probes, Processes, Services, Tasks |
| `GRANT`      | Guards, Leases, Locks, Resources                                             |
| `INCREMENT`  | Counters, Gauges                                                             |
| `MISS`       | Caches, Timers                                                               |
| `NORMAL`     | Situations, Systems                                                          |
| `OVERFLOW`   | Counters, Gauges, Pipelines, Queues, Stacks                                  |
| `PASS`       | Evals, Valves                                                                |
| `PROBE`      | Breakers, Leases                                                             |
| `PROGRESS`   | Changes, Tasks                                                               |
| `PROMISE`    | Actors, Agents                                                               |
| `REJECT`     | Changes, Services, Tasks                                                     |
| `RELEASE`    | Latches, Leases, Locks, Resources                                            |
| `RESET`      | Breakers, Counters, Gauges, Latches                                          |
| `RESUME`     | Processes, Services, Tasks                                                   |
| `SCHEDULE`   | Services, Tasks                                                              |
| `SKIP`       | Evals, Pipelines                                                             |
| `STABLE`     | Statuses, Trends                                                             |
| `START`      | Changes, Processes, Services, Tasks, Transactions                            |
| `STOP`       | Processes, Services                                                          |
| `SUCCESS`    | Atomics, Flows, Outcomes, Services                                           |
| `SUSPEND`    | Processes, Services, Tasks                                                   |
| `TIMEOUT`    | Latches, Locks, Resources, Tasks                                             |
| `TRANSFER`   | Exchanges, Probes                                                            |
| `UNDERFLOW`  | Gauges, Queues, Stacks                                                       |
| `UNKNOWN`    | Evals, Outcomes                                                              |
| `WARNING`    | Logs, Situations                                                             |

### Dimensions

| Lexeme      | Vocabularies    |
|-------------|-----------------|
| `THRESHOLD` | Sensors, Timers |

The only dimension lexeme shared across vocabularies. §5.4 records what both intend and why they
nonetheless remain two distinct dimensions in two distinct sets.
