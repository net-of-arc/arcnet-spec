---
"@id": about
"@type": Property
role: edge
merge: union
aligned: "schema:about"
bundle: core
---
# about

**Primary subject.** `X about:: Y` asserts that the work or fragment `X` is principally about the entity `Y`. Narrower than `mentions`, which records every entity a document touches. What *kind* of thing `Y` is (a technique, a framework, a field) is a property of `Y` (`category`, `tags`), never of `X`. *e.g.* `rescorla-2026-tls13` → `about::` `Transport Layer Security`.

## Object
- object:: [[Entity]]
