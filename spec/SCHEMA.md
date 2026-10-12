# SCHEMA — The Vocabulary Layer

**Status:** Proposal · **Layer:** 2 of 3 · **Depends on:** [`CORE.md`](CORE.md)

Every predicate and every class a graph uses is a node under one domain's `Class/` or `Property/`
folder in [`domain/`](domain/). This document specifies the two node formats, the meta-vocabulary
they are written in, and how a graph adopts a subset of them. **It declares no vocabulary itself**
— it describes the form. The vocabulary is each bundle's `Class/` and `Property/` folder, and those
folders are the only normative statement of what any predicate or class means.

## 1. Why the Vocabulary Is Not Prose

A conformance checker (linter) reads only the `Class/` and `Property/` folders of the adopted
domains. It treats each declaration as a specification the graph has to conform to.

So `Class/` and `Property/` form the registry. A rule that exists only in a README is invisible to
the checker and therefore not a rule (CORE §11, invariant 10).


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

The root cause — the underlying condition that made the incident possible; recorded under the incident's own `## Caused By` block.

## Object
- object:: [[Cause]]
```

| Predicate     | Card. | Meaning                                                                                                          |
| ------------- | ----- | ---------------------------------------------------------------------------------------------------------------- |
| `role`        | 1     | serialization position: `meta`, `text`, `edge`, `link` (CORE §4)                                                 |
| `merge`       | 1     | which contributed values are admitted, which is in force, and what happens to losers (CORE §8.4)                 |
| `description` | 1     | the node body — what the predicate means; the `Property` class's default text                                    |
| `bundle`      | 1     | the adoption unit this node belongs to (§6)                                                                      |
| `label`       | 0..1  | heading/bold label for a `link`- or `text`-role predicate; defaults to the name split and capitalized            |
| `aligned`     | 0..1  | the standard-vocabulary term this maps to, or `arc:<name>` when graph-native                                     |
| `datatype`    | 0..1  | literal type, present exactly for `meta`/`text` roles: `string`, `date`, `datetime`, `number`, `boolean`, `enum` |
| `oneOf`       | 0..n  | the legal values, present exactly when `datatype` is `enum`                                                      |
| `object`      | 0..n  | the class(es) this predicate's values are expected to be instances of — the documented, open range               |
| `inverseOf`   | 0..n  | the forward predicate(s) this predicate is a backlink of                                                         |

### 2.1 Range is documented; domain is not

`object` documents what a predicate's values are expected to be. It is guidance for producers and
readers, not a closed set: a consumer MUST NOT reject an edge whose target is of another class, so
a predicate stays usable by classes that later bundles introduce, and a subclass instance (§3.5)
always fits its ancestor's place. There is deliberately no matching `subject`
predicate, because the domain half of an edge signature is **already stated, inverted**, by every
class that requires or permits the predicate — and CORE §3 argues that inverted direction is the one
a document-ingestion graph can actually check. Declaring the domain twice would reintroduce exactly
the drift between two statements of one fact that this layer exists to prevent.

So: to learn which classes may carry `about`, query the classes. To learn what `about` is
expected to point at, read `about`.

### 2.2 Enumerations

Where a predicate's values are drawn from a fixed set, `datatype: enum` and the set as one
front-matter array: `oneOf: [paper, standard, tool, dataset, post]`. `oneOf` is `meta`-role, so
it lives in the front-matter like every other `meta` predicate. The set is closed unless the
description says otherwise.

A value set that varies by class — `class` means one thing on a `Class A` and another on an
`Class B` — cannot be expressed by `oneOf`, which is a property of the predicate. Such a set is
declared by each class, in a `constraint` (§3.3). A predicate needing this frequently is a sign that
two predicates are colliding under one name.

### 2.3 Backlinks

A predicate that is the backlink of another — `narrower` of `broader`, `mentionedIn` of `mentions`
— declares so with `inverseOf`. The relation is one-directional: a backlink MUST NOT exist without
its forward edge, while a forward edge needs its backlink only where a class constraint says so.
Stating the inverse in the predicate's description alone would be prose declaring a rule.

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

A condition that made an incident possible or initiated it, stated as a recurring condition rather than a dated occurrence.

**Identity**
The condition stated as a recurring condition, never a dated occurrence — `Manual Configuration Push`, never `Config Push On 12 April`.

- defaultText:: [[description]]

## Requires One
- requiresOne:: [[description]]
- requiresOne:: [[causeType]]

## Requires Many
- requiresMany:: [[derivedFrom]]

## Permits Many
- permitsMany:: [[contributingFactor]]
- permitsMany:: [[concerns]]

## Constraints
- constraint:: The role a cause played is carried by the edge from the incident, never by a predicate on this node: the same condition is a root cause in one incident and a trigger in the next.
```

