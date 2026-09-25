# The Serventis Specification

**Version 3.6.0** **Copyright © 2025–2026 William David Louth / Humainary**

---

## 1. Purpose

Serventis defines finite, typed vocabularies for reporting what a system did or observed. It also
defines how consumers can translate domain-specific observations into shared vocabularies for
operational status and urgency. This is **semiotic observability**: producers report classifications
with explicit meanings, and consumers interpret them without forcing every domain into one common
vocabulary.

Serventis separates reporting from assessment. In a *primary* vocabulary, the producer chooses every
part of an observation directly from a current act or result, without itself comparing against a
reference, measuring, correlating, remembering, or inferring a condition. If choosing any part
requires such work, the vocabulary is *derived* (§8.1.1). Translation is explicit, inspectable, and
deliberately partial, and the vocabularies are finite and fixed.

The specification is independent of any programming language, runtime, or transport mechanism. A
conformant implementation may be realized as an in-process library, a networked service, or any
combination thereof.

### 1.1 Conformance Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this specification are to be interpreted as described in RFC
2119 and RFC 8174 when, and only when, they appear in all capitals as shown here.

### 1.2 Scope and Layering

The Serventis specification distinguishes three layers, and only the first is specified normatively
here.

**Layer 1 — the language.** This layer defines symbols, signs, dimensions, signals, and finite fixed
vocabularies, including the category/spectrum distinction and the membership and meanings of the
universal and registered vocabularies. It defines instruments and their relationship to Substrates.
It also defines partial translation and abstention, and requires every component of an observation
in a *primary* vocabulary to be implementer-observable (§8.1.1). This is the normative content of
this specification, in §2 through §10.

**Layer 2 — interpretive policy.** Which status a given sign indicates, which episode role it takes,
whether it reports an act or a result, and how several readings reduce to one. These are judgments
about meaning, not structural facts about the language. Two projections may reasonably differ, and a
consumer that disagrees with a published reading is not thereby non-conformant. This layer is
documented — in Appendix B and in the accompanying registry — as guidance, and is **not normative**.

**Layer 3 — execution affordances.** Precomputed lookup tables, ordinal addressing, constant-time
resolution, interned signal spaces, and the concrete types that provide them. These are performance
mechanisms of a particular projection. They are described in Appendix A.2 and are **not normative**.

This division is deliberate and provisional. Standardizing layer 2 would freeze a set of
interpretive opinions before more than one projection has tested them, and standardizing layer 3
would impose one language's performance model on the rest. A future version of this specification
may promote parts of either layer once a second projection has demonstrated which of them are
genuinely portable. Until then, this specification defines the shared language and leaves each
projection to define how it interprets and executes that language.

### 1.3 Relationship to Substrates

Serventis is a **layered specification**. It defines vocabularies and instruments; it does not
define circulation. Emission, ordering, subscription, identity, and lifecycle are governed entirely
by the Substrates Specification, which Serventis presupposes.

**This version of Serventis requires Substrates 3.6.0**, pinned at
<https://github.com/humainary-io/substrates-api-spec/blob/3.6.0/SPEC.md>. Every reference to a
Substrates section in this document, and every `substrates:`-qualified traceability reference in a
projection of it, resolves against that version. A later Serventis version may require a later
Substrates version; a released one does not change what it requires.

A conformant Serventis implementation MUST be built over a conformant implementation of that
Substrates version, and MUST NOT weaken, reinterpret, or supplement any Substrates contract. In
particular:

- An instrument (§6) emits into a Substrates **Pipe**. All ordering, dispatch, and delivery
  guarantees for that emission are the Substrates guarantees, unchanged.
- Instrument pooling (§6.4) is the Substrates **Pool** contract: the same name resolves to the same
  instrument, materialized lazily on first lookup.
- Identity, naming, and subject semantics are the Substrates identity system. Serventis introduces
  no identity mechanism of its own.
- Interpretation of emitted observations is not specified here at all, and a projection may express
  it however it likes: a Substrates **Flow**, a generated switch, a native matching construct, or
  nothing. Where a projection *does* express an interpretation as a Flow, that Flow's contracts
  apply to it unchanged. The reference projection does so throughout (Appendix B).

Where this specification uses a term defined by Substrates — Pipe, Pool, Conduit, Flow, Window,
Circuit, Subject, Name — it uses it with the Substrates meaning. Section references of the form
"Substrates §N" refer to that specification.

The dependency is one-directional. Substrates has no knowledge of Serventis. Serventis relies on the
complete Substrates contract for its runtime behavior and adds no runtime primitive of its own. It
is a **vocabulary layer**, not a runtime.

### 1.4 Definitions

This specification uses the following abstract terms to remain independent of any specific runtime
model:

- **Observation**: The unit of semiotic expression. A sign, optionally qualified by a dimension.
- **Sign**: The principal semantic classification carried by an observation — what is being
  reported.
- **Dimension**: A secondary qualifier that gives a sign context — perspective, confidence, stage,
  criterion, or recurrence.
- **Signal**: A qualified observation — a sign together with a dimension.
- **Sign set**: The finite, indexed alphabet of signs belonging to one vocabulary.
- **Dimension set**: The finite, indexed alphabet of dimensions belonging to one vocabulary.
- **Vocabulary**: A named sign set, optionally paired with a dimension set, together with the
  instrument that emits it. Called a *domain* vocabulary when it describes a specific kind of
  subject, and *universal* when it is a target of translation.
- **Instrument**: The producer-side emission surface for one vocabulary — the object a producer
  holds and calls to express an observation.
- **Producer**: The component that holds an instrument and emits observations through it. A producer
  may be reporting its own behavior or reporting about something it is watching; the specification
  does not require either, and does not tie the distinction to the vocabulary being emitted
  (§8.1.1).
- **Consumer**: Any component that receives emitted observations and interprets them.
- **Projection**: A conformant realization of this specification in a particular programming
  language or runtime. A projection chooses representations and may provide non-normative
  conveniences beyond the required contract.
- **Translation**: A mapping of observations in one vocabulary onto observations in another,
  necessarily partial (§5.1).
- **Ascent**: Translation from a domain vocabulary toward a universal one, and from a universal
  vocabulary toward a higher one.
- **Absent value**: A sentinel indicating "no value." A translation abstains on a sign by yielding
  the absent value. Each language projection maps this to its idiomatic representation. This
  specification uses the `?` suffix in abstract signatures to denote a value that may be absent.
  Non-normative examples of idiomatic mappings: `null` in Java/C#, `None` in Python, `Option::None`
  in Rust, a nil interface or sum type in Go, `undefined` in JavaScript, `Nothing` in Haskell.
- **Canonical identity**: As defined by Substrates §1.2. Two references designate the same abstract
  entity, testable through the projection's standard equality mechanism.

## 2. Design Principles

Four governing principles shape the specification.

**Bounded vocabularies over open ontologies.** Every sign set is finite and indexed, and its members
are fixed when the vocabulary is published. A vocabulary whose membership can grow at runtime cannot
be checked for coverage, cannot be exhaustively interpreted, and offers a consumer no way to know it
has seen the whole alphabet.

**Implementer-observable signs.** In a *primary* vocabulary — one whose every observation can be
selected from direct observation alone — the producer is never asked to compute a rate, compare
against a baseline, correlate with another component, remember what it emitted before, or infer a
condition. It names an act it performed or a result it received, and nothing more. Recognition of
pattern is the consumer's work; the producer's work is faithful report. Every other vocabulary is
*derived*, and the rule does not apply to it; §8.1.1 draws the line from the vocabulary's own sets
and holds it narrow.

**Partial translation as a feature.** Translation between vocabularies is deliberately partial. A
sign that carries no reading in a target vocabulary yields the absent value rather than a default,
and the absence is meaningful: it says this observation supports no conclusion in that vocabulary.
Forcing totality would manufacture readings the evidence does not support.

**Ascent over flattening.** Domain vocabularies are not required to share a common sign set. They
are instead *encouraged* to be translatable toward the universal vocabularies — a recommendation
(§8.1.4), not a conformance requirement. Meaning is composed hierarchically — domain to status to
situation — rather than by forcing every domain into one vocabulary that fits none of them well.

## 3. Structural Overview

A Serventis deployment has four cooperating parts.

**Vocabularies** are the alphabets. A vocabulary publishes a sign set and optionally a dimension
set. Vocabularies divide into *domain* vocabularies, which describe a particular kind of subject (a
lock, a cache, a transaction), and *universal* vocabularies, which are the shared targets of
translation.

