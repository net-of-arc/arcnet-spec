---
"@id": label
"@type": Property
role: meta
merge: firstWriteWin
datatype: string
bundle: core
---
# label

Human-readable title shown as a `link`-role predicate's `## ` heading, or as a `text`-role predicate's bold label or `## ` heading wherever that predicate is not its class's `defaultText` (CORE §4). Defaults to the predicate name split at its camelCase boundaries and capitalized — `mentionedIn` → `Mentioned In`.
