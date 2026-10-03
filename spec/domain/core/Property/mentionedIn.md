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

The inverse of `mentions` when carried by an `Entity`; more generally, any derived node's backlink to the source it was drawn from (`Resource` uses it this way too) — recorded under the node's own `## Mentioned In` block.

## Object
- object:: [[Source]]