**Instruments** are the producer-side emission surfaces. An instrument wraps a Substrates Pipe and
exposes exactly one vocabulary. A **Signer** emits bare signs; a **Signaler** emits signals. A
producer holds an instrument and calls it.

**Circulation** is Substrates. Observations emitted by an instrument travel the owning circuit's
ordered path to subscribed consumers, with the ordering, dispatch, and lifecycle guarantees defined
there and nowhere restated here.

**Interpretation** translates circulated observations toward the universal vocabularies. This
specification defines what translation *is* (§5) and what the universal vocabularies *are* (§7), but
does not mandate any particular translation mechanism. Appendix B describes the mechanisms the
reference projection provides.

The ascent hierarchy is:

```text
Domain vocabulary          (Locks, Caches, Transactions, …)
      │
      ├── per-sign reading ────────▶ evidence is the sign itself   ─┐
      │                                                             │
      └── structural interpretation ▶ evidence is the sign's        │
                                      position among others       ─┤
                                                                    ▼
                                          Statuses  (universal operational reading)
                                                                    │
                                                                    ▼
                                          Situations (universal urgency reading)
```

The two modes are distinguished by what supplies the evidence, not by what kind of sign is read
(§5.3); a sign may feed either or both. They converge on the same shared vocabulary, which lets a
consumer reason across domains it was never specifically written for. This specification defines
the modes; it mandates a mechanism for neither.

## 4. The Observation Model

### 4.1 Symbol

A **symbol** is the base classification marker. Both signs and dimensions are symbols.

A symbol MUST carry, within its set:

- a **name** — a stable textual identifier, unique within its set;
- an **index** — a stable non-negative integer position within its set, dense from zero.

A projection MUST guarantee that a symbol's index is stable for the lifetime of the process, that
indices within one set are contiguous from zero, and that no two symbols in one set share an index.

Symbols MUST be comparable for canonical identity. A projection MAY supply symbol identity through
enumeration members, interned tokens, nominal type metadata, or string tags; the model requires only
a distinct, index-carrying value per member.

### 4.2 Sign

A **sign** is a symbol that classifies what is being reported. It is the principal semantic content
of an observation.

A sign MUST belong to exactly one sign set. The same textual name MAY appear in more than one sign
set, and when it does the two signs are distinct signs that happen to share a name (§8.4). A sign
carries no meaning independent of its set.

### 4.3 Dimension

A **dimension** is a symbol that qualifies a sign.

A dimension MUST belong to exactly one dimension set. As with signs (§4.2), the same textual name
MAY appear in more than one dimension set, and when it does the two are distinct dimensions:
`THRESHOLD` in one vocabulary and `THRESHOLD` in another are not the same symbol and MUST NOT be
unified (§8.4). Their meanings may well coincide — a vocabulary author reusing a name usually
intends exactly that — but the coincidence is not something an implementation may rely on or infer.

Every published dimension set MUST have a **uniform kind**: either all of its members are categories
or all of them are spectra. The kind is a property of the set, not of individual members, because it
says whether the set's index order carries meaning — a question that cannot have different answers
for different members of one ordering.

This is an obligation on the vocabulary, checkable when it is published, not a claim about what a
projection's type system can express. A projection MAY carry the kind on the set, on its members, or
on both, and MAY be unable to make a mixed set unrepresentable; where it cannot, it SHOULD detect
and reject one at set construction, and a conformance suite SHOULD test for it (§10.4).

There are exactly two kinds.

A **category** is an unordered classification whose members are peers. Relative position carries no
meaning: the caller perspective is not "more than" the callee perspective, and a space constraint is
not "greater than" a time constraint.

A **spectrum** is an ordered progression in which relative position carries meaning. Spectrum
members MUST be declared in ascending order, so that a member's index is its rank: the first member
is the low end, the last is the high end. A consumer MAY compare spectrum members by index; a
consumer MUST NOT compare category members by index for any purpose other than storage addressing.

A projection MUST make the category/spectrum distinction visible to consumers, so that an
interpretation can determine whether ordering is meaningful without knowing the specific dimension
set.

### 4.4 Observation and Signal

An **observation** is either *atomic* or *qualified*.

An atomic observation is a bare sign; the sign is the observation. A qualified observation — a
**signal** — is a sign together with a dimension. A signal is therefore a *qualified sign*.

Every observation MUST expose a sign. A qualified observation MUST additionally expose its
dimension. A consumer needing only the classification MUST be able to obtain the sign uniformly: for
an atomic observation the sign is the observation itself; for a qualified observation it is the
observation's sign component.

Whether a projection expresses this unification as a subtype relationship, a tagged union, an
accessor contract, or a generic emission type carrying a caller-supplied sign projection is a
projection concern (§A.2). The normative requirement is that the sign of any observation is
obtainable.

Signals MUST be immutable, and MUST compare equal when their sign and dimension components are
equal.

### 4.5 Sign Sets and Dimension Sets

A **sign set** is a finite, indexed, ordered collection of signs. A **dimension set** is the same
over dimensions.

Both MUST satisfy:

1. **Finite** — the member count is known when the set is published.
2. **Indexed** — the set is an alphabet of exactly `size` members, each carrying a distinct index,
   running contiguously from zero to `size - 1`.
3. **Fixed** — membership does not change after publication. Adding a member is a new version of the
   vocabulary, not a mutation of the existing one (§8.3).

These are conceptual properties of the set. In particular, being indexed is not an obligation to
expose traversal: §10.3 requires no traversal, indexing, or member-lookup operation, and a
projection that exposes none still conforms.

A vocabulary MUST publish its sign set as part of its public surface, so that whatever mechanism a
projection provides for working over an alphabet can be applied to it without reflection or
hard-coded member lists.

A projection SHOULD expose some means of visiting a set's members — a traversal, an index lookup, or
a projection-building operation such as the one in §B.2. Without one, a consumer cannot be written
generically over an arbitrary vocabulary, which limits what can be built on the projection. This is
a recommendation rather than a requirement because the useful shape of such an operation varies too
much between languages to specify one here.

### 4.6 The Signal Space

For a sign set `S` and a dimension set `D`, the **signal space** is the Cartesian product `S × D`.
Because both sets are finite, the signal space is finite, with cardinality `|S| × |D|`.

The signal space is a property of the vocabulary, not a required construct. An implementation MAY
precompute and intern it, construct signals on demand, or anything between. Interning is invisible
to the model: two references to the same `(sign, dimension)` observation MUST compare equal whether
or not they have been interned (§4.4).

## 5. Translation

**Translation** maps observations in one vocabulary onto observations in another. It is how a
domain-specific report becomes a reading in a language a general consumer understands.

This section defines what translation is and the constraints any translation MUST satisfy. It does
not define specific translations, nor mandate a mechanism for expressing them. Which status a
particular sign indicates is interpretive policy (§1.2, layer 2); the mechanisms the reference
projection provides for declaring and applying translations are described in Appendix B.

### 5.1 Partial Translation and Abstention

A translation is **partial**: it need not assign a target reading to every sign in the source
vocabulary. A sign for which the translation yields the absent value is said to **abstain**.

Abstention is a meaningful outcome and MUST be preserved as one. It means: *this sign supports no
conclusion in the target vocabulary.* It does not mean "unknown", "neutral", "not yet computed", or
"assume the default".

A consumer that reads a translation *as the reading of the sign itself* MUST treat abstention as a
directive to produce nothing. It MUST NOT substitute a default reading, and MUST NOT count an
abstaining sign as evidence for any reading.

This constraint governs the reading of a sign in isolation. It does not constrain an interpretation
that derives a conclusion from **structure** — from a sign's position in a sequence, from the shape
of a trace, or from any evidence other than the per-sign translation. Where the evidence is
structural, the conclusion does not come from the translation, and abstention in the translation
neither supplies nor forbids it.

**Abstention is not indeterminacy.** A sign that abstains yields the absent value. A sign that
reports a genuine but indeterminate result translates to the `UNKNOWN` member of the outcome
vocabulary (§7.4), which is a reading like any other. A lock contention that resolves neither way is
`UNKNOWN`; a lock release, which is simply not a verdict, abstains. An implementation MUST
distinguish the two and MUST NOT substitute one for the other.

### 5.2 Operations and Outcomes

Every sign is grammatically either an operation or an outcome.

