---
"@id": oneOf
"@type": Property
role: meta
merge: union
aligned: "owl:oneOf"
datatype: string
bundle: core
---
# oneOf

The legal values of a predicate whose `datatype` is `enum`, as one front-matter array. The set is closed unless the predicate's description says otherwise. Named apart from `permitsOne`/`permitsMany` on purpose: those say which predicates a class allows, this says which values a predicate allows.
