# ARCNET — Specification

A **file format and folder convention** for a knowledge graph stored as plain Markdown. It depends on no program, library, or language. Any producer and any consumer interoperate through the format.

## Three layers

The graph specification consists of three layers.

|                    | Document                                                        | Holds                                                                   | Normative?                                                      |
| ------------------ | --------------------------------------------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| **1 · Mechanism**  | [`CORE.md`](CORE.md)                                            | how a node is written, identified, linked, merged, versioned, exchanged | yes                                                             |
| **2 · Vocabulary** | [`SCHEMA.md`](SCHEMA.md) + [`domain/*/Class,Property`](domain/) | every predicate and every class, as nodes                               | yes — each bundle's `Class/`/`Property/` folder is the registry |
| **3 · Narrative**  | [`domain/*/README.md`](domain/)                                 | why a vocabulary is shaped as it is, how to map a document onto it      | **no**                                                          |

Read `CORE.md` to implement a parser. Read a bundle's `Class/` / `Property/` folder to learn what anything means. Read its `README.md` to understand why.

```
spec/
├── CORE.md                    # layer 1: mechanism only, no vocabulary
├── SCHEMA.md                  # layer 2: the two node formats and the meta-vocabulary
└── domain/                    # one self-contained folder per bundle
    ├── core/                  #   the core vocabulary every graph needs
    │   ├── README.md          #     layer 3: narrative
    │   ├── Class/             #     layer 2: 7 class nodes
    │   └── Property/          #     layer 2: 49 predicate nodes
    ├── domainA/               #   example place holder 
    ├── domainB/               #   example place holder 
    ├── .../                   #   example place holder 
    └── domainZ/               #   example place holder 
```

Each bundle folder is self-contained and independently readable: its `Class/` and `Property/` holds exactly the semantic layer, its `README.md` explains why, and nothing under it depends on another domain unless explicitly specified.

## Meta-vocabulary

Thirteen predicates were added so that a domain's semantics become strict and machine-checkable:

| Predicate                                               | Absorbs                                                          | Why it is needed                                                                           |
| ------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `requiresOne` `requiresMany` `permitsOne` `permitsMany` | the cardinality columns of the domain profiles' predicate tables | distinguish `1` from `1..n`, and whether a predicate is mandatory varies by class          |
| `object`                                                | the `from → to:` annotations                                     | an expected range no producer could read                                                   |
| `inverseOf`                                             | "the inverse of …" sentences in predicate descriptions           | a backlink rule no checker could read                                                      |
| `datatype`, `oneOf`                                     | "ISO-8601 date", the twelve Sowa categories, enums               | value constraints stated only in sentences                                                 |
| `identityRule`                                          | each profile's identity statement                                | identity is a class-level rule and belongs on the class                                    |
| `defaultText`                                           | "the unlabelled paragraph is …" conventions                      | which prose renders unlabelled is a class-level choice, not a property of the predicate    |
| `subClassOf`                                            | copy-pasted membership blocks across related classes             | a class extends another by adding or tightening predicates, never by repeating them        |
| `constraint`                                            | the profiles' normative structural rules                         | e.g. the timestamp ordering, the impact prefixes, the contributing-factor path restriction |
| `bundle`                                                | the adoption story in [`SCHEMA.md`](SCHEMA.md) §6                | which nodes a graph installs together                                                      |
