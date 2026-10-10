# CORE — Mechanism

**Status:** Draft · **Layer:** 1 of 3

This document specifies the **mechanism** of a knowledge graph stored as plain Markdown: the
RDF-aligned data model, the node model, identity, folder layout, edges, the merge algebra, version
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
have no `Property` node, and are never listed by any class.

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
  producer SHOULD write them in a stable order (the `permits` order for enumerations, otherwise
  lexicographic) so that independent writers produce identical files;
- typed edges are written in the list form (§7);
- untyped links stay inside the prose that carries them and produce no triple of their own.

The specifcation inverts the classical RDFS reading for a practical reason. `rdfs:domain` states which classes a
property's subjects belong to; for a document-ingestion graph the checkable direction runs the other
way — a **class declares which predicates its instances require**. The schema layer's `Class` and
`Property` nodes are exactly `rdfs:Class` and `rdf:Property`, authored for that direction. Only the
range half of an edge signature is declared explicitly, because only the range half is not already
implied by the classes that use the predicate.

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
  `@type` keywords. A key beginning with `@` MUST be written as a quoted YAML string key —
  `"@id"`, `"@type"` — never bare. YAML reserves the unquoted `@` character as an indicator for
  future use, so a bare `@id:` key is invalid or inconsistently accepted across parsers.
- **Body** carries, in this order:
  1. the H1 title, equal to the `@id`;
  2. prose from the one `text`-role predicate the class lists without a `label` — the node's
     unlabelled paragraph(s);
  3. each further single-valued `text`-role predicate, as its bold `**Label**` on its own line
     followed by its paragraph;
  4. one heterogeneous block of `predicate:: [[Target]]` bullets, holding every `edge`-role
     predicate in use, optionally grouped under bold labels where that aids reading;
  5. one `## Label` block per `link`-role predicate in use, its bullets written exactly as
     `edge`-role bullets are, and one `## Label` block per `text`-role predicate whose merge is
     `union` (a list of statements), one `predicate:: statement` bullet per value. The heading
     groups the bullets for display; it does not change the bullet syntax and carries no data.
     Where a class's body consists of exactly one `link`-role predicate, the heading MAY be omitted.

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

Use aliases to preserve original identity if it contains forbidden characters.

**How each class forms its `@id` is declared by that class**, in its `identityRule` predicate. CORE
fixes only the character set and the uniqueness requirement. An identity rule MUST be a function of
the subject alone, never of ingestion order: "whichever arrives second gets a suffix" makes the
graph depend on the order documents were ingested in.

<!-- 

TODO: decision about aliases is required

-->
A subject named differently across documents has exactly one **canonical** node; alternatives are
recorded in the node's alias predicate. An alias MUST resolve to exactly one node: it MUST NOT
equal, under the same comparison as `@id`, any other node's `@id` or alias. `_schema/aliases.md` is
a **derived** lookup table aggregating every node's aliases — a plain file, not a node; the alias
predicates are authoritative and the table MUST be regenerable from them. Producers MUST resolve
names against the aliases before creating a node, so synonyms collapse.

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
    ├── Class/            #   one node per class
    └── aliases.md        #   derived alias table — a plain file, not a node
```

Two folders are exempt because neither is a class folder: `_schema/` is a namespace prefix whose
children are class folders and do follow the rule; `timeline/` is an index whose nodes are bucketed
by granularity rather than filed flat. `_schema/aliases.md` is the only file in the graph that is
not a node.

Folders exist purely for filing convenience. Because links resolve strictly by file basename, moving a node to another folder will never break an edge. A consumer MUST determine a node's class using its internal `@type` property. Consumers MUST NOT infer classification from folder location. The folder structure mirrors the class for convenience, but never replaces it.

Because `@id` uniqueness is graph-wide, two files sharing the same basename—even in different folders—is a conformance error, not two distinct nodes.


## 7. Edges

An edge is `[[Target]]`, a basename reference; it MAY carry a display alias `[[Target|text]]`. The
display alias is presentation only: two edges are equal when their targets resolve to the same
canonical `@id` (§5), whatever their display text or the alias used to write them. A bare
`[[Target]]` with no predicate is an untyped link (§4).

A predicate types an edge. Three forms are recognized; consumers MUST read all three:

- **List form:** `- replaces:: [[SSL Protocol]]` — the canonical form.
- **Body form:** `conformsTo:: [[RFC 8446]]`
- **Inline form:** `... was [citesAsEvidence:: [[RFC 8446]]] standardized.`

Producers SHOULD use the list form for `edge`-role predicates, and MUST use the list form inside a
`## Label` block for `link`-role predicates (subject to the single-block exemption in §4). A round
trip through triples MAY normalize body and inline forms to the list form.

