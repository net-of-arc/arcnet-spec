---
"@id": mentionedIn
"@type": Property
role: link
merge: union
label: "Mentioned In"
aligned: "schema:subjectOf"
bundle: core
---
# mentionedIn

A derived node's provenance backlink to the source it was drawn from (`Entity` and `Resource` both carry it) — recorded under the node's own `## Mentioned In` block.

## Object
- object:: [[Source]]

## Inverse Of
- inverseOf:: [[mentions]]
