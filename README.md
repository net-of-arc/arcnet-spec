# ARCNET : Specification for Knowledge Graph

This document specifies a **file format and folder convention** for a knowledge graph stored as plain Markdown. It is **tool-agnostic**: it depends on no program, library, or language. Any producer (a person in an editor, or an extraction program) and any consumer (Obsidian, Juggl, a static-site generator, `grep`, a custom indexer) interoperate through this format.


## Documents

### Core

|                        | Document                                                               | Holds                                                                   |
| ---------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **1 · Mechanism**      | [`CORE.md`](./spec/CORE.md)                                            | how a node is written, identified, linked, merged, versioned, exchanged |
| **2 · Vocabulary**     | [`SCHEMA.md`](./spec/SCHEMA.md) + [`domain/*/Class,Property`](domain/) | every predicate and every class, as nodes to defined the graph shape    |
| **3 · Core Narrative** | [`domain/core/README.md`](./spec/domain/core/README.md)                | why a core vocabulary is shaped as it is, how to map a document onto it |



## License

[![See LICENSE](https://img.shields.io/github/license/net-of-arc/arcnet-spec.svg?style=for-the-badge)](LICENSE)