An **operation** reports an act the producer performed: a request, a lifecycle transition, a
flow-control move, a recovery action. An **outcome** reports a result the producer observed: a
verdict, a terminal condition, a boundary result. Operations ordinarily lead to outcomes.

The distinction is **grammatical**: it asks only "act or result?" of what the sign reports, and is
decided from the sign's own meaning rather than from any consumer's reading of it. It is therefore
independent of what a sign translates to, and an implementation MUST NOT derive either from the
other.

The distinction is **set-relative**. The same textual name MAY take different kinds in different
vocabularies: a clean `STOP` reporting that a process exited is an outcome, while a bracketing
`STOP` reporting the act of stopping is an operation. The kind belongs to the sign in its set, not
to the word (§8.4).

The distinction matters because the two kinds tend to carry evidence differently. An outcome is
usually self-contained: a single `DENY` supports a reading on its own. An operation usually is not:
one `CALL` supports no reading, because calls are what services do, and the information lies in what
followed it or in what conspicuously did not. This tendency is why translation is partial (§5.1) —
most operations have no reading to give in isolation.

It is a tendency and not a rule, and this specification does not turn it into one. The kind of a
sign MUST NOT be taken to determine how that sign may be interpreted. A sign whose kind is
`OPERATION` may still carry a reading in isolation, where the act is itself the verdict of something
that already went wrong — a forced revocation, a load-shedding drop. A sign whose kind is `OUTCOME`
may still be significant chiefly for its position, where the outcome is mid-episode rather than
terminal — a grant that advances an acquisition, a compare-and-swap failure that will be retried.
Both cases are common across the registered vocabularies, not exceptional.

Whether a projection materializes this distinction as a value a consumer can query, and how a
vocabulary declares which of its signs are which, is not specified here. Appendix B.2 describes the
reference projection's approach.

### 5.3 Ascent

**Ascent** is translation toward a universal vocabulary. The universal vocabularies (§7) are the
shared targets, and a domain vocabulary participates in the architecture by being translatable
toward them.

Ascent proceeds in one of two **modes**, and the modes are distinguished by what supplies the
evidence, not by what kind of sign is being read:

- **Per-sign reading** takes the evidence from the sign itself. This is translation in the sense of
  §5.1, and abstention applies to it directly: a sign the translation declines to read contributes
  nothing.
- **Structural interpretation** takes the evidence from a sign's position among others — the order
  of a sequence, the shape of a trace, the completion or abandonment of an episode. The per-sign
  translation may inform it but does not supply its conclusion, which is why §5.1's no-default rule
  does not reach it.

Any sign may participate in either mode, and the same sign may participate in both. Which signs a
given interpretation reads, and in which mode, is that interpretation's policy — this specification
defines the modes and constrains their handling of abstention, and specifies neither a mechanism nor
an assignment of signs to modes. Appendix B describes what the reference projection provides.

A vocabulary author SHOULD ensure at least one mode is available to consumers — that enough of its
signs can be read into `Statuses` individually, or that its signs form an episode structure a
consumer can interpret, or both. A vocabulary that supports neither cannot ascend and is of limited
use in this architecture.

Ascent is lossy by design. A `DENY` and a `TIMEOUT` may both read as `DEGRADED`; the distinction
between them is preserved in the domain observation, which remains available to any consumer that
needs it. Ascent produces a reading, not a replacement.

## 6. Instruments

An **instrument** is the producer-side emission surface for one vocabulary. It is the only object a
producer needs in order to express observations, and it exposes exactly one vocabulary.

### 6.1 Signer

A **Signer** is the instrument protocol for an unqualified vocabulary. It exposes one required
operation:

```text
Signer<S>.sign(sign: S)
```

which emits the given sign as an atomic observation.

### 6.2 Signaler

A **Signaler** is the instrument protocol for a qualified vocabulary. It exposes one required
operation:

```text
Signaler<S, D>.signal(sign: S, dimension: D)
```

which emits the signal composed of the given sign and dimension.

### 6.3 Named Operations

An instrument SHOULD additionally expose one named operation per sign in its vocabulary, as a
convenience over the generic `sign` / `signal` operation. A named operation MUST be exactly
equivalent to the generic operation applied to that sign: `lock.grant()` and `lock.sign(GRANT)` MUST
be indistinguishable to every consumer.

Named operations are ergonomic surface, not semantic surface. A projection MAY omit them where the
host language makes the generic form idiomatic enough, but MUST NOT give a named operation behavior
the generic form does not have.

### 6.4 Construction and Pooling

An instrument is constructed over a Substrates Pipe accepting the vocabulary's emission type:

```text
Vocabulary.of(pipe: Pipe<Observation>): Instrument
```

A vocabulary MUST additionally provide a pooling factory that derives a named instrument pool from a
Substrates Conduit carrying the vocabulary's emission type:

```text
Vocabulary.pool(conduit: Conduit<Observation>): Pool<Instrument>
```

The returned pool is a Substrates Pool and inherits its contract unchanged: the same name resolves
to the canonically identical instrument, and each instrument is materialized lazily on first lookup
of its name.

A **vocabulary template** (§7.7) MUST accept the caller's sign set as an additional argument to both
factories:

```text
Template.of(signs: SignSet<S>, pipe: Pipe<Observation>): Instrument
Template.pool(signs: SignSet<S>, conduit: Conduit<Observation>): Pool<Instrument>
```

### 6.5 Emission Semantics

Emission through an instrument is emission through the underlying Substrates Pipe, with the
Substrates guarantees and no others. An instrument MUST NOT buffer, coalesce, reorder, drop, or
otherwise alter the sequence of observations a producer expresses.

An instrument MUST reject an absent sign or dimension as an absence violation (§9), rather than
emitting a partially formed observation.

An instrument MUST NOT accumulate episode state, counts, or history. Interpretation is the
consumer's work, and an instrument that accumulated state could not be shared safely across the
producers that hold it by name.

## 7. Universal Vocabularies

The universal vocabularies are the shared targets of ascent. A conformant implementation MUST
provide all of them, and MUST use exactly the sign and dimension sets specified here — these are the
shared language across domains and projections, and a projection that varies them is not
interoperable with any other.

Six are vocabularies proper (§7.1–§7.6). Two — `Surveys` and `Cycles` — are **vocabulary templates**
(§7.7): they fix a dimension set and take their sign set from the caller.

What each vocabulary's signs *mean* is specified here. What any *other* vocabulary's signs translate
*into* them is interpretive policy and is not specified (§1.2). The accompanying registry records
each vocabulary's members in full.

### 7.1 Statuses

`Statuses` is the universal operational reading — the primary ascent target for domain vocabularies.

**Sign set** (7): `CONVERGING`, `STABLE`, `DIVERGING`, `ERRATIC`, `DEGRADED`, `DEFECTIVE`, `DOWN`.

**Dimension set** (3, spectrum): `TENTATIVE`, `MEASURED`, `CONFIRMED` — the confidence with which
the reading is offered, rising with the weight of evidence behind it.

The sign set defines **no intrinsic ordering or combination rule**. The seven signs describe three
different things: `CONVERGING` and `DIVERGING` describe trajectory; `STABLE`, `DEGRADED`,
`DEFECTIVE`, and `DOWN` describe level; `ERRATIC` describes variance. The vocabulary therefore
defines neither a lattice nor a total order, and a conformant implementation MUST NOT treat it as
defining one.

Reducing several status readings to one therefore requires a stated policy. An implementation that
performs such a reduction MUST document the policy it uses, and in particular whether that policy
can outvote a reading such as `DOWN` when it is held by a minority. Appendix B.3 describes the
reduction the reference projection provides.

Normalizing a *single* reading — mapping one status onto another, as a recognizer does when it
reports what an episode's outcome amounts to — is not a reduction and this requirement does not
reach it.

The dimension is a spectrum, so its members are ordered by confidence and a consumer MAY compare
them by index.

### 7.2 Situations

`Situations` is the universal urgency reading — the second-order ascent target, consumed by whoever
must decide whether to act.

**Sign set** (3): `NORMAL`, `WARNING`, `CRITICAL`.

**Dimension set** (3, spectrum): `CONSTANT`, `VARIABLE`, `VOLATILE` — the variability of the
condition being assessed.

This specification does not define a translation from status into situation, and does not constrain
what evidence such a translation may rest on — persistence, confidence, correlation across subjects,
or anything else. That choice is interpretive policy (§1.2). See §B.6 for why the reference
projection provides no such operator.

### 7.3 Operations

