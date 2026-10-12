---
"@id": Reference
"@type": Class
bundle: core
---
# Reference

A node for an external work the graph points to but has not ingested.

**Identity**
The work's own name, as its authors established it, with characters CORE §5 forbids removed; the verbatim name is the `title`.

## Requires One
- requiresOne:: [[title]]

## Permits One
- permitsOne:: [[url]]
- permitsOne:: [[published]]
- permitsOne:: [[doi]]
- permitsOne:: [[genre]]

## Permits Many
- permitsMany:: [[author]]
- permitsMany:: [[isCitedBy]]
- permitsMany:: [[about]]
- permitsMany:: [[sameAs]]
