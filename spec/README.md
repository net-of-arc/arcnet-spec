# ARCNET — Specification

A **file format and folder convention** for a knowledge graph stored as plain Markdown.It depends on no program, library, or language. Any producer and any consumer interoperate through the format.

## Three layers

The graph specification consists of three layers.

|                    | Document                                                        | Holds                                                                   | Normative?                                                      |
| ------------------ | --------------------------------------------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| **1 · Mechanism**  | [`CORE.md`](CORE.md)                                            | how a node is written, identified, linked, merged, versioned, exchanged | yes                                                             |
| **2 · Vocabulary** | [`SCHEMA.md`](SCHEMA.md) + [`domain/*/Class,Property`](domain/) | every predicate and every class, as nodes                               | yes — each bundle's `Class/`/`Property/` folder is the registry |
| **3 · Narrative**  | [`domain/*/README.md`](domain/)                                 | semantic layer and a vocabulary per problem domain                      | **no**                                                          |

Read `CORE.md` to implement a parser. Read a bundle's `Class/` / `Property/` folder to learn what anything means. Read its `README.md` to understand why.

```
spec/
├── CORE.md                    # layer 1: mechanism only, no vocabulary
├── SCHEMA.md                  # layer 2: the two node formats and the meta-vocabulary
└── domain/                    # layer 3: self-contained domains
    ├── core/                  #   the core vocabulary every graph needs
    │   ├── README.md          #     
    │   ├── Class/             #     7 class nodes
    │   └── Property/          #     51 predicate nodes
    ├── article/
    │   ├── README.md
    │   ├── Class/              #     2 class nodes
    │   └── Property/           #     11 predicate nodes
    ├── core-thought/
    │   ├── README.md
    │   ├── Class/              #     1 class node
    │   └── Property/           #     6 predicate nodes
    ├── incident/
    │   ├── README.md
    │   ├── Class/              #     5 class nodes
    │   └── Property/           #     23 predicate nodes
    └── universal/
        ├── README.md
        ├── Class/              #     5 class nodes
        └── Property/           #     11 predicate nodes
```

Each bundle folder is self-contained and independently readable: its `Class/` and `Property/` holds exactly the semantic layer, its `README.md` explains why, and nothing under it depends on another domain unless explicitly specified.  

## Meta-vocabulary

Eleven predicates were added so that domain's semantic became a strict:

| Predicate                                               | Absorbs                                                          | Why it needed                                                                              |
| ------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `requiresOne` `requiresMany` `permitsOne` `permitsMany` | the cardinality columns of the domain profiles' predicate tables | distinguish `1` from `1..n`, and cardinality varies by class                               |
| `object`                                                | the `from → to:` annotations                                     | a range no checker could read                                                              |
| `datatype`, `permits`                                   | "ISO-8601 date", the twelve Sowa combinations, enums             | value constraints stated only in sentences                                                 |
| `identityRule`                                          | each profile's identity statement                                | identity is a class-level rule and belongs on the class                                    |
| `constraint`                                            | the profiles' normative structural rules                         | e.g. the timestamp ordering, the impact prefixes, the contributing-factor path restriction |
| `bundle`                                                | the adoption story in `schema/README.md`                         | which nodes a graph installs together                                                      |