`Operations` is the universal episode vocabulary, used to express the structure of an episode
independently of the domain in which it occurred.

**Sign set** (3): `BEGIN`, `ADVANCE`, `END`. No dimensions.

`BEGIN` opens an episode; `ADVANCE` progresses within an open episode without opening or closing it;
`END` closes an open episode. Only `END` closes.

### 7.4 Outcomes

`Outcomes` is the universal verdict vocabulary.

**Sign set** (3): `SUCCESS`, `FAIL`, `UNKNOWN`. No dimensions.

`UNKNOWN` is a **verdict-bearing** member: it reports that a result was reached and was
indeterminate. It MUST NOT be used to represent abstention, which is the absent value (§5.1).

### 7.5 Systems

`Systems` is the universal constraint vocabulary, describing which class of system limit a condition
implicates.

**Sign set** (4): `NORMAL`, `LIMIT`, `ALARM`, `FAULT`.

**Dimension set** (4, category): `SPACE`, `FLOW`, `LINK`, `TIME` — the kind of constraint
implicated.

### 7.6 Trends

`Trends` is the universal shape vocabulary, describing the form of a series over time rather than
its level.

**Sign set** (5): `STABLE`, `DRIFT`, `SPIKE`, `CYCLE`, `CHAOS`. No dimensions.

`Trends.STABLE` and `Statuses.STABLE` are distinct signs in distinct sets (§8.4): the former reports
the shape of a series, the latter an operational reading.

### 7.7 Vocabulary Templates

`Surveys` (§7.8) and `Cycles` (§7.9) are not vocabularies but **vocabulary templates**: each fixes a
dimension set and leaves the sign set open, so that the qualification it expresses can be applied to
any sign set a caller already has.

A template MUST fix its dimension set at publication, exactly as a vocabulary does. It MUST NOT
publish a sign set, and the obligation of §4.5 to publish one, and the corresponding entry in the
required static surface (§10.3), do not apply to it.

A template is not itself an alphabet and nothing is emitted through it directly. **Materializing** a
template — supplying a sign set to one of its factories (§6.4) — yields an ordinary vocabulary whose
sign set is that one, and every requirement of §4.5 applies to the result from that point on: for
the lifetime of the materialized instrument the sign set is finite, indexed, and fixed. Two
materializations of the same template over different sign sets are two different vocabularies that
happen to share a dimension set.

A conformant implementation MUST provide both templates, with exactly the dimension sets specified
below.

### 7.8 Surveys

`Surveys` is the agreement template: it qualifies a caller-supplied sign by how far observers agreed
on it.

**Sign set**: supplied at materialization.

**Dimension set** (3, spectrum): `DIVIDED`, `MAJORITY`, `UNANIMOUS`.

### 7.9 Cycles

`Cycles` is the recurrence template: it qualifies a caller-supplied sign by how it recurs in a
stream.

**Sign set**: supplied at materialization.

**Dimension set** (3, category): `SINGLE` (first occurrence of this sign), `REPEAT` (same sign as
the immediately preceding observation), `RETURN` (seen before, but not immediately preceding).

## 8. Vocabulary Design

This section governs how vocabularies are defined. It applies to the universal vocabularies of §7,
to the domain vocabularies recorded in the registry, and to any vocabulary a projection adds.

### 8.1 Design Rules

A conformant vocabulary MUST satisfy the six rules of §8.1.1 through §8.1.6. Each is a stable
subsection: rules are cited by their section number, and a future revision that adds a rule appends
it rather than renumbering the existing ones.

#### 8.1.1 Implementer-observable — primary vocabularies

A sign or dimension is **implementer-observable** when the producer can choose it directly from a
current act or result, without itself comparing against a reference, measuring, correlating,
remembering, or inferring a condition. A vocabulary is **primary** when every component of every
observation it can express is implementer-observable. It is **derived** when choosing any component
requires more: accumulated history, comparison with a configured reference, correlation with another
component, measurement, or judgment.

The categories are exhaustive and mutually exclusive. A vocabulary with directly observed signs but
a computed dimension is derived, because one derived component is enough.

The classification belongs to the vocabulary and is decided from the meanings of its sign and
dimension sets when the vocabulary is published. It does not depend on whether a producer reports
about itself or about another subject. An assessor can emit a primary vocabulary about something it
is watching; a subject can emit a derived vocabulary about itself.

In a primary vocabulary every component of every observation MUST be implementer-observable. The
vocabulary MUST NOT require the producer itself to compare against a reference, measure, correlate,
remember prior emissions, or infer a condition in order to choose one. Recognition of pattern
belongs to consumers.

The boundary is who performs the work. If the mechanism being observed has already performed a
comparison or measurement and presents its result to the producer, reporting that result is direct
observation. If the producer receives raw facts and must compare, measure, correlate, or remember in
order to choose a vocabulary member, the observation is derived.

`TIMEOUT` and `MISS` illustrate the boundary. A lock reporting `TIMEOUT` received the unsuccessful
result of its blocking acquisition; it reports that result directly. A timer reporting `MISS`
received an elapsed duration and a configured threshold and had to compare them to choose between
`MEET` and `MISS`. The first is primary and the second derived, although their names alone do not
reveal the difference.

"Derived" does not mean "interpretive opinion". It means only that the producer must do some of that
work itself. A recurrence dimension reporting whether a sign has appeared before is a fact about the
stream, but selecting it requires memory, so a vocabulary carrying one is derived.

The direct-observation requirement does not apply to derived vocabularies. Producing their
observations is precisely the work that distinguishes them from primary vocabularies.

Applying the test to the universal vocabularies: `Statuses`, `Situations`, `Systems`, and `Trends`
are derived, since nobody selects `DEGRADED` or `SPIKE` without weighing evidence. `Operations` and
`Outcomes` are primary — an episode role and a verdict are both witnessed.

The classification of each **registered domain vocabulary** is recorded in the registry alongside
its membership and is normative for that vocabulary (§8.2). Several classifications turn on one
member: `Routers` is derived because `REORDER` requires remembering packet order, and `Pipelines` is
derived because `LAG` requires measuring progress against an expectation. One component is enough.

A vocabulary template (§7.7) is classified by what it materializes into, having no sign set of its
own. Materializations of `Surveys` and `Cycles` are derived on the strength of their dimension
alone, whatever sign set they are given: agreement among observers must be gathered, and recurrence
must be remembered.

A vocabulary author MUST NOT declare a vocabulary derived in order to escape this rule. The test is
what selecting its observations requires, not what its author would prefer: a vocabulary whose
observations could have been selected from direct observation is primary whether or not it says so.

#### 8.1.2 Bounded

The sign set and dimension set MUST be finite, indexed, and fixed at publication (§4.5).

#### 8.1.3 Balanced

A vocabulary that expresses operations SHOULD also express the outcomes those operations lead to. A
vocabulary whose signs carry no reading in isolation leaves a consumer nothing to read per-sign, and
must rely entirely on structural interpretation (§5.3).

#### 8.1.4 Translatable

A vocabulary SHOULD be translatable toward at least one universal vocabulary (§5.3).

#### 8.1.5 Dimensions are perspective, not location

A dimension set MUST classify the observation, not locate it. Whose viewpoint, which role, which
criterion, what confidence — these are dimensions. Which datastore, which host, which region are
not; they are subject identity, and belong to the Substrates naming system.

Many vocabularies have no dimensions at all. A dimension set SHOULD be introduced only when the same
sign genuinely means something different under each member.

#### 8.1.6 Reuse before invention

A vocabulary SHOULD reuse an established sign family where the semantics align, and SHOULD add
domain-specific signs only where the family does not already say what needs saying.

### 8.2 Registration

Vocabularies divide into three tiers by conformance obligation.

**Universal vocabularies** (§7) are REQUIRED. Every conformant implementation MUST provide all of
them, with exactly the specified members.

**Registered domain vocabularies** are OPTIONAL as a whole but **fixed in membership, meanings, and
classification**. A projection need not provide any of them; a projection that provides a vocabulary
under a registered name MUST use exactly the sign set, the dimension set, the member meanings, and
the primary/derived classification recorded for it in the registry accompanying this specification.
Partial or varied membership under a registered name is non-conformant, because a consumer that
recognizes the name would misread it.

Classification is part of the contract rather than commentary because it decides whether §8.1.1's
implementer-observable requirement applies at all. A projection that reclassified a registered
vocabulary would be claiming a different obligation for the same name — asserting, for instance,
that `Timers` signs are directly observable when the registry records that selecting `MEET` over
`MISS` requires the producer to compare.

