---
"@id": category
"@type": Property
role: meta
merge: union
datatype: enum
oneOf: [object, process, schema, script, juncture, participation, description, history, structure, situation, reason, purpose]
bundle: core
---
# category

What kind of thing a subject *is*, as the leaf of John F. Sowa's top-level ontology. Each of the twelve leaves decodes to exactly one combination of the three upper levels, so only the leaf is stored and the upper levels are read from this table:

| Leaf (Level 4)  | Level 1     | Level 2  | Level 3    |
| --------------- | ----------- | -------- | ---------- |
| `object`        | independent | physical | continuant |
| `process`       | independent | physical | occurrent  |
| `schema`        | independent | abstract | continuant |
| `script`        | independent | abstract | occurrent  |
| `juncture`      | relative    | physical | continuant |
| `participation` | relative    | physical | occurrent  |
| `description`   | relative    | abstract | continuant |
| `history`       | relative    | abstract | occurrent  |
| `structure`     | mediating   | physical | continuant |
| `situation`     | mediating   | physical | occurrent  |
| `reason`        | mediating   | abstract | continuant |
| `purpose`       | mediating   | abstract | occurrent  |

`category` classifies subjects (`Entity`) only. The form of a work is `genre`; the subject a work is about is `about`.
