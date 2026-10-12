---
"@id": defaultText
"@type": Property
role: edge
merge: firstWriteWin
aligned: "arc:defaultText"
bundle: core
---
# defaultText

Names the `text`-role predicate that a class's instances render as their unlabelled body prose, directly under the H1 title (CORE §4). Stated on the class, because which prose is a node's default is a property of the class, not of the predicate: the same predicate is the default on one class and a labelled block on another. A class without `defaultText` has no unlabelled prose; every `text`-role predicate it lists renders under its label.

## Object
- object:: [[Property]]