The obligation stops there. The **interpretive policy** the registry records alongside a vocabulary
— what its signs indicate, which episode role they take, whether they report acts or results — is
non-normative (§1.2), and a projection MAY publish different policy without affecting its
conformance.

**Projection-specific vocabularies** are unconstrained in membership but MUST satisfy the design
rules of §8.1, and MUST NOT use a registered name.

### 8.3 Versioning

A vocabulary's members are fixed at publication (§4.5), and so are their meanings (§8.2). Each of
the following is an **incompatible** change, producing a new version of the vocabulary rather than
an update to the existing one:

- adding, removing, or renaming a **sign or a dimension**;
- changing the index of a sign or a dimension, or reordering either set;
- changing a dimension set's kind between category and spectrum (§4.3), since that changes whether
  member order carries meaning;
- changing what a sign or a dimension **means** — the normative gloss by which a consumer interprets
  it (§8.2);
- changing a registered vocabulary's **primary/derived classification** (§8.1.1), since that changes
  which obligations its signs carry.

The rule is symmetric over signs and dimensions because a qualified vocabulary's signal space is
their product (§4.6): a change to either changes what a consumer receives. Two vocabularies that
differ in any of the above are different vocabularies and MUST NOT claim the same version.

Changing an **interpretive reading** — what a sign is said to indicate, which episode role it takes,
whether it reports an act or a result — is a **compatible** change. Those are opinions about the
signs rather than the signs themselves, they are non-normative (§1.2, §8.2), and consumers that
depend on them consult them dynamically.

### 8.4 Homographs

The same textual name MAY appear in more than one sign set, or in more than one dimension set.
Symbols that do are **distinct symbols that share a name**, and this holds whether their meanings
differ, overlap, or coincide exactly: sharing a name establishes no relationship between them.

A conformant implementation MUST NOT treat such symbols as interchangeable, unify them across sets,
or infer one vocabulary's interpretation of a name from another's — and MUST NOT do so on the
grounds that the two are documented as meaning the same thing. Two dimension sets may both define
`THRESHOLD` for a configured limit, and an implementation still may not substitute one for the
other.

A consumer MUST resolve a symbol against the set that produced it. The registry records the known
homographs across the registered vocabularies.

## 9. Absence and Error Model

Serventis introduces no error mechanism of its own. Error signaling, isolation of consumer
callbacks, and closed-resource behavior are Substrates concerns (Substrates §15) and apply unchanged
to instrument emission.

Two Serventis-specific constraints apply.

**Absence violations.** A sign or dimension argument to an instrument operation MUST NOT be absent.
An implementation MUST reject an absent argument as an absence violation, using whatever mechanism
its projection uses for Substrates absence violations, rather than emitting a partially formed
observation.

**Meaningful absence.** The absent value yielded by a translation is **not** an error. It is
abstention (§5.1), and an implementation MUST NOT convert it into an error, a default value, or a
diagnostic.

## 10. Conformance

A conformant implementation MUST satisfy the behavioral requirements, provide the required types and
operations, and provide every universal vocabulary.

Conformance is assessed against this specification only. Appendix A and Appendix B are
non-normative, and an implementation that provides none of Appendix B's extensions is fully
conformant.

### 10.1 Behavioral Requirements

The requirements below are identified by their **bold name**, not by their position in the list. A
future revision may add, remove, or reorder them; a citation such as "§10.1 *Status ordering is
policy*" therefore remains stable where a positional one would not.

1. **Substrates conformance**: The implementation MUST be built over a conformant implementation of
   the required Substrates version (§1.3), and MUST NOT weaken or supplement any Substrates
   contract.
2. **Bounded vocabularies**: Every published sign set and dimension set MUST be finite, densely
   indexed from zero, and fixed after publication (§4.5).
3. **Symbol identity**: Symbol names MUST be unique within a set, and symbol indices MUST be stable,
   dense, and unique within a set (§4.1).
4. **Dimension kinds**: Every dimension MUST be identifiable as a category or a spectrum, and the
   distinction MUST be visible to consumers. Every dimension MUST be exactly one of the two, and
   every published dimension set MUST be uniform — all categories or all spectra. Spectrum members
   MUST be ordered by rank (§4.3).
5. **Sign obtainability**: The sign of any observation, atomic or qualified, MUST be obtainable by a
   consumer (§4.4).
6. **Signal equality**: Signals MUST be immutable and MUST compare equal when their components are
   equal, whether or not the signal space is interned (§4.4, §4.6).
7. **Abstention preserved**: A consumer reading a translation as the reading of a sign MUST produce
   nothing where the translation abstains, and MUST NOT default or count it (§5.1).
8. **Abstention distinct from UNKNOWN**: An implementation MUST distinguish the absent value
   (abstention) from the `UNKNOWN` outcome member (an indeterminate verdict), and MUST NOT
   substitute one for the other (§5.1).
9. **Kind independence**: An implementation MUST NOT derive whether a sign reports an operation or
   an outcome from what that sign translates to, or the reverse (§5.2).
10. **Named-operation equivalence**: A named instrument operation MUST be indistinguishable from the
    generic operation applied to the same sign (§6.3).
11. **Instrument transparency**: An instrument MUST NOT buffer, coalesce, reorder, or drop
    observations, and MUST NOT accumulate interpretation state (§6.5).
12. **Instrument pooling**: A vocabulary's pool factory MUST return a Substrates Pool, with that
    contract unchanged: same name, canonically identical instrument, materialized lazily (§6.4).
13. **Universal vocabulary fidelity**: Every universal vocabulary MUST be provided with exactly the
    sign set and dimension set specified in §7. Every vocabulary template MUST be provided with
    exactly the dimension set specified in §7, MUST NOT publish a sign set, and MUST yield a
    vocabulary satisfying §4.5 when materialized (§7.7, §8.2).
14. **Status ordering is policy**: The `Statuses` sign set defines no lattice, total order, or
    reduction operation, and an implementation MUST NOT treat it as defining one. An implementation
    that reduces several status readings to one MUST document the policy by which it does so; an
    implementation that provides no such reduction has nothing to document (§7.1).
15. **Primary vocabularies are implementer-observable**: Every component of every observation a
    primary vocabulary can express — sign and dimension alike — MUST be selectable by the producer
    directly from a current act or result, without itself comparing against a reference, measuring,
    correlating, remembering, or inferring a condition. A vocabulary is derived exactly when it is
    not primary; the two are exhaustive. Which one a vocabulary is follows from its sign and
    dimension sets, not from who emits them (§8.1.1).
16. **Registered vocabulary fidelity**: A vocabulary published under a registered name MUST use
    exactly the registered sign set and dimension set, with the registered meanings for their
    members, and MUST carry the registered primary/derived classification — which determines whether
    requirement *Primary vocabularies are implementer-observable* applies to it. Interpretive policy
    is exempt (§8.2).
17. **Homograph separation**: An implementation MUST NOT unify symbols that share a textual name
    across distinct sets — signs across sign sets, dimensions across dimension sets — regardless of
    whether their documented meanings coincide (§8.4).

### 10.2 Required Types

A conformant implementation MUST provide realizations of the following abstract types.

| Type      | Category   | Description                                |
|-----------|------------|--------------------------------------------|
| Symbol    | Model      | Base classification marker: name and index |
| Sign      | Model      | Principal semantic classification          |
| Dimension | Model      | Secondary qualifier; category or spectrum  |
| Category  | Model      | Unordered dimension kind                   |
| Spectrum  | Model      | Ordered dimension kind                     |
| Signal    | Model      | Qualified observation: sign × dimension    |
| SymbolSet | Set        | Finite indexed symbol alphabet             |
| SignSet   | Set        | Finite indexed sign alphabet               |
| Signer    | Instrument | Unqualified emission surface               |
| Signaler  | Instrument | Qualified emission surface                 |

Notes:

- `SymbolSet` and `SignSet` are the bounded-alphabet types. A projection MAY unify them where its
  type system makes the distinction unnecessary, provided §4.5 is satisfied.
- Instrument classes for individual vocabularies (Status, Lock, Cache, …) are not listed
  individually: they are the vocabulary's realization of `Signer` or `Signaler` (§6).
- Types supporting interpretation — projection tables, translation and sequencing operators, and any
  materialized operation/outcome classifier — are **not** required. Appendix B describes those the
  reference projection provides.