The `::` token separates predicate from target.


## 8. Merge Algebra
<!-- 

TODO: Decision about Merge Algebra is required.

-->

When an ingested document contributes to a node whose `@id` already exists, each predicate on that
contribution is merged per its own declared `merge` value. A predicate declares exactly one of:

- **`immutable`** — single-valued, fixed by the first accepted write. A divergent contribution means
  the incoming node is **not the same subject**: it is an identity collision (§5), and the apply
  MUST fail without committing (§10.2).
- **`union`** — merge by set union, deduplicated by value equality (below).
- **`firstWriteWin`** — single-valued. The first accepted write establishes the value; a later
  divergent contribution does not overwrite it and is reported as **Needs-Review** (§9.3).
- **`fillIfEmpty`** — single-valued, initially absent. The first write fills it; once present,
  later contributions have no effect and produce no report.
- **`lastWriteWin`** — single-valued mutable state. Each accepted write replaces the current value.
- **`earliest`** — single-valued, ordered literal (a `date` or `datetime`). The smallest value
  contributed is kept.
- **`latest`** — single-valued, ordered literal. The greatest value contributed is kept.
- **`append`** — accumulate prose without removing existing content. The value is the set of its
  paragraphs; a contributed paragraph already present (compared after whitespace normalization) is
  not appended again. New paragraphs are appended in contribution order.

**Value equality.** Literals compare by exact string equality after trimming surrounding
whitespace; `date` literals compare by value. Edge targets compare by resolved canonical `@id` (§7).

**Order and determinism.** Every merge MUST be **idempotent**. `union`, `append`, `earliest` and
`latest` are also **commutative**. The other four single-valued behaviours are commutative only over
agreeing contributions;
when contributions diverge, the retained value depends on **contribution order**, which is defined
as the commit order on the branch being written to, and, when two branches are combined (§9.4), by
the contributing documents' production dates, ties broken by the documents' `@id`. Under that order,
replay and reordering are well-defined, and every divergence is reported regardless of the order in
which it arrived.

**Unknown predicates** — present on a node but registered in no adopted bundle — are preserved in
the position they were found and merged as `firstWriteWin`.

A class-level "this node never changes after creation" behavior is not a separate merge value — it
emerges from every one of that class's predicates independently being `immutable`, reinforced by the
ingestion idempotency check (§9.2).

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

**Retract** via `git revert <commit>` is reliable only for the most recent commit. Retracting a
document from the middle of history requires analysis: if a node's front-matter was established by
that document and later merged into by others, a plain revert removes predicates the later documents
depend on and corrupts the node.

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

Mandatory: the subject line (retraction uses `graph(retract): <id>`), the `Nodes:` stats line, the
`Source-Id:` trailer, and one `Needs-Review:` trailer per divergent `firstWriteWin` contribution
(§8). Recommended: `Edges:`, `Timeline:`, and any URL or date trailers.

### 9.4 Combining branches

Two branches of a graph (or two graphs) MUST be combined node by node under §8, never by accepting
a line-based textual merge as is. A line-based `git merge` MAY be used only when it reports no
conflict **and** the result passes conformance; otherwise, for every node touched on both sides:

