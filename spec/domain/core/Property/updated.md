---
"@id": updated
"@type": Property
role: meta
merge: latest
aligned: "dcterms:modified"
datatype: datetime
bundle: core
---
# updated

The time of the last accepted contribution that changed any other predicate of the node. A contribution that changes nothing leaves it untouched, so re-applying a patch stays idempotent. Universal and mandatory on every graph node (SCHEMA §3.4); never earlier than `created`.
