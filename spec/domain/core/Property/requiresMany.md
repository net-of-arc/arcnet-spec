---
"@id": requiresMany
"@type": Property
role: link
merge: union
label: "Requires Many"
bundle: core
---
# requiresMany

A conforming instance MUST carry at least one value of the target predicate, with no upper bound (cardinality `1..n`). The target predicate MUST have an accumulating merge (SCHEMA §3.1).

## Object
- object:: [[Property]]
