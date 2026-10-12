---
"@id": bundle
"@type": Property
role: meta
merge: immutable
datatype: string
bundle: core
---
# bundle

The adoption unit this schema node belongs to, equal to the name of the `domain/<bundle>/` folder that holds it. A graph adopts `core` plus zero or more domain bundles; a node is never introduced by two bundles (SCHEMA §6). Required, because once a graph flattens its adopted bundles into `_schema/` this predicate is the only record of where a node came from.
