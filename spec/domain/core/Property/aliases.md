---
"@id": aliases
"@type": Property
role: meta
merge: union
aligned: "skos:altLabel"
datatype: string
bundle: core
---
# aliases

Alternative identities of the subject — other names the sources use for it, or its original name where that contains characters an `@id` may not (CORE §5). An alias resolves to exactly one node: compared as `@id`s are compared, it MUST NOT equal any other node's `@id` or any other node's alias. The check runs against the nodes themselves, exactly as `@id` uniqueness does; no separate alias table exists.

The name is plural, against the singular naming of most predicates, on purpose: `aliases` is the front-matter key Obsidian reads natively, so a graph opened in Obsidian gets alias search, link autocompletion and unlinked-mention detection without any plugin or tooling. Renaming it would give up that interoperability for no gain in meaning.
