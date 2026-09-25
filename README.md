# Serventis API Specification

This directory contains the language-neutral specification for Serventis — finite, typed
vocabularies through which observed systems report what they did or saw, layered over Substrates.

## Contents

- **[SPEC.md](SPEC.md)** — The Serventis Specification (Version 3.6.0)
- **[RATIONALE.md](RATIONALE.md)** — Design Rationale (non-normative companion to SPEC.md)
- **[REGISTRY.md](REGISTRY.md)** — The Vocabulary Registry: the sign and dimension taxonomy, and the
  registered vocabularies (membership, meanings, and classification normative; interpretive policy
  documented, not binding)

## Purpose

The specification defines finite, typed vocabularies for reporting what a system did or observed. It
also defines how consumers can translate domain-specific observations into shared vocabularies for
operational status and urgency.

A *primary* vocabulary lets a producer choose every sign and dimension directly from a current act
or result, without itself comparing against a reference, measuring, correlating, remembering, or
inferring a condition. A vocabulary is *derived* if choosing any component requires such work.
Consumers translate those observations through explicit, inspectable interpretation rather than
policy hidden in a query.

It is independent of any programming language, runtime, or transport mechanism.

## Scope

The specification is deliberately scoped to **the language**, not to its interpretation or its
execution. It covers:

- **The observation model** — Symbol, Sign, Dimension (Category and Spectrum), Signal, sign sets,
  dimension sets, and the signal space
- **Translation** — partial translation, abstention as a meaningful outcome, the operation/outcome
  distinction, and ascent
- **Instruments** — Signer and Signaler, their construction, pooling, and emission semantics
- **Universal vocabularies** — Statuses, Situations, Operations, Outcomes, Systems, Trends — and the
  two **vocabulary templates**, Surveys and Cycles, which fix a dimension set and take their sign
  set from the caller
- **Vocabulary design** — the design rules, the registration tiers, versioning, and homographs
- **Conformance requirements** — required types, operations, and behavioral invariants

Two things are documented but **not normative**, and live in appendices:

- **Interpretive policy** — which status a given sign indicates, which episode role it takes,
  whether it reports an act or a result. These are judgments, and standardizing them before a
  second projection has tested them would freeze untested opinion.
- **Execution affordances** — precomputed lookup tables, ordinal addressing, interned signal spaces.
  These are one runtime's performance model.

Appendix B describes the property maps and the translation, sequencing, and recurrence operators the
reference projection provides, and states what promotion into the normative core would require. An
implementation providing none of them is fully conformant.

## Relationship to Substrates

Serventis is a **vocabulary layer**, not a runtime. It defines vocabularies and instruments, and
does not define circulation. Emission, ordering, subscription, identity, and lifecycle are governed
entirely by the [Substrates Specification](https://github.com/humainary-io/substrates-api-spec),
which Serventis presupposes and never reinterprets.

It defines no operators either. Appendix B documents the translation, sequencing, and recurrence
operators the Java reference projection provides, as non-normative extensions built on Substrates
Flow; a projection that provides none of them is fully conformant.

The dependency is one-directional: Substrates has no knowledge of Serventis. Serventis relies on the
complete Substrates contract for runtime behavior and adds no runtime primitive of its own.

## Conformance Tiers

Vocabularies carry three different obligations (SPEC.md §8.2):

| Tier                | Obligation                                                                     |
|---------------------|--------------------------------------------------------------------------------|
| Universal           | REQUIRED — every projection provides all of them                               |
| Registered domain   | OPTIONAL to provide, but **fixed in membership, meanings, and classification** |
| Projection-specific | Unconstrained in membership, must satisfy the design rules                     |

That binds the sign set, the dimension set, what those members mean, and whether the vocabulary is
primary or derived — the last because it decides whether the implementer-observable rule applies. It
does not bind the interpretive policy a projection publishes alongside them.

A minimal conformant projection provides the observation model, the two set types, the two
instrument protocols, the six universal vocabularies, and both vocabulary templates. The registered
domain vocabularies are recorded in REGISTRY.md.

## Reference Projection

The Java projection of this specification can be found
[here](https://github.com/humainary-io/serventis-api-java). Appendix A.2 of the spec documents
Java-specific binding decisions that are non-normative for other language projections.

## License

Copyright © 2025–2026 William David Louth. Licensed under the Apache License, Version 2.0.