1. if the two sides carry different `@type` for one `@id`, or diverge on an `immutable` predicate,
   it is an identity collision — the combine MUST stop until the colliding identity is
   disambiguated;
2. otherwise, merge each predicate per §8 using the contribution order defined there;
3. regenerate derived data — `Timeline` nodes and `_schema/aliases.md` — from the merged nodes;
4. re-check graph-wide uniqueness (§5), including the same basename appearing in two folders and an
   alias now claimed by two nodes.

A textual conflict on a node's creation or modification time is always resolved by its merge
(`earliest`/`latest`), never by hand.

### 9.5 Time

Time is a first-class dimension of the graph. Every node MUST carry two **universal** timestamps
(SCHEMA §3.4), both UTC `datetime` values:

- the **creation time** — when the node first entered the graph; merged as `earliest`;
- the **modification time** — when an accepted contribution last changed any other predicate of
  the node; merged as `latest`. It is never earlier than the creation time.

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
  predicate's merge behavior.
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
   apply: nothing is written and nothing is committed (§8).
4. Otherwise apply each predicate's merge behavior.
5. Set the creation time of every new node, and the modification time of every node the apply
   changed, to the patch's ingest time (§9.5).
6. Update the period index from the document's production date.
7. Commit per §9 — applying one patch is exactly one commit.

A patch is **idempotent**: applying it twice yields the same graph. Applying a patch is atomic:
either the whole contribution lands in one commit, or nothing does.

## 11. Design Invariants

A conforming graph MUST uphold these:

1. **One node, one file.** A YAML front-matter header and a Markdown body. The only non-node file is
   the derived `_schema/aliases.md`.
2. **Identity is the `@id`**, equal to the basename, unique graph-wide under case-insensitive
   comparison. Links resolve by basename, independent of folder.
3. **Edges are wiki-links**, optionally typed by a predicate.
4. **Predicates are first class.** Every predicate used anywhere — front-matter field, body edge, or
   citation type — is itself a node under `_schema/Property/`. Only the keywords `@id` and `@type`
   are exempt.
5. **Classes are first class.** Every `@type` value in use is itself a node under `_schema/Class/`.
6. **Derived nodes carry provenance.** A node distilled from a document links to the document
   node(s) it came from.
7. **Classification is data, not location.** `@type` and every other classification lives in
   predicates; folders mirror it but MUST NOT be its sole carrier.
8. **Append-only growth, defined merges.** Merges are idempotent, commutative where §8 says so, and
   deterministic under contribution order everywhere else, so replay, reordering, and rollback are
   well-defined.
9. **Git is the history; time is first class.** The format stores no change log, but every node
   carries its own creation and modification time (§9.5), so the graph can be sliced by time.
10. **Prose declares nothing.** Every normative statement about a predicate or a class lives in that
    predicate's or class's own schema node. A specification document that states a rule a schema node
    could carry is a defect in the specification, not a source of truth.

## 12. Conformance

Mechanism-level checks. Vocabulary-level checks are in [`SCHEMA.md`](SCHEMA.md).

- [ ] Every file except `_schema/aliases.md` is one node with valid YAML front-matter and a
      mandatory `@id` and `@type`.
- [ ] Every node carries a creation and a modification time (§9.5), UTC, and the modification
      time is not earlier than the creation time.
- [ ] Every basename equals its node's `@id`, is unique graph-wide under case-insensitive NFC
      comparison, and contains none of `/ \ : * ? " < > | . # ^ [ ]`.
- [ ] Every alias resolves to exactly one node.
- [ ] Every `[[link]]` resolves to some node's basename.
- [ ] Every predicate name is camelCase; every `@type` value is UpperCamelCase.
- [ ] Every class folder's name is character-for-character identical to its class name.
- [ ] Every predicate in use renders in the position its `role` declares.
- [ ] No prose contains a heading or a `predicate::` bullet line.
- [ ] Each predicate's merge behavior is one of §8's eight.
- [ ] Each document was ingested as exactly one git commit.
