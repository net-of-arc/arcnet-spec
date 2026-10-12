---
"@id": inverseOf
"@type": Property
role: link
merge: union
label: "Inverse Of"
aligned: "owl:inverseOf"
bundle: core
---
# inverseOf

Declares this predicate a backlink of the target predicate. If node X carries `b:: [[Y]]` and `b` declares `inverseOf` one or more predicates, Y MUST carry at least one of them pointing back to X. The backlink never exists without its forward edge; the forward edge MAY exist without the backlink unless a class constraint says otherwise.

## Object
- object:: [[Property]]
