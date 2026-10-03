# SCHEMA — The Vocabulary Layer

**Status:** Draft · **Layer:** 2 of 3 · **Depends on:** [`CORE.md`](CORE.md)

Every predicate and every class a graph uses is a node under one domain's `Class/` or `Property/`
folder in [`domain/`](domain/). This document specifies the two node formats, the meta-vocabulary
they are written in, and how a graph adopts a subset of them. **It declares no vocabulary itself**
— it describes the form. The vocabulary is each bundle's `Class/` and `Property/` folder, and those
folders are the only normative statement of what any predicate or class means.

## 1. Why the Vocabulary Is Not Prose

A conformance checker (linter) reads only the `Class/` and `Property/` folders of the adopted domains. It treats each declaration as a specification document the graph has to be conformant.

So `Class/` and `Property/` formt the the registry. 


## 2. `Property` Nodes

One node per predicate, at `domain/<bundle>/Property/<name>.md`. The name is camelCase and equal to
the basename.

```markdown
---
"@id": causedBy
"@type": Property
role: link
merge: union
label: "Caused By"
aligned: "arc:causedBy"
bundle: incident
---
# causedBy

The root cause — the underlying condition that made the incident possible; recorded under the
incident's own `## Caused By` block.

## Object
- object:: [[Cause]]
```

| Predicate     | Card. | Meaning                                                                              |
| ------------- | ----- | ------------------------------------------------------------------------------------ |
| `role`        | 1     | serialization position: `meta`, `text`, `href`, `edge`, `link` (CORE §4)             |
| `merge`       | 1     | how two contributions combine (CORE §8)                                              |
| `description` | 1     | the node body — what the predicate means                                             |
| `label`       | 0..1  | heading shown for a `link`-role predicate's block; defaults to the name, capitalized |
| `aligned`     | 0..1  | the standard-vocabulary term this maps to, or `arc:<name>` when graph-native         |
| `datatype`    | 0..1  | literal type for `meta`/`text` roles: `string`, `date`, `number`, `boolean`, `enum`  |
| `permits`     | 0..n  | a legal value, when `datatype` is `enum`                                             |
| `object`      | 0..n  | the class this predicate's values are instances of — the range                       |
| `bundle`      | 0..1  | the adoption unit this node belongs to (§6)                                          |

### 2.1 Range is declared; domain is not

`object` states what a predicate's values may be. There is deliberately no matching `subject`
predicate, because the domain half of an edge signature is **already stated, inverted**, by every
class that requires or permits the predicate — and CORE §3 argues that inverted direction is the one
a document-ingestion graph can actually check. Declaring the domain twice would reintroduce the drift to specification, it is exactly the issue to prevent (caused by LLM behaviour).

So: to learn which classes may carry `concerns`, query the classes. To learn what `concerns` may
point at, read `concerns`.

### 2.2 Enumerations

Where a predicate's values are drawn from a fixed set, `datatype: enum` and one `permits` bullet per
value. The set is closed unless the description says otherwise.

A value set that varies by class — `class` means one thing on a `Hypothesis` and another on an
`Aporia` — cannot be expressed by `permits`, which is a property of the predicate. Such a set is
declared by each class, in a `constraint` (§3.3). A predicate needing this frequently is a sign that
two predicates are colliding under one name.

## 3. `Class` Nodes

One node per class, at `domain/<bundle>/Class/<Name>.md`. The name is UpperCamelCase and equal to
both the basename and the class's own folder name under `CORE.md` §6 (a graph's `<Class>/`, not
this bundle's `Class/`).

```markdown
---
"@id": Cause
"@type": Class
bundle: incident
---
# Cause

A condition that made an incident possible or initiated it, stated as a recurring condition rather
than a dated occurrence.

**Identity**
The condition stated as a recurring condition, never a dated occurrence — `Manual Configuration
Push`, never `Config Push On 12 April`.

## Requires One
- requiresOne:: [[text]]
- requiresOne:: [[causeType]]

## Requires Many
- requiresMany:: [[derivedFrom]]

## Permits Any
- permitsMany:: [[contributingFactor]]
- permitsMany:: [[concerns]]

## Constraints
- constraint:: The role a cause played is carried by the edge from the incident, never by a
  predicate on this node: the same condition is a root cause in one incident and a trigger in
  the next.
