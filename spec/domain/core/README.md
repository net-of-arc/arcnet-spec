# CORE — The Vocabulary Every Graph Needs

**Status:** Proposal · **Layer:** 3 of 3 (narrative for a layer-2 bundle) · **Bundle:** `core`

Narrative for the `core` bundle. **Nothing here is normative.** The vocabulary is
[`Class/`](Class/) and [`Property/`](Property/); `CORE.md` (mechanism) and `SCHEMA.md` (node
formats) at the spec root cover the rest of what this bundle needs explained.

**Classes:** `Source`, `Entity`, `Resource`, `Reference`, `Timeline`, `Property`, `Class`.
**Reused from:** nothing — this is the bundle every other bundle reuses.

## What the bundle is for

Every arcnet domain bundle adopts `core` and nothing composes without it. It supplies the five classes a knowledge graph needs regardless of subject matter, and the two classes the schema layer itself is written in:

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

`core`'s vocabulary is exactly the contents of this bundle's `Class/` and `Property/` folders.
Nothing is added to it because two domain bundles happen to need the same predicate; a predicate
enters `core` only when every document-ingestion graph needs it.

Cross-domain vocabulary that is not part of `core` belongs in a shared domain bundle (such as
`universal`), never under whichever domain coined it first. Filing it under the first domain would
mean a second domain only resolves if the first happens to be installed, which is the exact drift
this specification's bundle layout exists to prevent; `core` staying closed keeps that fix from
quietly reopening `core`'s own boundary as an escape hatch.


## Resource is the catch-all, not a dumping ground

`Resource` exists so that a domain does not have to author a class for every recurring fragment
before it can be recorded — `tags` classifies it informally in the meantime. It is not a place to
put content instead of thinking about whether it recurs; a `Resource` whose tag never accumulates a
second member, forever, is a candidate for prose instead.

## Three classifiers, three questions

`core` classifies along three axes that never apply to the same thing:

| Predicate  | Carried by                        | Answers                             | Values                         |
| ---------- | --------------------------------- | ----------------------------------- | ------------------------------ |
| `category` | `Entity`                          | what kind of thing is this subject? | one of Sowa's twelve leaves    |
| `genre`    | `Source`, `Reference`             | what form does this work take?      | `paper`, `standard`, `tool`, … |
| `about`    | `Source`, `Reference`, `Resource` | which subject is this work about?   | an edge to an `Entity`         |

A work is never asked what Sowa category it is, and a subject is never asked its genre. "This paper
is about a framework" is written as `about:: [[Some Framework]]`, with the framework's own node
saying what it is.

## Design notes

- **`Property` and `Class` are classes too.** The schema layer does not step outside the node model
  to describe itself — a predicate's definition and a class's definition are nodes like any other,
  which is what makes the registry self-describing and machine-readable.
- **A cited work and an ingested work are two records, linked.** A `Reference` is named as citing
  documents name the work (`RFC 8446`), taken from their links and reference lists; a `Source` is
  named by its citekey (`rescorla-2018-tls13`). When a work exists as both, the two nodes are
  linked by `sameAs` in both directions rather than merged, so every `cites` edge keeps resolving
  and each node keeps its own `title` — as cited and as published.
- **Citation stance is a polarity, not a taxonomy.** `core` keeps `cites` (no stance), `supports`
  and `disputes`. The finer CITO intents (`citesAsEvidence`, `confirms`, `refutes`, …) are not
  distinctions every graph needs, and producers cannot draw them reliably; a domain that needs them
  registers them and attaches them to `Source` by extension (SCHEMA §3.6).
- **`description` is about the subject; `text` is what the subject carries.** Schema nodes are
  defined by a single `description`; an `Entity` accumulates `text` from every source that mentions
  it. Each class names its own unlabelled prose with `defaultText`.
- **`core` is closed, its classes are not.** A domain bundle adds predicates to a `core` class by
  extension (SCHEMA §3.6), or specializes it with `subClassOf` (SCHEMA §3.5); it never edits a
  `core` file.
- **`Timeline` indexes production dates, not content dates.** A document recounting an occurrence
  from years earlier is still filed under when it was *written*. Git (`CORE.md` §9) is the
  append-only ingestion log; `Timeline` is a derived read index over the `Source` nodes.
- **Backlinks are optional, forward links are not.** `narrower`, `hasPart`, `isReplacedBy` and
  `isCitedBy` may be written for readability, but each declares `inverseOf` and is valid only while
  its forward edge exists. `mentions`/`mentionedIn` are the exception: provenance is required in
  both directions.
- **Time is first class.** Every node carries `created` and `updated` (CORE §9.5), so the graph
  can be sliced by time — what it knew as of a date, what changed in a window — without replaying
  git. They merge as `earliest`/`latest`, which is order-independent, so concurrent branches never
  conflict on them. They are graph times; a document's own date is `published`.
- **No `Person`/`Agent` class here.** `Entity` covers an organization or a role; a named individual
  is a domain-specific decision (`incident`'s blameless rule, for one), not a `core` one.