### 10.3 Required Operations

The signatures use abstract notation: `Type.operation(param: ParamType): ReturnType`. A return type
suffixed with `?` may be the absent value (§1.4). Type parameters are written `<T>`. Operations with
no return value omit the return type.

**Symbol**

```text
Symbol.name(): String
Symbol.index(): Integer
```

**Signal** (also Observation)

```text
Signal<S, D>.sign(): S
Signal<S, D>.dimension(): D
```

**SymbolSet**

```text
SymbolSet<X>.size(): Integer
```

**SignSet** (also SymbolSet)

```text
SignSet<S>.size(): Integer
```

How a set is *constructed* is projection-specific and is not specified here: the enumeration
metadata, generated table, or literal list a projection builds one from has no portable form. What
is required is that every published set satisfies §4.5 and that each vocabulary exposes its own, per
the **Vocabulary** entry below.

**Signer**

```text
Signer<S>.sign(sign: S)
```

**Signaler**

```text
Signaler<S, D>.signal(sign: S, dimension: D)
```

**Vocabulary** (static surface of every vocabulary)

```text
Vocabulary.SIGNS: SignSet<S>
Vocabulary.DIMENSIONS: SymbolSet<D>                     -- qualified vocabularies only
Vocabulary.of(pipe: Pipe<Observation>): Instrument
Vocabulary.pool(conduit: Conduit<Observation>): Pool<Instrument>
```

**Vocabulary template** (§7.7). A template publishes no `SIGNS`, and takes the caller's sign set as
a leading argument to both factories:

```text
Template.DIMENSIONS: SymbolSet<D>
Template.of(signs: SignSet<S>, pipe: Pipe<Observation>): Instrument
Template.pool(signs: SignSet<S>, conduit: Conduit<Observation>): Pool<Instrument>
```

No traversal operation is required on either set type (§4.5), and no projection or operator surface
is required at all.

### 10.4 Conformance Testing

A conformance suite SHOULD verify, at minimum:

- Every universal vocabulary's sign and dimension sets match §7 exactly, in membership and order.
- Every vocabulary published under a registered name matches the registry's membership exactly.
- Symbol indices are dense, stable, and unique within each set.
- Every dimension is identifiable as a category or a spectrum, is exactly one of the two, and every
  published dimension set is uniform in kind. Where a projection cannot make a mixed set
  unrepresentable, a set constructed from one is rejected.
- The sign of a qualified observation is obtainable, and signals with equal components compare
  equal.
- Named instrument operations are indistinguishable from the generic operation.
- Instrument pools satisfy the Substrates Pool contract, and instruments carry no accumulated state.
- Instruments reject absent sign and dimension arguments.
- `Outcomes.UNKNOWN` is a member of the outcome sign set and is distinguishable from the
  projection's representation of the absent value.

Where a projection exposes a translation surface at all, a suite SHOULD additionally verify that a
consumer reading a translation produces nothing where the translation abstains. A projection may
conformantly expose no translation type, mapping surface, or operator, in which case there is
nothing through which to exercise that test and it does not apply.

A suite testing an Appendix B extension SHOULD keep those tests separable from the above, so that an
implementation providing no extensions can run the conformance suite.

## Appendix A. Projection Guidance (Non-Normative)

This appendix summarizes the representation choices left to a projection (§1.4). The specification
constrains behavior; projections choose representation.

### A.1 Binding Obligations vs. Implementation Freedoms

| Topic                  | What the spec requires (normative)                            | What projections may choose freely                                     |
|------------------------|---------------------------------------------------------------|------------------------------------------------------------------------|
| **Symbol identity**    | Stable name and dense stable index within a set (§4.1)        | Enum members, interned tokens, string tags, generated constants        |
| **Set representation** | Finite, densely indexed, fixed (§4.5)                         | Enum metadata, static arrays, generated tables, frozen slices          |
| **Set traversal**      | Nothing — no traversal or lookup is required (§4.5)           | Whether to expose an iterator, indexer, or nothing at all              |
| **Dimension kinds**    | Category/spectrum distinction visible to consumers (§4.3)     | Marker interfaces, traits, a tag field, separate types                 |
| **Observation unity**  | Sign obtainable from any observation (§4.4)                   | Subtyping, tagged union, accessor contract, caller-supplied projection |
| **Signal space**       | Finite; equal components imply equal signals (§4.6)           | Interned table, per-emission construction, flyweight cache             |
| **Absent value**       | Abstention distinct from `UNKNOWN`; never an error (§5.1, §9) | `null`, `Option`, `Maybe`, nil, sum types                              |
| **Translation**        | Partial; abstention preserved (§5.1)                          | Whether and how translations are declared, stored, and applied         |
| **Kind**               | Grammatical, set-relative, not derived from reading (§5.2)    | Whether it is materialized as a queryable value at all                 |
| **Named operations**   | Equivalent to the generic form when present (§6.3)            | Whether to generate them at all; naming convention                     |

### A.2 The Java Projection

The Java API, `io.humainary.serventis`, is the reference projection of this specification. The
following decisions are **non-normative** — other projections are not bound by them.

**Model binding:**

- `Symbol`, `Sign`, and `Dimension` are nested interfaces on the `Serventis` type; `Symbol` is
  sealed, permitting `Sign` and `Dimension`, and `Dimension` is sealed, permitting `Category` and
  `Spectrum`.
- Signs and dimensions are Java `enum` types. Enum membership supplies finiteness and fixedness;
  `name()` supplies the symbol name and `ordinal()` supplies the symbol index. The `Symbol`
  interface declares exactly these two operations, which every enum already satisfies.
- A qualified observation is a `record` implementing `Signal<S, D>`, with `sign()` and `dimension()`
  components.
- `Signal` is **not** a subtype of `Sign`. An atomic sign's identity is its ordinal, but a composite
  `(sign, dimension)` cannot supply a single ordinal without its dimension set's cardinality, so the
  subtype relation is not expressible through Java's enum mechanism. An interpretation that must
  read the sign of an observation which may be atomic or qualified is therefore generic over the
  emission type, obtaining the sign through a caller-supplied projection — identity for an atomic
  sign, `Signal::sign` for a qualified one. This is a projection accommodation, not a weakening of
  §4.4: a projection whose identity mechanism does not depend on ordinal indexing may express the
  unification as a direct subtype.

**Set binding:**

- `SymbolSet.of(Class)` and `SignSet.of(Class)` read the enum constants through
  `Class::getEnumConstants`.
- Neither set type exposes public traversal. Members are reached through `SignSet.map`, the
  property-map builder (Appendix B.2), and `SignSet.signals`, the signal-space builder described
  under execution affordances below; both visit every member exactly once in ordinal order.

**Execution affordances** (specification layer 3, §1.2):

- The signal space is interned. `SignalSet` precomputes every `(sign, dimension)` pair at
  construction and resolves a lookup as `signals[sign.ordinal() * columns + dimension.ordinal()]`,
  so emission of a signal costs a table read rather than an allocation.
- Per-sign projections are precomputed. `SignMap` and `SignalMap` are flat arrays addressed by
  ordinal, built by applying a mapping function exactly once per member at construction, never per
  lookup. Both implement `java.util.function.Function`, so a projection is directly usable as an
  argument to an operator.
- These are performance mechanisms, not model requirements. A projection whose vocabularies are
  small, or whose runtime makes a match expression cheaper than an array read, may reasonably do
  none of this.

**Vocabulary binding:**

- Each vocabulary is a `final` utility class with a private constructor, implementing `Serventis` so
  that the Substrates and Serventis nested types are in scope unqualified.
- The sign set is `public static final SignSet<Sign> SIGNS`; the dimension set, where present, is
  `public static final SymbolSet<Dimension> DIMENSIONS`.
- Instrument classes are `static final` nested classes named for the singular subject (`Status`,
  `Lock`, `Cache`), with private constructors reached through the vocabulary's `of` factory.

**Absence binding:**

- Absence is `null` throughout, for abstention and for optional arguments alike.
- `NullPointerException` for absence violations, matching the Substrates projection.

**Annotation binding:**

- `@Abstract`, `@Extension`, `@Immutable`, `@Provided`, `@Queued`, `@New`, `@NotNull`, and
  `@Utility` are the Substrates vocabulary annotations, applied to the Serventis surface with the
  same meanings.