```

### 3.1 Membership and cardinality are one statement

Four predicates folding cardinality into membership:

| Predicate      | Block heading      | A conforming instance …        |
| -------------- | ------------------ | ------------------------------ |
| `requiresOne`  | `## Requires One`  | MUST carry exactly one value   |
| `requiresMany` | `## Requires Many` | MUST carry at least one value  |
| `permitsOne`   | `## Permits One`   | MAY carry at most one value    |
| `permitsMany`  | `## Permits Any`   | MAY carry any number of values |


Cardinality has to live on class rather than on the predicate, because **the same predicate carries different cardinality on different classes**.

The merge behaviour MAY influence the cardinality: single-valued merges (`immutable`, `firstWriteWin`, `fillIfEmpty`, `lastWriteWin`) give `…One`, accumulating merges (`union`, `append`) give `…Many`.

### 3.2 Identity

`identityRule` states how an instance forms its `@id`. It is a class-level statement because
identity is a property of the class: CORE §5 fixes only the character set and uniqueness.

### 3.3 Constraints

`constraint` carries a normative rule no predicate declaration can express — a path restriction, an
ordering, a value dependency, a shape requirement on a linked node. Each is one bullet, phrased as a
checkable MUST.

These are the statements that would otherwise sit in a domain document and be invisible to a
checker: *"a contributing factor is reached only through a cause"*, *"every timestamp present
respects this order"*, *"impact bullets are facet-prefixed"*. They are normative and they live here.

### 3.4 Universal predicates

`@id` and `@type` are carried by every node and are never listed on any class. `created` and
`updated` are optional node bookkeeping, permitted on every class and likewise never listed. No
other predicate is universal.

## 4. Entailment

A consumer reading the adopted bundles' `Class/` and `Property/` folders MUST be able to answer,
for any node, whether it conforms:

1. Its `@type` resolves to a `Class` node in an adopted bundle.
2. Every predicate on it resolves to a `Property` node in an adopted bundle.
3. Every predicate on it is listed by its class, or is universal (§3.4).
4. Every `requiresOne`/`requiresMany` predicate is present, at the stated cardinality.
5. No `permitsOne` predicate carries more than one value.
6. Every predicate renders in the position its `role` declares.
7. Every `enum`-typed predicate's values are drawn from its `permits` set.
8. Every edge's target is an instance of a class in the predicate's `object` set, where one is
   declared.
9. Every `constraint` on its class holds.

Checks 1–8 are mechanical. Check 9 is mechanical per constraint but its expression is prose; a graph
wanting machine-checked constraints refines `constraint` into a shape language of its own, which is
an extension of this layer, not a change to it.

## 5. Adding to the Vocabulary

- A predicate MUST be registered before use, and reused before a new one is coined.
- A predicate name is registered **exactly once across every bundle in use**. Two bundles
  introducing the same name is a collision, not a merge — and is the failure this layout makes
  visible, because both would claim the same file.
- Two predicates that differ in `role`, `merge`, or meaning are two predicates and MUST have two
  names, however similar the English.
- A predicate SHOULD be `aligned` to a standard vocabulary term where one exists.

## 6. Bundles

`bundle` names the adoption unit a schema node belongs to, and also names the `domain/<bundle>/` folder that holds it. A graph adopts `core` plus zero or more domain bundles, and its consumer-side `_schema/` folder (`CORE.md` §6) holds exactly the union of those bundles' nodes.

The `core` bundle defines the vocabulary every document-ingestion graph needs, mirrored verbatim from `CORE.md` §10–§11, see [`domain/core/README.md`](domain/core/README.md).


## 7. Conformance

In addition to CORE §12:

- [ ] Every `@type` in use resolves to a `Class` node in an adopted bundle.
- [ ] Every predicate in use resolves to a `Property` node in an adopted bundle.
- [ ] Every `Property` node carries `role`, `merge`, and a description.
- [ ] Every `Class` node carries a description and an `identityRule`.
- [ ] Every predicate name is registered exactly once across all adopted bundles.
- [ ] Every predicate a class lists resolves to a registered `Property`.
- [ ] Every class an `object` bullet names resolves to a registered `Class`.
- [ ] Every predicate registered in an adopted bundle is listed by at least one class in that
      bundle, or is universal (§3.4).
- [ ] Every `enum`-typed predicate declares at least one `permits` value.
- [ ] No `[[link]]` inside a schema node's description creates an edge — illustrative names are
      written in backticks, never as wiki-links.