### 3.1 Membership and cardinality are one statement

Four predicates fold cardinality into membership. They form a 2 × 2 grid: the verb sets the lower
bound (`requires` = at least one, `permits` = possibly none), the suffix sets the upper bound
(`One` = at most one, `Many` = unbounded):

| Predicate      | Block heading      | Card. | A conforming instance …        |
| -------------- | ------------------ | ----- | ------------------------------ |
| `requiresOne`  | `## Requires One`  | 1     | MUST carry exactly one value   |
| `requiresMany` | `## Requires Many` | 1..n  | MUST carry at least one value  |
| `permitsOne`   | `## Permits One`   | 0..1  | MAY carry at most one value    |
| `permitsMany`  | `## Permits Many`  | 0..n  | MAY carry any number of values |

The lower bound has to live on the class, because **the same predicate is mandatory on one class
and optional on another**. The upper bound is already fixed by the predicate's merge behaviour, and
the suffix MUST agree with it: a single-valued merge (`immutable`, `firstWriteWin`, `fillIfEmpty`,
`lastWriteWin`, `earliest`, `latest`) and `text`-role `append` prose take `…One`; `union` takes `…Many`. The suffix is a
deliberate, checked denormalization so a class reads without opening every predicate. A `union`
predicate listed under `…One` would be corrupted by the first merge that contributed a second
value, which is why the agreement is a constraint on `Class`, not advice.

### 3.2 Identity and default text

`identityRule` states how an instance forms its `@id`. It is a class-level statement because
identity is a property of the class: CORE §5 fixes only the character set and uniqueness.

`defaultText` names the `text`-role predicate an instance renders as its unlabelled body prose,
directly under the H1 (CORE §4). It too is a class-level statement: `description` is the default
text of a `Property` node, `text` that of an `Entity`, `abstract` that of a `Source`, and the same
predicate rendered on another class appears under its label. A class without `defaultText` has no
unlabelled prose. The two prose predicates of `core` differ in meaning and are never substitutes:
`description` is a statement **about** the subject, mostly a definition; `text` is the content the
subject **carries**, built up from sources.

### 3.3 Constraints

`constraint` carries a normative rule no predicate declaration can express — a path restriction, an
ordering, a value dependency, a shape requirement on a linked node. Each is one bullet, phrased as a
checkable MUST.

These are the statements that would otherwise sit in a domain document and be invisible to a
checker: *"a contributing factor is reached only through a cause"*, *"every timestamp present
respects this order"*, *"impact bullets are facet-prefixed"*. They are normative and they live here.

### 3.4 Keywords and universal predicates

`@id` and `@type` are keywords carried by every node (CORE §3); they have no `Property` node and
are never listed on any class. `@conflicts`, carried only by a node with open conflicts (CORE §8.2), and
`@provenance`, recording which documents contributed which values (CORE §8.3), are keywords too:
neither has a `Property` node or is listed on any class.

`created` and `updated` are the only **universal predicates**: they are the creation and
modification times CORE §9.5 requires, and they are mandatory on every graph node, including the
`_schema/` nodes. Because they apply to every class, they are never listed on any class. A bundle's
own source files under `domain/` MAY omit them; installing a bundle into a graph's `_schema/`
stamps them like any other contribution.

Every other predicate a node carries is listed by its class or one of its ancestors.

### 3.5 Subclasses

`subClassOf` declares a class a specialization of exactly one parent class (`rdfs:subClassOf`). An
instance of the subclass is an instance of the parent and of every ancestor above it, so it fits
wherever an ancestor is expected. The subclass inherits everything the ancestors declare —
`identityRule`, `defaultText`, every membership block and every constraint — and inheritance only
adds or tightens:

- it MAY list predicates no ancestor lists, under any of the four blocks;
- it MAY tighten an inherited predicate from `permitsOne` to `requiresOne`, or from `permitsMany`
  to `requiresMany`;
- it MAY add constraints;
- it MUST NOT redeclare `identityRule`, or `defaultText` where an ancestor declares one;
- it MUST NOT relist, remove or loosen any predicate it inherits, other than by the tightening
  above, and every inherited constraint holds on it unchanged.

The chain is single-parent and acyclic; its root, the class without `subClassOf`, carries the
`identityRule`. A node's `@type` names its most specific class, and the node is filed in that
class's folder.

### 3.6 Extending a class from another bundle

A class is **declared** by exactly one bundle: its node carries `bundle`, `description` and its
other class-level statements. Another bundle MAY **extend** it, attaching predicates of its own to
the class without subclassing, with an extension file at `domain/<bundle>/Class/<Name>.md`:

```markdown
---
"@id": Entity
"@type": Class
---
# Entity

## Permits Many
- permitsMany:: [[secures]]
```

