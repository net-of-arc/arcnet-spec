# CORE — Mechanism

**Status:** Draft · **Layer:** 1 of 3

This document specifies the **mechanism** of a knowledge graph stored as plain Markdown: the
RDF-aligned data model, the node model, identity, folder layout, edges, the merge protocol, version
control, and the patch exchange format. It depends on no program, library, or
language.

CORE defines **no predicates and no classes.** The vocabulary lives entirely in the schema layer
([`SCHEMA.md`](SCHEMA.md) and each bundle's `Class/` / `Property/` folder under [`domain/`](domain/)),
where it is machine-readable. This document never names a predicate except to illustrate a
mechanism, and a graph may replace the entire vocabulary without changing one word here.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in
RFC 2119.

## 2. Purpose and Layers

A knowledge graph is a set of linked Markdown files. Each file is a node that depicts a subject; `[[wiki-links]]` between files are edges (predicates). Underneath the Markdown, every node is a **bag of predicates about one subject** —
the same statement an RDF triple makes — and Markdown is a lossless, serialization of that bag. The format combines Markdown with a domain-specific language that gives the best of both worlds: human-friendly prose and structured, machine-readable data in a single, version-controllable text format that both readers and AI agents can seamlessly parse.

The specification is three layers, each depending only downward:

| Layer          | Artifact                                | Holds                                                                   | Normative for                             |
| -------------- | --------------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------- |
| 1 · Mechanism  | `CORE.md`                               | how a node is written, identified, linked, merged, versioned, exchanged | every graph                               |
| 2 · Vocabulary | `SCHEMA.md` + `domain/*/Class,Property` | every predicate and every class, as nodes                               | every graph, per adopted domain           |
| 3 · Narrative  | `domain/*/README.md`                    | why a vocabulary is shaped as it is, how to map a document onto it      | nobody — it explains, it does not declare |


## 3. RDF Model

A node names one **subject**. Its `@id` is the subject's identifier; its `@type` names the class the
subject is an instance of. Every other predicate on the node is a statement **about** that subject —
one RDF triple `(@id, predicate, value)` per value; a multi-valued predicate decomposes into one
triple per element.

`@id` and `@type` are **keywords**, not predicates (as in JSON-LD): they are carried by every node,
have no `Property` node, and are never listed by any class. Two further keywords record merge
state beside the triples, never as triples: `@conflicts`, present only on a node with open
conflicts (§8.2), and `@provenance`, recording which documents contributed which values (§8.3).

```markdown
---
"@id": Transport Layer Security
"@type": Entity
category: script
tags: [cryptography]
---
# Transport Layer Security

A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted
network.

- replaces:: [[SSL Protocol]]
- conformsTo:: [[RFC 8446]]

## Mentioned In
- mentionedIn:: [[rescorla-2026-tls13]]
```

The same node as triples (illustrative notation):

```text
"Transport Layer Security"
  a "Entity"
  category "script"
  tags "cryptography"
  text "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network."
  replaces "SSL Protocol"
  conformsTo "RFC 8446"
  mentionedIn "rescorla-2026-tls13"
  .
```

The conversion is lossless and reversible in both directions **up to canonical form** — Markdown →
triples → Markdown reproduces the original file once it is in canonical form, and triples →
Markdown → triples reproduces the original bag — because every predicate's rendering position is
fixed once, on the predicate's own schema node, not decided ad hoc per document. §4 gives the
positions a predicate can render to. Canonical form is:

- the values of a multi-valued predicate are a **set**; their written order carries no meaning. A
  producer SHOULD write them in a stable order (the `oneOf` order for enumerations, otherwise
  lexicographic) so that independent writers produce identical files;
- typed edges are written in the list form (§7);
- untyped links stay inside the prose that carries them and produce no triple of their own.

The specifcation inverts the classical RDFS reading for a practical reason. `rdfs:domain` states which classes a
property's subjects belong to; for a document-ingestion graph the checkable direction runs the other
way — a **class declares which predicates its instances require**. The schema layer's `Class` and
`Property` nodes are exactly `rdfs:Class` and `rdf:Property`, authored for that direction. Only the
range half of an edge signature is written down, because only the range half is not already
implied by the classes that use the predicate — and, as with `rdfs:range`, it documents the expected
target rather than restricting it.

## 4. Node Model

Every node is a YAML front-matter block (delimited by `---`) followed by a Markdown body. Both are
constructed from the same bag of predicates about the node's subject; the split between front-matter
and body, and the exact placement within the body, is fixed by each predicate's declared **role** —
an attribute of the predicate's own schema node, not a convention applied per document.  Allowed roles are:

- **`meta`** — a front-matter scalar or array, in the YAML block.
- **`text`** — prose in the body.
- **`edge`** — a typed edge, a bullet in a heterogeneous block: `predicate:: [[Target]]`.
- **`link`** — a typed, homogeneous edge list, its own `## Label` block, one bullet per target.

An **untyped link** — a bare `[[Target]]` inside prose — is not a role of any predicate. It is part
of the `text` value that contains it, annotates that prose, and never replaces it.

Because every predicate's role is declared exactly once, reconstructing a node's Markdown from its
triples — or recovering its triples by reading the Markdown — needs no per-class special-casing: a
consumer resolves each predicate's role and the rendering follows mechanically. Consumers MUST
preserve unknown predicates, so the format tolerates a vocabulary that grows over time.

Concretely:

- **Front-matter** carries every predicate with role `meta`, together with the mandatory `@id` and
  `@type` keywords, the `@provenance` keyword (§8.3) and, while the node has open conflicts, the
  `@conflicts` keyword (§8.2). A key beginning with `@` MUST be written as a quoted YAML string
  key — `"@id"`, `"@type"`, `"@conflicts"`, `"@provenance"` — never bare. YAML reserves the unquoted `@` character as an indicator for
  future use, so a bare `@id:` key is invalid or inconsistently accepted across parsers.
- **Body** carries, in this order:
  1. the H1 title, equal to the `@id`;
  2. prose from the `text`-role predicate the class declares as its **default text** — the node's
     unlabelled paragraph(s). Which predicate that is, if any, is a statement of the class, not of
     the predicate, because one predicate can be the default on one class and labelled on another;
  3. each further `text`-role predicate in use whose merge is not `union`, as its bold `**Label**`
     on its own line followed by its paragraph;
  4. one heterogeneous block of `predicate:: [[Target]]` bullets, holding every `edge`-role
     predicate in use, optionally grouped under bold labels where that aids reading;
  5. one `## Label` block per `link`-role predicate in use, its bullets written exactly as
     `edge`-role bullets are, and one `## Label` block per `text`-role predicate whose merge is
     `union` (a list of statements), one `predicate:: statement` bullet per value. The heading
     groups the bullets for display; it does not change the bullet syntax and carries no data.
     The heading is mandatory, even when it is the node's only block: it is what separates a
     `link` block from the heterogeneous `edge` block, so a parser can tell the two apart without
     consulting the schema.

  `Label` is the predicate's declared `label`, or by default its name split at camelCase boundaries
  and capitalized (`mentionedIn` → `Mentioned In`).
- Prose MUST NOT contain Markdown ATX headings (`#` … `######`) or lines that begin with a
  `predicate::` bullet: both are structure, and prose containing them would be re-read as structure.

## 5. Identity

A node’s identity (`@id`) is defined by its Markdown file basename (excluding .md), which must match its H1 title. The `@id` MUST be unique across the entire graph, including `_schema/` nodes. Uniqueness is evaluated **case-insensitively using Unicode NFC normalization**. Links resolve by file basename, and common file systems (macOS, Windows) along with wiki-link resolvers fold case. Example Collision: `Object` (an entity) and `object` (a predicate) conflict because they normalize to the same case-insensitive key.

The identity MUST NOT contain any of `/ \ : * ? " < > | . # ^ [ ]` — characters that make an
identity unusable as a filename or that break `[[wiki-link]]`, heading-anchor (`#`) or block-ref
(`^`) syntax — and MUST NOT begin or end with a space or contain two consecutive spaces. A producer
SHOULD restrict itself to the safe subset: **letters, digits, spaces, hyphens, and parentheses.**
A colliding identity MUST be disambiguated with a parenthetical qualifier, never with a slash or a
colon.

**How each class forms its `@id` is declared by that class**, in its `identityRule` predicate. CORE
fixes only the character set and the uniqueness requirement. An identity rule MUST be a function of
the subject alone, never of ingestion order: "whichever arrives second gets a suffix" makes the
graph depend on the order documents were ingested in.

**Aliases.** A subject named differently across documents has exactly one **canonical** node. Every
other name for it , including an original name that contains characters forbidden above , is an
**alias**. The alias is a `meta`-role value on the canonical node itself, declared by the schema layer. An alias MUST resolve to exactly one node: it MUST NOT equal, under the same comparison as `@id`, any other node's `@id` or alias. Like `@id` uniqueness, this is checked against the nodes directly; the graph keeps no separate alias table. Producers MUST resolve a name against existing `@id`s and aliases before creating a node, so synonyms collapse onto the canonical node, and MUST write edges to the canonical `@id` — an alias is never a link target (§7).

Predicate names MUST be **camelCase**; class names MUST be **UpperCamelCase**. The two vocabularies
are therefore distinguishable wherever they appear, including bare in prose.


## 6. Folder Structure

Each class has exactly one folder and each folder holds exactly one class. A class's folder name
SHOULD be **character-for-character identical to the class name** — same case, no pluralization, no
abbreviation, no synonym. Folder ↔ class is string equality, so no consumer carries a mapping table.

```
graph/
├── <Class>/              # one folder per class in the adopted bundles
├── timeline/             # bucketed index — exempt, see below
│   ├── yearly/
│   └── monthly/
└── _schema/              # namespace prefix — exempt
    ├── Property/         #   one node per predicate
    └── Class/            #   one node per class
```

Two folders are exempt because neither is a class folder: `_schema/` is a namespace prefix whose
children are class folders and do follow the rule; `timeline/` is an index whose nodes are bucketed
by granularity rather than filed flat. Every file in the graph is a node.

Folders exist purely for filing convenience. Because links resolve strictly by file basename, moving a node to another folder will never break an edge. A consumer MUST determine a node's class using its internal `@type` property. Consumers MUST NOT infer classification from folder location. The folder structure mirrors the class for convenience, but never replaces it.

Because `@id` uniqueness is graph-wide, two files sharing the same basename—even in different folders—is a conformance error, not two distinct nodes.


## 7. Edges

An edge is `[[Target]]`, a basename reference to the target's canonical `@id`; it MAY carry display
text `[[Target|text]]`. Display text is presentation only: two edges are equal when their targets are the same `@id`, whatever their display text. A bare `[[Target]]` with no predicate is an untyped link (§4).

A predicate types an edge. Three forms are recognized; consumers MUST read all three:

- **List form:** `- replaces:: [[SSL Protocol]]` — the canonical form.
- **Body form:** `conformsTo:: [[RFC 8446]]`
- **Inline form:** `... was [supports:: [[RFC 8446]]] standardized.`

Producers SHOULD use the list form for `edge`-role predicates, and MUST use the list form inside a
`## Label` block for `link`-role predicates — the heading is never omitted (§4). A round
trip through triples MAY normalize body and inline forms to the list form.

The `::` token separates predicate from target.


## 8. Merge

Merging is defined on **predicate values, never on lines of text**. Every operation that brings two states of a node together — applying a patch (§10.2), combining two branches (§9.4), resolving a conflict, retracting a document, reverting a commit — parses both into predicate bags, merges them under this section, and writes the result in canonical form (§3). A file in the graph therefore never contains merge markers: it is always valid YAML front-matter over a valid body.

### 8.1 Candidate sets

For each predicate on a node, merging operates on that predicate's **candidate set** `S`:

- for a **multi-valued** predicate (`union`), `S` is its set of values, all of them in force;
- for a **single-valued** predicate, `S = {w} ∪ C`, where `w` is the **value in force** — rendered
  in the position the predicate's role declares — and `C` is the set of **open alternatives**:
  values that were contributed but did not win, kept for review;
- for **`append`** prose, `S` is an ordered list of paragraphs, each present at most once.

A single-valued predicate whose `C` is empty is **settled**; otherwise it is **in conflict**. Its
value in force is always a legal value at the declared cardinality, so a node in conflict still
conforms: disagreement sits beside the value in force, never in place of it.

### 8.2 The `@conflicts` keyword

A node's open alternatives are recorded in its front-matter under the keyword `"@conflicts"`: a map
from predicate name to the array of that predicate's open alternatives.

```yaml
---
"@id": Transport Layer Security
"@type": Entity
category: script
"@conflicts":
  category: [object, process]
  description:
    - |
      A protocol suite for securing transport connections, standardized by the IETF.
  replaces: ["[[SSL 3.0]]"]
---
```

- **Encoding.** A value is written as the predicate would write it in front-matter: a `meta` value
  as is, a `text` value as a YAML block scalar, an edge as a quoted `"[[Target]]"` string. Every
  role therefore has a valid representation, including the front-matter itself.
- **Fixed depth.** The structure is predicate → array of atomic values, and nothing deeper. Merging
  two conflicts on one predicate yields one array, their union; a conflict cannot contain a
  conflict.
- **Not part of the graph.** Values under `@conflicts` produce no triples and are ignored by the
  cardinality, role and conformance checks (SCHEMA §4). A consumer not concerned with review MAY
  ignore the key entirely.
- **Present only while needed.** An entry exists exactly while its predicate is in conflict; the
  key exists exactly while some entry does. Each array is written in canonical order (§8.5) and
  never contains the value in force.
- **Graph only.** A patch MUST NOT carry `@conflicts` (§10): producers contribute values, not
  disputes.

### 8.3 The `@provenance` keyword

Which documents contributed which values is recorded in the node's front-matter under the keyword
`"@provenance"`: a map from a contributing document's `@id` to the values that document contributed
to this node, grouped by predicate.

```yaml
---
"@id": Transport Layer Security
"@type": Entity
category: script
"@provenance":
  dierks-2008-tls12:
    category: [script]
    text: ["sha256:91ab03ce55f2a7d0"]
  rescorla-2018-tls13:
    category: [script]
    replaces: ["[[SSL Protocol]]"]
    text: ["sha256:4f1c2a9b0e7d3c18"]
---
```

- **Attribution.** Each (document, predicate, value) entry is one **attribution**: that document
  contributed that value. A value may be attributed to several documents. A value attributed to
  none — written by hand, or present before the graph kept provenance — is **unattributed**.
- **Encoding.** Keys are document `@id`s written as plain strings. Values are encoded as under
  `@conflicts` (§8.2), except a `text`-role value, which is written as its **digest**:
  `sha256:` followed by the first 16 lowercase hexadecimal digits of the SHA-256 of the paragraph's
  UTF-8 bytes, after the normalization paragraph equality uses (§8.5). A digest keeps the node from
  carrying every paragraph twice.
- **Fixed depth.** Document → predicate → array of atomic values, and nothing deeper. Arrays and
  keys are written in canonical order (§8.5).
- **Consistent with the candidate sets.** Only values in a candidate set — in force or open
  alternatives — are attributed. When a value leaves its candidate set, its attributions go with it;
  a document left with no attributed value loses its key, and the keyword is absent when no
  document has one.
- **Not part of the graph.** Attributions produce no triples and are ignored by the cardinality,
  role and conformance checks (SCHEMA §4). The universal timestamps (§9.5) are never attributed:
  they are stamped by the apply, not contributed.
- **Written by apply only.** Every apply (§8.6) MUST record the attributions of the contribution it
  merges. A patch MUST NOT carry `@provenance` (§10): the attributing document is the patch's own
  `@id`.

Provenance is what makes retracting any document, not only the most recent one, well defined
(§8.9).

### 8.4 Merge values

Each predicate declares one `merge` value. It decides which contributed values are admitted into `S`, which one is in force, and what happens to the values that lose. The **previous value in force** is the one the node held before the operation — before the patch when applying (§8.6), in the merge base when combining (§8.7).

| `merge`         | Admitted into `S`               | Value in force                                                                                                                    | Losing values                                                                                                   |
| --------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `immutable`     | the value in force only         | the first value; it never changes                                                                                                 | a different value is an identity collision (§8.10)                                                              |
| `union`         | every value                     | all of `S`                                                                                                                        | none                                                                                                            |
| `firstWriteWin` | every value                     | the previous value in force while it is in `S`; otherwise the least of `S` in canonical order                                     | open alternatives, reported                                                                                     |
| `fillIfEmpty`   | the value in force only         | the previous value in force if any; otherwise the least contributed value in canonical order                                      | discarded, not reported                                                                                         |
| `lastWriteWin`  | every value                     | on apply, the incoming value; on combine, the value of the side that changed it, or the greatest in canonical order when both did | discarded, except that when both sides of a combine changed it the other value is an open alternative, reported |
| `earliest`      | the value in force only         | the least value (`date`/`datetime` order)                                                                                         | discarded — not a disagreement                                                                                  |
| `latest`        | the value in force only         | the greatest value                                                                                                                | discarded — not a disagreement                                                                                  |
| `append`        | every paragraph not yet present | the whole ordered list                                                                                                            | none                                                                                                            |

**Unknown predicates** — present on a node but registered in no adopted bundle — are preserved in
the position they were found and merged as `firstWriteWin`, so no contributed value is lost.

A class-level "this node never changes after creation" behaviour is not a separate merge value — it
emerges from every one of that class's predicates independently being `immutable`, reinforced by the
ingestion idempotency check (§9.2).

### 8.5 Equality and canonical order

**Value equality.** Literals compare by exact string equality after trimming surrounding whitespace
and NFC normalization; `date` and `datetime` literals compare by value; edges compare by target
`@id`, ignoring display text (§7); paragraphs compare after whitespace normalization.

**Canonical order** is a total order over the values of one predicate: `date` and `datetime` values
by value, every other value by the Unicode code points of its NFC-normalized canonical rendering.
It is used only where no previous value in force exists to prefer, and to order concurrent
insertions. It is arbitrary but deterministic and symmetric, and every choice it makes between
disagreeing values leaves the losing value visible under `@conflicts` (§8.4), so it never decides
silently.

### 8.6 Apply — a contribution onto a node

A contribution, one node of a patch, only **adds**: it never removes a value or an open
alternative. For each predicate it carries:

1. admit its values into `S` as the predicate's merge value allows (§8.4);
2. recompute the value in force, preferring the previous value in force where §8.4 says so;
3. record every new loser the merge value keeps under `@conflicts`, and report it with a
   `Needs-Review:` trailer (§9.3);
4. attribute to the contributing document every value it carries that is now in `S`, whether in
   force or an open alternative (§8.3); a value the merge value discarded is not attributed;
5. a diverging `immutable` value or `@type` fails the whole apply (§8.10).

A value already in `S` — in force or as an open alternative — changes nothing, so applying a
contribution twice is a no-op — its attributions are already recorded too. A value that was dismissed earlier (§8.8) re-enters as an open
alternative: a new document repeating it is new evidence.

### 8.7 Combine — a three-way merge

Two states `A` and `B` of one node are merged against their common ancestor `base` — the git merge
base, or an empty node when both sides created the node independently:

1. **Identity.** A different `@type` on the two sides, or an `immutable` predicate holding different
   values, is an identity collision (§8.10).
2. **Candidate set.** For every predicate,
   `S = (S_base − removed_A − removed_B) ∪ added_A ∪ added_B`, where each side's additions and
   removals are taken relative to `base`. A value removed on one side and untouched on the other is
   removed: this is how a deleted edge or a dismissed alternative propagates. No value can be added
   on one side and removed on the other, since a removal needs the value in `base`.
3. **Value in force.** A single-valued predicate's value in force merges as a register: unchanged
   on both sides, it keeps the `base` value; changed on one side, it takes that side's value;
   changed on both to the same value, it takes that value; changed on both to different values, the
   merge value chooses (§8.4) and the other value becomes an open alternative unless the merge value
   discards losers.
4. **Choosing beats dismissing.** If the merged value in force is not in the merged `S` — one side
   dismissed what the other chose — it is added back. The value in force is never lost.
5. **Never empty by contradiction.** If a single-valued predicate's merged `S` is empty while both
   `A` and `B` hold a value, then `S = S_A ∪ S_B`. This happens only when two resolutions
   contradict each other (§8.8), and the result is a visible conflict.
6. **Prose.** An `append` list merges three-way as a list: a paragraph removed on either side is
   removed; a paragraph added on either side is inserted after the paragraph it follows on that
   side; concurrent insertions at the same point are ordered canonically (§8.5). A `base` paragraph
   replaced on both sides by different text keeps both replacements and is reported with
   `Needs-Review:` — prose is never dropped silently.
7. **Provenance.** The attributions merge as a set of (document, predicate, value) entries by the
   same three-way formula as step 2; then every attribution whose value is not in the merged `S` is
   dropped (§8.3).
8. **Result.** Write `C = S − {w}` under `@conflicts`, the attributions under `@provenance`, and the
   node in canonical form.

### 8.8 Resolve — settling a conflict

Settling a conflict is an ordinary edit, committed on its own (§9.3), built from two moves on one
single-valued predicate:

- **choose `v`** (`v` an open alternative) — `v` becomes the value in force and the previous value
  in force becomes an open alternative;
- **dismiss `v`** — remove `v` from the open alternatives, together with its attributions (§8.3).

The usual resolution chooses one value and dismisses the rest, which settles the predicate and
removes its `@conflicts` entry. Resolutions are commits, so they combine under §8.7: two branches
resolving a conflict the same way merge cleanly; two resolving it differently merge back into one
flat conflict holding both choices (§8.7, steps 3–5). A multi-valued or prose predicate has no
open alternatives; a wrong value or paragraph in one is simply edited out, and §8.7 propagates the
removal.

### 8.9 Retract — withdrawing a document

Retracting a document `D` removes exactly what `D` alone contributed, wherever it sits in history.
It is a defined operation, committed on its own (§9.3), not a `git revert`:

1. **Withdraw attributions.** On every node whose `@provenance` has the key `D`, remove that key.
2. **Remove orphaned values.** Every value that was attributed to `D` and is now attributed to no
   document is removed from its candidate set — a set element, a paragraph, an open alternative, or
   the value in force. A value attributed to another document stays, and an unattributed value is
   never removed: retraction removes only what provenance proves `D` alone supplied.
3. **Re-elect the value in force.** Where a single-valued predicate lost its value in force, the
   merge value chooses a new one from the remaining open alternatives as if there were no previous
   value in force (§8.4), and the choice is reported with `Needs-Review:`. A predicate left with no
   value is absent; if its class requires it, that is reported too.
4. **Remove orphaned nodes.** A node left with no value of any predicate other than the universal
   timestamps is deleted — this includes `D`'s own document node — unless an edge that remains in
   the graph still targets it, in which case it is kept and reported.
5. **Finish.** Regenerate the derived `Timeline` nodes, set the modification time of every changed
   node (§9.5), and commit as `graph(retract): <D>`. Like apply, retraction is atomic.

Retraction is exact for every document ingested while the graph kept provenance. For a document
ingested before, `D` has no attributions, so steps 1–4 remove nothing but `D`'s own unattributed
node; such a document can only be retracted by `git revert`, which is correct for the most recent
commit alone.

### 8.10 Identity collisions

Two different subjects claiming one `@id` cannot be represented inside one node. Apply and combine
MUST stop, writing and committing nothing, and report each collision. It is resolved by
disambiguating one of the subjects under its class's identity rule (§5) — on its own branch,
rewriting the edges to it there — and retrying. This is the only condition under which a merge
refuses.

### 8.11 Guarantees

For a combine `M(base, A, B)`:

- **symmetric** — `M(base, A, B) = M(base, B, A)`: every tie is broken by canonical value order,
  never by which side is "ours";
- **identity** — `M(base, base, A) = A`, and **idempotent** — `M(base, A, A) = A`;
- **total** — it yields valid, conforming files, or stops on an identity collision; it never yields
  merge markers;
- **bounded** — the conflict structure has depth two (§8.2) and the provenance structure depth
  three (§8.3).

An apply only adds to candidate sets and attributions (**monotone**), changes nothing when repeated
(**idempotent**), and lands whole or not at all (**atomic**, §10.2). A retraction removes exactly the
values whose only attribution was the retracted document, and nothing unattributed (§8.9).

## 9. Version Control

Git **MUST** be the version-control system. Git is the authoritative record of *what* changed in the
graph; the format stores no change log. *When* each node entered and last changed is carried by the
node itself (§9.5), so a graph can be sliced by time without replaying git.

### 9.1 One document, one commit

Ingesting a document is exactly one commit containing its entire contribution: the new document
node, any new nodes, merged predicates on existing nodes, and the index entries for its period.

### 9.2 Operations

- **Ingest** — `git add -A` then `git commit -F <msg>`
- **Locate commit** — `git log --grep=<id>`
- **Node history** — `git log --follow -- '<path-to-node>'`
- **Idempotency** — before ingesting, check whether the document's node is already tracked. If it is
  and its `immutable` predicates equal the incoming document's, skip. If it is and they differ, the
  incoming document is a **different** document whose identity collides: the producer MUST
  disambiguate its `@id` per the class's identity rule, and MUST NOT skip it as already ingested.

**Retract** a document by the retraction operation (§8.9), which uses the attributions under
`@provenance` (§8.3) to remove exactly the values that document alone contributed, wherever it sits
in history. `git revert <commit>` remains available — it runs through the same merge driver (§9.4)
and never leaves markers — but it is correct only for the most recent document, since it cannot
tell a value only the reverted document supplied from one later documents also supplied.

### 9.3 Commit message format

```
graph(ingest): <id> — <title>

<one-line summary>

Nodes: +<counts by class>
Edges: +<n>
Timeline: <YYYY-MM>
Source-Id: <id>
Needs-Review: <node-id> <predicate>
```

The subject line names the operation:

| Subject                                 | Operation                             |
| --------------------------------------- | ------------------------------------- |
| `graph(ingest): <id> — <title>`         | applying one document's patch (§10.2) |
| `graph(combine): <branch>`              | combining a branch (§9.4)             |
| `graph(resolve): <node-id> <predicate>` | settling one conflict (§8.8)          |
| `graph(retract): <id>`                  | retracting one document (§8.9)        |

Mandatory: the subject line; for `ingest`, the `Nodes:` stats line and the `Source-Id:` trailer;
and for every operation, one `Needs-Review:` trailer per conflict it opened — each open alternative
newly recorded under `@conflicts`, and each prose replacement kept twice (§8.6, §8.7). The node's
`@conflicts` entry is the lasting record; the trailer announces it. Recommended: `Edges:`,
`Timeline:`, and any URL or date trailers.

### 9.4 Combining branches

Two branches of a graph (or two graphs) MUST be combined node by node under §8.7, never by a
line-based textual merge. A combine:

1. **Schema first.** If `_schema/` differs between the sides, it is combined first, in its own
   commit, and the nodes are then merged under the combined schema — the merge of a node needs one
   agreed merge value per predicate. A predicate whose own `merge` or `role` differs between the
   sides is a schema collision, and the combine stops.
2. **Nodes, by `@id`.** Nodes are matched by `@id`, never by path, since one node may sit in
   different folders on the two sides. Every node changed on both sides is merged per §8.7; a node
   created on both sides is merged against an empty `base`. A node deleted on one side and changed
   on the other is kept, changed, and reported with `Needs-Review:` — only retraction deletes a
   node, and a retraction that meets later changes is never applied silently. A node deleted on one
   side and unchanged on the other is deleted.
3. **Derived data.** `Timeline` nodes are regenerated from the merged nodes, never merged.
4. **Checks.** Graph-wide uniqueness (§5) — including one basename in two folders and an alias now
   claimed by two nodes — and conformance (§12).
5. **Commit** as `graph(combine): <branch>` (§9.3). If any step fails, nothing is committed.

**Git integration.** The per-node merge of §8.7 is implemented as a git **merge driver**, declared in
the graph's `.gitattributes`:

```
*.md          merge=arcnet
timeline/**   merge=ours
```

Git hands the driver the `base`, ours and theirs versions of one file — exactly the inputs of §8.7;
the driver writes the merged node and fails only on an identity collision. `git revert` and
`git cherry-pick` use the same driver. Git does not call a merge driver for a file deleted on one
side, for a node moved between folders, or for the schema-first ordering; a combine tool wrapped
around `git merge` performs those steps (1–5 above), and runs `git merge --abort` when a step fails.

### 9.5 Time

Time is a first-class dimension of the graph. Every node MUST carry two **universal** timestamps
(SCHEMA §3.4), both UTC `datetime` values:

- the **creation time** — when the node first entered the graph; merged as `earliest`;
- the **modification time** — when an accepted contribution, combine or resolution last changed
  the value in force of any other predicate of the node; merged as `latest`. Opening or closing a
  conflict alone (§8.2), or adding or removing an attribution alone (§8.3), does not change it. It is never earlier than the creation time.

Both are graph times (ingest times), unrelated to the production date of any document. The apply
tool sets them from a single ingest time per patch, and uses that same time as the commit's date,
so the node and git agree. A contribution that changes no other predicate of a node leaves its
modification time unchanged; this keeps re-applying a patch idempotent (§10.2).

This supports **time slices** without replaying git:

- **as-of `T`** — the nodes whose creation time is ≤ `T`: the set of subjects the graph knew at `T`;
- **changed in `[T1, T2]`** — the nodes whose modification time falls in the window;
- **new in `[T1, T2]`** — the nodes whose creation time falls in the window.

A node's *content* as of `T` is in git: the last commit dated ≤ `T` that touched the node.

## 10. Document Patch

A **patch** is a single Markdown file serializing one document's entire contribution. A tool
produces a patch from a document and applies it to any graph. A patch is a **parallel exchange
serialization, not part of the graph**: never indexed as a node, never stored under graph folders.

- A patch always carries **full nodes**, never deltas; reconciliation happens at apply time, by each
  predicate's merge behavior (§8.6).
- A patch carries values, never disputes, decisions or attributions: it MUST NOT carry
  `@conflicts` or `@provenance`, and it cannot remove a value. Every value it carries is attributed
  to the patch's `@id` on apply (§8.6).
- A patch renders as ordinary Markdown anywhere.

### 10.1 Structure

A YAML front-matter **manifest** followed by a **graph body** whose nodes are grouped by class. The
manifest's mandatory keys are `"@type": Patch`, `"@id"` — the contributing document node's `@id` —
and `published`, the document's production date. `Patch` is a format keyword, not a graph class.

- **H1 = the node's `@type`**, one per class present.
- **H2 = the node's `@id`**, one per node. `@type` comes from the H1 and `@id` from the H2; neither
  is repeated below.
- **Under each H2:** a fenced `yaml` block with the node's remaining `meta`-role predicates, then
  the node body as §4 lays it out, except that every `## Label` heading is written as a bold
  `**Label**` line.

Markdown headings are reserved for class and identity; node bodies use bold labels, never headings.
The exchange format does not carry index nodes; the apply tool derives them.

### 10.2 Apply

1. For each H2 node under each H1 class, reconstruct the node.
2. If the `@id` does not exist in the target graph, create the file.
3. If it exists with a different `@type`, or an `immutable` predicate diverges, fail the whole
   apply: nothing is written and nothing is committed (§8.10).
4. Otherwise merge the node as a contribution (§8.6), recording every new open alternative under
   `@conflicts` and in a `Needs-Review:` trailer, and every attribution to the patch's `@id` under
   `@provenance`.
5. Set the creation time of every new node, and the modification time of every node the apply
   changed, to the patch's ingest time (§9.5).
6. Update the period index from the document's production date.
7. Commit per §9 — applying one patch is exactly one commit.

A patch is **idempotent**: applying it twice yields the same graph. Applying a patch is atomic:
either the whole contribution lands in one commit, or nothing does.

## 11. Design Invariants

A conforming graph MUST uphold these:

1. **One node, one file.** A YAML front-matter header and a Markdown body. Every file in the graph
   is a node.
2. **Identity is the `@id`**, equal to the basename, unique graph-wide under case-insensitive
   comparison. Links resolve by basename, independent of folder.
3. **Edges are wiki-links**, optionally typed by a predicate.
4. **Predicates are first class.** Every predicate used anywhere — front-matter field, body edge, or
   citation type — is itself a node under `_schema/Property/`. Only the keywords `@id`, `@type`,
   `@conflicts` and `@provenance` are exempt.
5. **Classes are first class.** Every `@type` value in use is itself a node under `_schema/Class/`.
6. **Derived nodes carry provenance.** A node distilled from a document links to the document
   node(s) it came from, and every contributed value is attributed to the documents that supplied it
   (§8.3), so any document can be retracted exactly (§8.9).
7. **Classification is data, not location.** `@type` and every other classification lives in
   predicates; folders mirror it but MUST NOT be its sole carrier.
8. **Merges are on values, never on text; conflicts are data.** Every merge parses nodes and writes
   canonical nodes (§8), so no file ever holds merge markers. Disagreement is recorded beside the
   value in force, in the flat `@conflicts` keyword, and never nests. Apply is monotone and
   idempotent; combine is symmetric and idempotent; both are atomic.
9. **Git is the history; time is first class.** The format stores no change log, but every node
   carries its own creation and modification time (§9.5), so the graph can be sliced by time.
10. **Prose declares nothing.** Every normative statement about a predicate or a class lives in that
    predicate's or class's own schema node. A specification document that states a rule a schema node
    could carry is a defect in the specification, not a source of truth.

## 12. Conformance

Mechanism-level checks. Vocabulary-level checks are in [`SCHEMA.md`](SCHEMA.md).

- [ ] Every file is one node with valid YAML front-matter and a
      mandatory `@id` and `@type`.
- [ ] Every node carries a creation and a modification time (§9.5), UTC, and the modification
      time is not earlier than the creation time.
- [ ] Every basename equals its node's `@id`, is unique graph-wide under case-insensitive NFC
      comparison, and contains none of `/ \ : * ? " < > | . # ^ [ ]`.
- [ ] Every alias resolves to exactly one node: it equals no other node's `@id` or alias (§5).
- [ ] Every `[[link]]` resolves to some node's basename.
- [ ] Every predicate name is camelCase; every `@type` value is UpperCamelCase.
- [ ] Every class folder's name is character-for-character identical to its class name.
- [ ] Every predicate in use renders in the position its `role` declares.
- [ ] No prose contains a heading or a `predicate::` bullet line.
- [ ] Each predicate's merge behavior is one of §8.4's eight.
- [ ] No file contains git merge markers (lines beginning `<<<<<<<`, `=======` or `>>>>>>>` outside
      fenced code).
- [ ] `@conflicts`, where present, is a non-empty map whose keys are single-valued predicates the
      node carries, and whose arrays are non-empty, hold values legal for that predicate, are in
      canonical order, and never contain the value in force (§8.2).
- [ ] `@provenance`, where present, is a non-empty map from the `@id` of an existing node to a
      non-empty map of predicates the node carries, each to a non-empty array of values that are
      in that predicate's candidate set — `text`-role values as digests — in canonical order
      (§8.3). The universal timestamps are never attributed.
- [ ] The graph's `.gitattributes` routes node files to the merge driver (§9.4).
- [ ] Each document was ingested as exactly one git commit.
