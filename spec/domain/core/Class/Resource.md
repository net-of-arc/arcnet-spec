---
"@id": Resource
"@type": Class
bundle: core
---
# Resource

A fragment of an ingested document's content that is relevant to the graph but does not warrant its own dedicated class/type.

**Identity**
A concise title for the content, phrased so a second document carrying the same content yields the same title.

- defaultText:: [[text]]

## Requires One
- requiresOne:: [[text]]

## Requires Many
- requiresMany:: [[tags]]
- requiresMany:: [[mentionedIn]]

## Permits Many
- permitsMany:: [[about]]
