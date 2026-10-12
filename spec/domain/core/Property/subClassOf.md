---
"@id": subClassOf
"@type": Property
role: edge
merge: immutable
aligned: "rdfs:subClassOf"
bundle: core
---
# subClassOf

Declares this class a specialization of the target class (SCHEMA §3.5). Every instance of this class is also an instance of the target and of all its ancestors. The subclass inherits the target's identity rule, default text, membership blocks and constraints; it may only **add** predicates or **tighten** an inherited one from `permits…` to `requires…`, never redeclare, remove or loosen anything it inherits. Single inheritance: a class has at most one parent, and the chain of parents MUST NOT contain a cycle. Immutable, because changing a class's parent changes what every one of its instances must carry.

## Object
- object:: [[Class]]
