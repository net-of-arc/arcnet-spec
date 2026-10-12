---
"@id": permitsMany
"@type": Property
role: link
merge: union
label: "Permits Many"
bundle: core
---
# permitsMany

A conforming instance MAY carry any number of values of the target predicate, including none (cardinality `0..n`). The target predicate MUST have an accumulating merge (SCHEMA §3.1).

## Object
- object:: [[Property]]
