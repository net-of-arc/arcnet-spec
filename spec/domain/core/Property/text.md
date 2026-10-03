---
"@id": text
"@type": Property
role: text
merge: append
aligned: "schema:text"
datatype: string
bundle: core
---
# text

Generic prose predicate. Each contribution appends to the existing prose rather than overwriting it, since separate documents may each add relevant text about the same subject over time. A type MAY instead declare its own, more specific text predicate (e.g. `abstract`, §10.2) when a precise name aids reading and a single, first-fixed value is wanted instead.
