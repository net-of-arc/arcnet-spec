---
"@id": object
"@type": Property
role: link
merge: union
label: "Object"
aligned: "rdfs:range"
bundle: core
---
# object

The class or classes a predicate's values are **expected** to be instances of — documentation and producer guidance, never a closed set. A producer SHOULD point the predicate at an instance of a listed class or of one of its subclasses (`subClassOf`); a consumer MUST NOT reject an edge whose target is of another class, and a checker MAY report it as a warning only. This matches `rdfs:range`, which licenses an inference about the target, not a restriction on it, and keeps every predicate open to classes that later bundles introduce. Only the range is documented: the domain half is already stated, inverted, by every class that requires or permits the predicate (SCHEMA §2.1).

## Object
- object:: [[Class]]
