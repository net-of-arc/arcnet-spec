---
"@id": sameAs
"@type": Property
role: edge
merge: union
aligned: "owl:sameAs"
bundle: core
---
# sameAs

**Same work, two records.** `X sameAs:: Y` asserts that `X` and `Y` denote the same work: typically a `Reference`, named as citing documents name the work (`RFC 8446`), and the `Source` the graph made when it ingested that work (`rescorla-2018-tls13`). The two nodes stay separate, so every edge written to either keeps resolving, and each keeps its own predicates: a `Reference`'s `title` records how the work is cited, a `Source`'s how it was published. Symmetric: whichever side carries it, the other MUST carry it back (`inverseOf` itself). *e.g.* `RFC 8446` → `sameAs::` `rescorla-2018-tls13`.

## Object
- object:: [[Reference]]
- object:: [[Source]]

## Inverse Of
- inverseOf:: [[sameAs]]
