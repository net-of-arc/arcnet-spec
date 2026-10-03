---
"@id": Resource
"@type": Class
bundle: core
---
# Resource

A fragment of an ingested document's content that is relevant to the graph but does not warrant its own dedicated type.

**Identity**
A concise title for the content, phrased so a second document carrying the same content yields the same title.

## Requires One
- requiresOne:: [[text]]

## Requires Many
- requiresMany:: [[tags]]
- requiresMany:: [[mentionedIn]]

## Permits Any
- permitsMany:: [[about]]