- `@SpecDoc` is the neutral annotation from `humainary-specs-api` — an artifact belonging to neither
  this specification nor Substrates — naming by URL the document against which the bare identifiers
  of `@SpecRef`s in its lexical scope resolve. It asserts no conformance of its own — a file
  carrying it may hold nothing but projection affordances — and its URL pins a release tag, so
  released source resolves against the specification text it was written for. Every Serventis source
  file carries it on its outermost type, naming this specification:

  ```java
  @SpecDoc ( "https://github.com/humainary-io/serventis-api-spec/blob/3.6.0/SPEC.md" )
  public final class Locks { … }
  ```

- `@SpecRef` is the companion traceability annotation from the same artifact. It defines a grammar —
  an identifier, optionally qualified by a **namespace** token — and resolves an unqualified
  identifier against the `@SpecDoc` in scope: the annotated type itself, else its nearest lexically
  enclosing annotated type. In a Serventis file that is this specification.

So a Serventis declaration cites this specification with a bare identifier, exactly as a Substrates
declaration cites the Substrates specification with one:

```java
@SpecRef("4.5")                    // Serventis §4.5, on a Serventis declaration
@SpecRef({"6.1", "6.4"})           // two sections of this specification
```

Two qualified forms exist for what a bare identifier cannot express:

| Reference              | Addresses                                                               | Example           |
|------------------------|-------------------------------------------------------------------------|-------------------|
| `registry:<entry>`     | a registered domain vocabulary's entry in this specification's registry | `registry:locks`  |
| `substrates:<section>` | a section of the Substrates specification version this one requires     | `substrates:10.1` |

**Both resolve to pinned documents.** A qualified reference would otherwise be less permanent than
the bare identifiers around it, which resolve through a `@SpecDoc` naming an immutable revision:

- `registry:<entry>` resolves to `REGISTRY.md` **alongside the `SPEC.md` that the enclosing
  `@SpecDoc` names, at the same revision**. The two documents are published together and versioned
  together, so the enclosing `@SpecDoc` pins the registry as well; no second URL is needed.
- `substrates:<section>` resolves to the Substrates specification at **the version this
  specification requires** (§1.3), not at whatever version is current. A Serventis release states
  that version, and it does not change for a released Serventis version.

`registry:` exists because a registered domain vocabulary's normative content lives in the
accompanying registry rather than in a numbered section here. The entry identifier is the
vocabulary's name in lower case, and is stable: an entry is never renamed, because a rename would be
a different vocabulary (§8.3). The namespace is **reserved for the registered domain vocabularies**
— the universal vocabularies and templates appear in the registry only as mirrors, their authority
is §7, and they are cited with a bare section identifier. Where a registry mirror and §7 disagree,
§7 governs.

`substrates:` is for the layering seam. A Serventis declaration whose contract is inherited from
Substrates — instrument pooling is the Pool contract, emission is the Pipe contract (§1.3) — cites
it explicitly, because a bare identifier there would resolve to this specification and mean
something else. These citations are rare: this specification refers to Substrates in a handful of
places, and the annotations mirror that.

Section identifiers address numbered sections and subsections. They do not address items within a
numbered list: the requirements of §10.1 are identified by name rather than position, so an
annotation bound to one names §10.1 and identifies the requirement in prose.

The scheme is applied: the Java projection annotates its portable declarations and the TCK methods
that verify them, and every identifier resolves against one of three pinned documents — this one,
the registry accompanying it, or the Substrates specification this version requires (§1.3).
Declarations realizing only the non-normative material of Appendix A.2 and Appendix B carry a
`@SpecDoc` and no `@SpecRef`, which is how a projection says a type claims no portable coverage.

**Packaging:**

- `io.humainary.serventis.api` — the observation model.
- `io.humainary.serventis.sdk` — set and projection types, universal vocabularies, translation.
- `io.humainary.serventis.sdk.meta` — recurrence and sequencing.
- `io.humainary.serventis.opt.*` — registered domain vocabularies, grouped by subject area.

The `opt` prefix marks the tier: these vocabularies are optional to provide and fixed in membership
and meanings (§8.2). A projection targeting a narrow domain may ship the `api` and `sdk` tiers alone
and remain conformant.

## Appendix B. Standard Extensions (Non-Normative)

### B.1 Status of this Appendix

This appendix describes mechanisms the reference projection provides for declaring interpretive
policy and applying it to streams of observations. **None of it is normative.** An implementation
that provides none of these extensions is fully conformant (§10).

The material is recorded here rather than omitted because it is the working answer to a real
question — how does a consumer get from a domain observation to a status reading? — and because a
second projection benefits from a described starting point even where it is not bound to it.

It sits outside the normative core for two reasons. The property classifications are *opinions*
about meaning, and freezing them across languages before a second projection has tested them would
standardize an untested judgment. The operators bake in particular reduction policies — a decay
factor, a confidence band, a bracket state machine — which are tuning decisions, not language facts.

A future version of this specification may promote parts of this appendix once experience from
another projection shows which are genuinely portable. What that would take is stated in §B.7.

### B.2 Property Maps

The reference projection attaches interpretive policy to a vocabulary as **property maps**: total or
partial functions from the vocabulary's sign set into a standard value set, published as part of the
vocabulary's static surface and evaluated once per sign at construction rather than per observation.

Four are defined:

| Map         | Target                 | Totality | Question answered                |
|-------------|------------------------|----------|----------------------------------|
| `KIND`      | `{OPERATION, OUTCOME}` | total    | Is this sign an act or a result? |
| `STATUS`    | `Statuses.Sign?`       | partial  | What does this sign indicate?    |
| `OPERATION` | `Operations.Sign`      | total    | Where does it sit in an episode? |
| `OUTCOME`   | `Outcomes.Sign?`       | partial  | What did it decide?              |

`KIND` materializes the operation/outcome distinction of §5.2 as a queryable value. It is total: a
vocabulary publishing it classifies every sign, and the reference projection makes that a
compile-time obligation by switching over the sign enum without a default arm, so adding a sign
without classifying it fails to compile.

`OPERATION` is total, and a vocabulary that publishes one declares itself **bracketable**: its signs
can be read as an episode structure. The invariant the reference projection maintains is that a sign
mapped to `BEGIN` has kind `OPERATION` — an outcome cannot open an episode — while an outcome may
map to `ADVANCE` where it is mid-episode rather than terminal. Not every vocabulary is bracketable:
one whose episodes nest, interleave, or change shape with the dimension is better left without an
`OPERATION` map than given a flat one that misrepresents it.

`STATUS` and `OUTCOME` are partial, and their abstention carries the meaning §5.1 gives it. Neither
is required to abstain on anything: a vocabulary in which every sign carries a reading publishes a
map that is total in fact while remaining partial in kind.

The four maps are independent declarations over one sign set, and the reference projection does not
derive any from another. `KIND` asks what a sign *is*; `STATUS` what it *indicates*; `OPERATION`
where it *sits*; `OUTCOME` what it *decided*. The seams between them are informative: a `REVOKE` is
an operation that nonetheless reads `DEFECTIVE`, because a forced revocation is itself the verdict
of something that already went wrong.

The registry accompanying this specification records the property maps published by every registered
vocabulary. Those tables are non-normative (§8.2).

### B.3 The Translation Operator

The reference projection ascends by **tally** over emissions, as a Substrates Flow:

```text
Scorecards.flow(ballot: (E) -> Statuses.Sign?): Flow<E, Statuses.Signal>
Scorecards.score(window: Window<E>, ballot: (E) -> Statuses.Sign?): Statuses.Signal?
```

The ballot reads the source vocabulary; passing a vocabulary's `STATUS` map is the canonical use.

The reduction is a **plurality** — the status with the greatest accumulated weight — and the emitted
signal's dimension is a confidence band derived from how decisively that plurality leads and how
much evidence has accumulated. A ballot returning the absent value contributes nothing and produces
nothing, per §5.1.

When two or more statuses hold the greatest weight, the reference projection breaks the tie by
choosing the one with the **lowest index in the `Statuses` sign set**, in both the streaming and the
windowed form. This is a deterministic tie-break carrying no meaning: the sign set is not ordered by
severity or by anything else (§7.1), so index order here is an arbitrary but stable choice made so
that equal evidence yields a repeatable answer. It is recorded because the winner is externally
observable, and a projection adopting this extension whole should produce the same one.

Being a tally, this operator legitimately outvotes a minority reading, including a `DOWN` one. That
is what a tally is for: it reports where the weight of evidence lies, not the worst thing seen. A
consumer that must not lose a severe reading should not aggregate with it, and this is the property
§7.1 and §10.1 *Status ordering is policy* require an implementation to document.

