---
"@id": created
"@type": Property
role: meta
merge: earliest
aligned: "dcterms:created"
datatype: datetime
bundle: core
---
# created

The time the node first entered the graph — the ingest time of the contribution that created it, not the production date of any document (that is `published`). Universal and mandatory on every graph node (SCHEMA §3.4). When two branches create the same node independently, the earlier time is kept.
