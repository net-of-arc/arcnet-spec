# CORE — The Vocabulary Every Graph Needs

**Status:** Draft · **Layer:** 2 of 3 · **Bundle:** `core`

Narrative for the `core` bundle. **Nothing here is normative.** The vocabulary is
[`Class/`](Class/) and [`Property/`](Property/); `CORE.md` (mechanism) and `SCHEMA.md` (node
formats) at the spec root cover the rest of what this bundle needs explained.

**Classes:** `Source`, `Entity`, `Resource`, `Reference`, `Timeline`, `Property`, `Class`.
**Reused from:** nothing — this is the bundle every other bundle reuses.

## What the bundle is for

Every arcnet domain bundle adopts `core` and nothing composes without it. It supplies the five classes a knowledge graph needs regardless of genre, and the two classes the schema layer itself is written in:

| Class       | What it holds                                                                   |
| ----------- | ------------------------------------------------------------------------------- |
| `Source`    | one ingested document — the provenance origin every derived node traces back to |
| `Entity`    | a durable subject occurring in sources, typed by Sowa category                  |
| `Resource`  | a fragment of a document worth keeping that does not warrant its own type       |
| `Reference` | an external work the graph points to but has not ingested                       |
| `Timeline`  | a production-date index of ingested documents                                   |
| `Property`  | a node declaring one predicate — the schema layer's own vocabulary              |
| `Class`     | a node declaring one class — the schema layer's own vocabulary                  |

## `core` is closed, not a place to file shared vocabulary
`core`'s vocabulary is exactly `ARCNET-CORE.md` §10–§11, mirrored here — nothing is added to this
bundle because two domain bundles happen to need the same predicate. A predicate is added to `core`
only by amending `ARCNET-CORE.md` itself; the fact that a second domain wants to reuse something a
first domain coined is not, on its own, grounds to promote that predicate into `core`.

Cross-domain vocabulary that is not part of `ARCNET-CORE.md`'s own set. Filing it under whichever domain coined it first would mean a second
domain only resolves if the first happens to be installed, which is the exact drift this
specification's restructuring exists to prevent (see the root [`README.md`](../../README.md)); `core`
staying closed keeps that fix from quietly reopening `core`'s own boundary as an escape hatch.


## Resource is the catch-all, not a dumping ground

`Resource` exists so that a domain does not have to author a class for every recurring fragment
before it can be recorded — `tags` classifies it informally in the meantime. It is not a place to
put content instead of thinking about whether it recurs; a `Resource` whose tag never accumulates a
second member, forever, is a candidate for prose instead.

## Design notes

- **`Property` and `Class` are classes too.** The schema layer does not step outside the node model
  to describe itself — a predicate's definition and a class's definition are nodes like any other,
  which is what makes the registry self-describing and machine-readable.
- **`Timeline` indexes production dates, not content dates.** A document recounting an occurrence
  from years earlier is still filed under when it was *written* — `CORE.md` §9 is the append-only
  ingestion log; `Timeline` is the read index over it.
- **No `Person`/`Agent` class here.** `Entity` covers an organization or a role; a named individual
  is a domain-specific decision (`incident`'s blameless rule, for one), not a `core` one.