The sequencers (§B.4) are not an alternative reduction. They aggregate nothing: an admission yields
its own reading or none at all, and severity is preserved by the fact that a severe reading is
emitted when it occurs, not by any policy for combining readings.

The streaming form weights recent votes above older ones so that a recovered subject is eventually
read as recovered; the windowed form weights every member equally and yields the absent value for an
empty tally. The reference projection uses exponential decay with factor 0.95, a warm-up threshold
of 0.25 accumulated evidence, and band boundaries at 0.50 and 0.80 of the accumulated total. These
are tuning constants, not portable facts.

### B.4 The Sequencing Operators

The reference projection ascends **traces** by shape, reading what only position in a sequence can
prove — the release no acquisition preceded, the episode that never closed — and emitting the
universal status vocabulary, so both ascent modes converge on one language.

What a sequencer consumes is a stream of domain signs of any kind, read through an episode-role map.
An outcome-kind sign placed at `ADVANCE` or `END` is read structurally like any other; the kind of a
sign does not select this mode and does not exclude a sign from it (§5.2, §5.3).

Both forms presume their input is a **trace**: a sequential history whose order is causally
meaningful. Interleaving concurrent episodes onto one subject destroys that premise, and neither
operator attempts correlation to recover it.

Time is not part of either recognizer. They answer one question — what ordered shape did this trace
take? — and speak from sequence evidence only. Whether an episode has been open too long is a
liveness question; a producer that can observe a timeout expresses it as an ordinary sign, so the
temporal judgment enters the trace as sequence shape.

Because these operators read structure rather than per-sign meaning, they are the case §5.1
contemplates when it declines to extend the no-default rule to structural interpretation: an
abstaining sign in a structurally significant position may still yield a reading.

#### B.4.1 The Bracket Sequencer

```text
Sequencers.flow(operation: SignMap<S, Operations.Sign>,
                status:    SignMap<S, Statuses.Sign?>): Flow<S, Statuses.Sign?>
```

The second map is the vocabulary's `STATUS` map, not its `OUTCOME` map, for two reasons. `STATUS`
has the range to express severity, where `OUTCOME` has only three members, so every failed close
would flatten to `FAIL` and a degraded close would be indistinguishable from a defective one. And
`OUTCOME` abstains on every sign that is not verdict-bearing, which includes operation signs that
nonetheless close episodes — `REVOKE` in `Leases`, `KILL` in `Processes`.

Neither map is total over closing signs: `STATUS` abstains on several of them too. The difference is
that abstention is handled structurally here — the fold below assigns a reading to an abstaining
close from the fact that an episode completed, not from the sign's own meaning — and `STATUS`
carries the severity range that makes the non-abstaining cases worth distinguishing.

The operator carries one bit of state, whether an episode is open, and produces at most one reading
per admission. Its complete behavior:

| Episode role | Episode open? | Reading                       | Episode after |
|--------------|---------------|-------------------------------|---------------|
| `BEGIN`      | no            | `DIVERGING`                   | open          |
| `BEGIN`      | yes           | `DEFECTIVE` (prior abandoned) | open          |
| `ADVANCE`    | no            | absent                        | closed        |
| `ADVANCE`    | yes           | `advancing(status)`           | open          |
| `END`        | no            | `DEFECTIVE` (orphan close)    | closed        |
| `END`        | yes           | `closing(status)`             | closed        |

where the two severity folds are:

| Admitted sign's `STATUS`           | `advancing`  | `closing`   |
|------------------------------------|--------------|-------------|
| absent (abstains)                  | absent       | `STABLE`    |
| `STABLE`, `CONVERGING`             | `CONVERGING` | `STABLE`    |
| `DEGRADED`, `ERRATIC`, `DIVERGING` | `DEGRADED`   | `DEGRADED`  |
| `DEFECTIVE`, `DOWN`                | `DEFECTIVE`  | `DEFECTIVE` |

Both folds are single-value normalizations, not reductions in the sense of §7.1: each maps the one
`STATUS` value of the sign being admitted onto the reading the operator emits for that admission,
and neither ever combines readings. The operator's only state is the episode-open bit. A `DOWN`
advance followed by a clean close therefore produces two readings in sequence — `DEFECTIVE` then
`STABLE` — rather than one reduced reading that preserves the `DOWN`. A consumer that needs the
severe reading to persist beyond the admission that produced it must retain it itself.

Two rows deserve note. A clean `BEGIN` reads `DIVERGING` because opening an episode is a departure
from idle, and a clean `END` reads `STABLE` because completing one is a return to it; the trajectory
a healthy episode paints is `DIVERGING` → `CONVERGING` → `STABLE`. And an `END` whose sign abstains
in `STATUS` reads `STABLE` rather than nothing: the reading comes from the structural fact that an
episode completed, not from the sign's own meaning. This matters in practice — `Locks.RELEASE`,
`Resources.RELEASE`, `Leases.RELEASE`, `Tasks.CANCEL`, `Probes.DISCONNECT`, and `Agents.RETRACT` are
all registered closers that abstain.

#### B.4.2 The Machine Sequencer

```text
Sequencers.flow(root: SignMap<S, Transition<S>?>): Flow<S, Statuses.Sign?>
Sequencers.emit(status: Statuses.Sign?, next: SignMap<S, Transition<S>?>?): Transition<S>
```

States are sign maps; a transition names an optional status reading and an optional next state. The
root map is the initial state.

Recognition is eager: each admission reads the current state's map, and if a transition is found the
walk moves immediately and speaks that transition's reading, if any. An admission with no transition
leaves the state unchanged and reads nothing. A transition with no next state returns the walk to
the root. A transition with no status moves silently. Each admission produces at most one reading.

The machine carries all of the semantics and the operator is pure mechanism. A later reading may
supersede an earlier one; with a trajectory-aware vocabulary this is refinement rather than
invalidation, since each reading was the truth of the trace at its moment.

Cycles are deliberately not expressible through the construction interface — a state can only
reference states built before it. Permitting them would require mutable states, a registration step,
or a compilation pass, each of which replaces a value that is finished when constructed with one
that has a construction lifecycle. A leaf transition returning the walk to the root expresses "start
over", which covers the common case.

### B.5 The Recurrence Operator

```text
Cycles.flow(signs: SignSet<S>): Flow<S, Cycles.Signal<S>>
```

Qualifies each admitted sign by how it recurs: `SINGLE` where the sign has not been seen before,
`REPEAT` where it equals the immediately preceding sign, `RETURN` otherwise. The operator is
sign-scoped, so a consumer holding qualified observations projects to the sign upstream.

Unlike §B.3 and §B.4, this operator embodies no interpretive policy — recurrence is a fact about the
stream, not an opinion about meaning — and it is the most likely candidate for promotion into the
normative core.

### B.6 No Situation Operator

The reference projection provides no translation from `Statuses` into `Situations`, and the gap is
deliberate rather than unfinished work.

Every other operator in this appendix reads evidence that is already present in the stream: a sign,
or a sign's position among the signs around it. A situation reading is not like that. Whether a
degraded status is a `WARNING` or a `CRITICAL` depends on considerations the stream does not carry —
how long the condition has held, how much it matters to whoever is deciding, what else is degraded
at the same time, and what the consequence of acting or not acting would be. Those are properties of
a deployment and of a decision, not of the observations.

An operator would therefore have to take that judgment as configuration, and the configuration
would be the whole of the semantics. Shipping one under a shared name would suggest a portable
meaning that the parameters immediately take back.

Nothing prevents a projection from providing one, and §7.2 deliberately constrains neither the
evidence such a translation rests on nor the mechanism. A projection that provides one is
encouraged to document what it reads and how, since a consumer cannot infer it from the vocabulary.

### B.7 Adopting an Extension

A projection adopting any of this appendix is encouraged to adopt it whole rather than in part,
since the value of a shared extension is that a consumer can rely on it, and to document which
extensions it provides.

A projection that deliberately diverges — a different reduction policy, a different bracket reading,
a different property set — remains fully conformant (§10). It is encouraged to say so plainly, so
that consumers do not assume the reference behavior.

Promotion of any part of this appendix into the normative core requires evidence that it is portable
rather than merely available: at least two independent projections providing it, agreement on the
observable behavior rather than only the surface, and a demonstrated consumer need for identical
results across projections. Absent that, it stays here.
