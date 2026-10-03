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

A knowledge graph is a set of linked Markdown files. Each file is a node; `[[wiki-links]]` between
files are edges. Underneath the Markdown, every node is a **bag of predicates about one subject** —
the same statement an RDF triple makes — and Markdown is a lossless, human-readable serialization of
that bag.

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

```markdown
---
"@id": Transport Layer Security
"@type": Entity
category: [independent, abstract, occurrent, script]
tags: [cryptography]
---
# Transport Layer Security

A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted
network.

- replaces:: [[SSL Protocol]]
- conformsTo:: [[RFC 8446]]

## mentionedIn
- mentionedIn:: [[rescorla-2026-tls13]]
```

```nt
"Transport Layer Security"
  a "Entity"
  category "independent" ; category "abstract" ; category "occurrent" ; category "script"
  tags "cryptography"
  text "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network."
  replaces "SSL Protocol"
  conformsTo "RFC 8446"
  mentionedIn "rescorla-2026-tls13"
  .
```

The conversion is lossless and reversible in both directions — Markdown → triples → Markdown
reproduces the original file, and triples → Markdown → triples reproduces the original bag — because
every predicate's rendering position is fixed once, on the predicate's own schema node, not decided
ad hoc per document. §4 gives the five positions a predicate can render to.

This inverts the classical RDFS reading for a practical reason. `rdfs:domain` states which classes a
property's subjects belong to; for a document-ingestion graph the checkable direction runs the other
way — a **class declares which predicates its instances require**. The schema layer's `Class` and
`Property` nodes are exactly `rdfs:Class` and `rdf:Property`, authored for that direction. Only the
range half of an edge signature is declared explicitly, because only the range half is not already
implied by the classes that use the predicate.

## 4. Node Model

Every node is a YAML front-matter block (delimited by `---`) followed by a Markdown body. Both are
constructed from the same bag of predicates about the node's subject; the split between front-matter
and body, and the exact placement within the body, is fixed by each predicate's declared **role** —
an attribute of the predicate's own schema node, not a convention applied per document:

- **`meta`** — a front-matter scalar or array, in the YAML block.
- **`text`** — prose, as a paragraph in the body.
- **`href`** — an inline, **untyped** wiki-link, inside a `text` predicate's prose.
- **`edge`** — a typed edge, a bullet in a heterogeneous block: `predicate:: [[Target]]`.
- **`link`** — a typed, homogeneous edge list, its own `## Predicate` block, one bullet per target.

Because every predicate's role is declared exactly once, reconstructing a node's Markdown from its
triples — or recovering its triples by reading the Markdown — needs no per-class special-casing: a
consumer resolves each predicate's role and the rendering follows mechanically. Consumers MUST
preserve unknown predicates, so the format tolerates a vocabulary that grows over time.

Concretely:

- **Front-matter** carries every predicate with role `meta`, including the mandatory `@id` and
  `@type`. A predicate name beginning with `@` MUST be written as a quoted YAML string key —
  `"@id"`, `"@type"` — never bare. YAML reserves the unquoted `@` character as an indicator for
  future use, so a bare `@id:` key is invalid or inconsistently accepted across parsers.
- **Body** carries:
  - prose, from `text`-role predicates. Start with the default `text` predicate's value, then each
    further `text`-role predicate, prefixed by its bold label.
  - inline `[[link]]`s, from `href`-role predicates — untyped mentions embedded in prose. A `text`
    predicate's own statement is never replaced by an embedded link, only annotated by it.
  - one heterogeneous block of `predicate:: [[Target]]` bullets, holding every `edge`-role predicate
    in use, grouped under a bold label where that aids reading.
  - one `## Predicate` block per `link`-role predicate in use, its bullets written exactly as
    `edge`-role bullets are. The heading groups them for display; it does not change the bullet
    syntax. Where a class's body consists of exactly one `link`-role predicate, the heading MAY be
    omitted.

## 5. Identity

