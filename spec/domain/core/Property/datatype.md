---
"@id": datatype
"@type": Property
role: meta
merge: immutable
datatype: enum
oneOf: [string, date, datetime, number, boolean, enum]
bundle: core
---
# datatype

The literal type a `meta`- or `text`-role predicate's values take. `date` is an ISO-8601 calendar date — `YYYY-MM-DD`, or the reduced precisions `YYYY-MM` and `YYYY`. `datetime` is an ISO-8601 instant in UTC, `YYYY-MM-DDThh:mm:ssZ`, so that values from different producers compare correctly. `enum` means the legal values are listed by `oneOf`.