An extension file carries only `@id`, `@type: Class` and membership blocks, plus any constraints
that concern only the predicates it attaches. It carries no `bundle`, `description`,
`identityRule`, `defaultText` or `subClassOf` — those belong to the declaring bundle. Installing the
bundle merges the extension into the declared class node in `_schema/` by CORE §8: the membership
predicates merge by `union`, so installation order does not matter. The origin of each attached
predicate stays recorded on the predicate's own `Property` node, by its `bundle`.

An extension MUST NOT:

- list a predicate the class or any of its ancestors already lists — not even to tighten it, which
  only a subclass may do;
- remove any predicate from the class, or change the meaning of any predicate it does not register;
- extend a class whose declaring bundle is not adopted.

An extension SHOULD attach predicates under `permitsOne`/`permitsMany`. Attaching one under
`requiresOne`/`requiresMany` is allowed, but makes every existing instance of the class — and of
its subclasses — non-conforming until the predicate is supplied, so a bundle doing so MUST say
how existing instances are brought into conformance. An attached predicate applies to the class's
subclasses by inheritance.

## 4. Entailment

A consumer reading the adopted bundles' `Class/` and `Property/` folders MUST be able to answer,
for any node, whether it conforms. Every check reads the values in force; the open alternatives
under `@conflicts` (CORE §8.2) and the attributions under `@provenance` (CORE §8.3) are not triples
and are ignored, so a node in conflict conforms exactly when its values in force do.

1. Its `@type` resolves to a `Class` node in an adopted bundle.
2. Every predicate on it resolves to a `Property` node in an adopted bundle.
3. Every predicate on it is listed by its class or an ancestor of it (§3.5), including what
   adopted extensions attach (§3.6), or is universal (§3.4).
4. Every `requiresOne`/`requiresMany` predicate of its class and its ancestors, and every
   universal predicate, is present, at the stated cardinality.
5. No `…One` predicate carries more than one value.
6. Every predicate renders in the position its `role` declares.
7. Every `enum`-typed predicate's values are drawn from its `oneOf` set.
8. Every backlink's forward edge exists (`inverseOf`, §2.3).
9. Every `constraint` on its class and its ancestors holds.

An edge whose target is not an instance of the predicate's documented `object` classes, or their
subclasses, MAY be reported as a warning; it is never a conformance failure (§2.1).

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
- One meaning MUST have one name: a new predicate whose meaning an existing one already covers is a
  duplicate, however different the English.
- A predicate SHOULD be `aligned` to a standard vocabulary term where one exists.

## 6. Bundles

`bundle` names the adoption unit a schema node belongs to, and also names the `domain/<bundle>/`
folder that holds it. A graph adopts `core` plus zero or more domain bundles, and its consumer-side
`_schema/` folder (`CORE.md` §6) holds exactly the union of those bundles' nodes, with every
adopted extension (§3.6) merged into the class it extends.

The `core` bundle defines the vocabulary every document-ingestion graph needs. Its vocabulary is
exactly the contents of [`domain/core/Class/`](domain/core/Class/) and
[`domain/core/Property/`](domain/core/Property/); see [`domain/core/README.md`](domain/core/README.md).


## 7. Conformance

In addition to CORE §12:

- [ ] Every `@type` in use resolves to a `Class` node in an adopted bundle.
- [ ] Every predicate in use resolves to a `Property` node in an adopted bundle.
- [ ] Every `Property` node carries `role`, `merge`, `bundle`, and a `description`.
- [ ] Every `Class` node carries `bundle` and a `description`; every root class (no `subClassOf`)
      carries an `identityRule`, and no subclass does.
- [ ] Every `subClassOf` chain is acyclic and every parent resolves to a registered `Class`.
- [ ] No subclass relists, removes or loosens an inherited predicate except by tightening (§3.5).
- [ ] Every extension file names a class declared by an adopted bundle, carries only membership
      blocks and constraints, and lists no predicate the class or its ancestors already list (§3.6).
- [ ] Every predicate name is registered exactly once across all adopted bundles.
- [ ] Every predicate a class lists resolves to a registered `Property`.
- [ ] Every class an `object` bullet names resolves to a registered `Class`.
- [ ] Every predicate an `inverseOf` bullet names resolves to a registered `Property`.
- [ ] Every predicate registered in an adopted bundle is listed by at least one class or extension
      in that bundle, or is universal (§3.4).
- [ ] Every class's `…One`/`…Many` listing agrees with each listed predicate's `merge` (§3.1).
- [ ] Every `enum`-typed predicate declares at least one `oneOf` value.
- [ ] No `[[link]]` inside a schema node's description creates an edge — illustrative names are
      written in backticks, never as wiki-links.
