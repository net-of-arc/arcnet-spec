---
"@id": merge
"@type": Property
role: meta
merge: immutable
datatype: enum
bundle: core
---
# merge

One of the merge behaviors (§9.3): how contributions to this predicate combine.

## Permits
- permits:: immutable
- permits:: union
- permits:: firstWriteWin
- permits:: fillIfEmpty
- permits:: lastWriteWin
- permits:: append