A node's identity is its `@id`, equal to its basename, and MUST be unique across the whole graph.

The identity MUST NOT contain any of `/ \ : * ? " < > | .` — ten characters that make an identity
unusable as a filename. Restated as a whitelist, which is easier to satisfy: **letters, digits,
spaces, hyphens, and parentheses.** A colliding identity MUST be disambiguated with a parenthetical
qualifier, never with a slash or a colon.

Use aliases to preserve original identity if it contains forbidden characters.

**How each class forms its `@id` is declared by that class**, in its `identityRule` predicate. CORE
fixes only the character set and the uniqueness requirement.

A subject named differently across documents has exactly one **canonical** node; alternatives are
recorded in the node's alias predicate and in `_schema/aliases.md`, a plain aggregate file rather
than a node. Producers MUST consult the alias table before creating a node, so synonyms collapse.

Predicate names MUST be **camelCase**; class names MUST be **UpperCamelCase**. The two vocabularies
are therefore distinguishable wherever they appear, including bare in prose.


## 6. Folder Structure

Each class has exactly one folder and each folder holds exactly one class. A class's folder name
MUST be **character-for-character identical to the class name** — same case, no pluralization, no
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
    └── aliases.md        #   alias table — a plain file, not a node
```

Two folders are exempt because neither is a class folder: `_schema/` is a namespace prefix whose
children are class folders and do follow the rule; `timeline/` is an index whose nodes are bucketed
by granularity rather than filed flat.

Folders remain a filing convenience: because links resolve by basename, a node may be re-filed
without breaking any edge, and its classification lives in `@type`, not in its location. A consumer
MUST read `@type` from the node and MUST NOT infer it from the folder — the naming rule makes the
folder a reliable *mirror* of the class, never a substitute for it.

## 7. Edges

An edge is `[[Target]]`, a basename reference; it MAY carry a display alias `[[Target|text]]`. A
bare `[[Target]]` with no predicate is an `href`-role, untyped mention.

A predicate types an edge. Three forms are permitted; producers SHOULD use the list form for
`edge`-role predicates and MUST use the heading form for `link`-role predicates:

- **List form:** `- replaces:: [[SSL Protocol]]`
- **Body form:** `conformsTo:: [[RFC 8446]]`
- **Inline form:** `... was [citesAsEvidence:: [[RFC 8446]]] standardized.`

The `::` token separates predicate from target.


## 8. Merge Algebra

When an ingested document contributes to a node whose `@id` already exists, each predicate on that
contribution is merged per its own declared `merge` value. **All merges MUST be commutative and
idempotent**, so replay, reordering, and rollback are well-defined. A predicate declares exactly one
of:

- **`immutable`** — single-valued, permanently fixed by the first accepted write. Later writes
  cannot change it; divergent contributions are rejected per the conflict policy.
- **`union`** — merge by set union, deduplicated per the predicate's equality semantics.
- **`firstWriteWin`** — single-valued. The first accepted write establishes the value; later
  divergent contributions do not overwrite it and produce `needsReview`.
- **`fillIfEmpty`** — single-valued, initially absent. The first write fills it; once present,
  later contributions have no effect and produce no conflict.
- **`lastWriteWin`** — single-valued mutable state. Each accepted write replaces the current value.
- **`append`** — accumulate without removing existing content. For keyed lists, add only unseen
  keys while preserving order; for `text` roles, append new prose per the predicate's concatenation
  rules.

A class-level "this node never changes after creation" behavior is not a separate merge value — it
emerges from every one of that class's predicates independently being `immutable`, reinforced by the
ingestion idempotency check (§9.2).

## 9. Version Control

Git **MUST** be the version-control system. Git is the authoritative record of how the graph
changed; the format stores no change log.

### 9.1 One document, one commit

Ingesting a document is exactly one commit containing its entire contribution: the new document
node, any new nodes, merged predicates on existing nodes, and the index entries for its period.

### 9.2 Operations

- **Ingest** — `git add -A` then `git commit -F <msg>`
- **Locate commit** — `git log --grep=<id>`
- **Node history** — `git log --follow -- '<path-to-node>'`
- **Idempotency** — before ingesting, check whether the document's node is already tracked; skip if
  present.

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
```

