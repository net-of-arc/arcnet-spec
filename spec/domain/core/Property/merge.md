---
"@id": merge
"@type": Property
role: meta
merge: immutable
datatype: enum
oneOf: [immutable, union, firstWriteWin, fillIfEmpty, lastWriteWin, earliest, latest, append]
bundle: core
---
# merge

One of the eight merge values (CORE §8.4): which contributed values this predicate admits, which one is in force, and whether a losing value is discarded or kept as an open alternative under `@conflicts`. The same value governs applying a patch, combining branches and resolving a conflict.
