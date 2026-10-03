---
"@id": datatype
"@type": Property
role: meta
merge: immutable
datatype: enum
bundle: core
---
# datatype

The literal type a `meta`- or `text`-role predicate's values take. `enum` means the legal values are listed by `permits`.

## Permits
- permits:: string
- permits:: date
- permits:: number
- permits:: boolean
- permits:: enum