Mandatory: the subject line (retraction uses `graph(retract): <id>`), the `Nodes:` stats line, and
the `Source-Id:` trailer. Recommended: `Edges:`, `Timeline:`, and any URL or date trailers.

## 10. Document Patch

A **patch** is a single Markdown file serializing one document's entire contribution. A tool
produces a patch from a document and applies it to any graph. A patch is a **parallel exchange
serialization, not part of the graph**: never indexed as a node, never stored under graph folders.

- A patch always carries **full nodes**, never deltas; reconciliation happens at apply time, by each
  predicate's merge behavior.
- A patch renders as ordinary Markdown anywhere.

### 10.1 Structure

A YAML front-matter **manifest** (mandatory: `@type: patch`, the document id, its production date)
followed by a **graph body** whose nodes are grouped by class.

- **H1 = the node's `@type`**, one per class present.
- **H2 = the node's `@id`**, one per node. `@type` comes from the H1 and `@id` from the H2; neither
  is repeated below.
- **Under each H2:** a fenced `yaml` block with the node's remaining `meta`-role predicates, then
  the node body — the `text`-role predicate as an unlabeled paragraph, each `link`-role predicate as
  a bold label followed by its bullets, and `edge`-role predicates as one heterogeneous bullet
  block.

Markdown headings are reserved for class and identity; node bodies use bold labels, never headings.
The exchange format does not carry index nodes; the apply tool derives them.

### 10.2 Apply

1. For each H2 node under each H1 class, reconstruct the node.
2. If the `@id` does not exist in the target graph, create the file.
3. If it exists, apply each predicate's merge behavior.
4. Update the period index from the document's production date.
5. Commit per §9 — applying one patch is exactly one commit.

A patch is **idempotent**: applying it twice yields the same graph.

## 11. Design Invariants

A conforming graph MUST uphold these:

1. **One node, one file.** A YAML front-matter header and a Markdown body.
2. **Identity is the `@id`**, equal to the basename, unique graph-wide. Links resolve by basename,
   independent of folder.
3. **Edges are wiki-links**, optionally typed by a predicate.
4. **Predicates are first class.** Every predicate used anywhere — front-matter field, body edge, or
   citation type — is itself a node under `_schema/Property/`.
5. **Classes are first class.** Every `@type` value in use is itself a node under `_schema/Class/`.
6. **Derived nodes carry provenance.** A node distilled from a document links to the document
   node(s) it came from.
7. **Classification is data, not location.** `@type` and every other classification lives in
   predicates; folders mirror it but MUST NOT be its sole carrier.
8. **Append-only growth, defined merges.** Merges are commutative and idempotent at the predicate
   level, so replay, reordering, and rollback are well-defined.
9. **Git is the history.** The format stores no change log.
10. **Prose declares nothing.** Every normative statement about a predicate or a class lives in that
    predicate's or class's own schema node. A specification document that states a rule a schema node
    could carry is a defect in the specification, not a source of truth.

## 12. Conformance

Mechanism-level checks. Vocabulary-level checks are in [`SCHEMA.md`](SCHEMA.md).

- [ ] Every file is one node with valid YAML front-matter and a mandatory `@id` and `@type`.
- [ ] Every basename equals its node's `@id`, is unique, and contains none of `/ \ : * ? " < > | .`
- [ ] Every `[[link]]` resolves to some node's basename.
- [ ] Every predicate name is camelCase; every `@type` value is UpperCamelCase.
- [ ] Every class folder's name is character-for-character identical to its class name.
- [ ] Every predicate in use renders in the position its `role` declares.
- [ ] Each predicate's merge behavior is one of §8's six and is commutative and idempotent.
- [ ] Each document was ingested as exactly one git commit.
