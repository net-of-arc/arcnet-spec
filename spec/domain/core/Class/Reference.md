---
"@id": Reference
"@type": Class
bundle: core
---
# Reference

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research.

**Identity**
The work's own name, as its authors established it.

## Requires One
- requiresOne:: [[title]]

## Permits One
- permitsOne:: [[url]]
- permitsOne:: [[published]]
- permitsOne:: [[doi]]

## Permits Any
- permitsMany:: [[author]]
- permitsMany:: [[isCitedBy]]
- permitsMany:: [[about]]
- permitsMany:: [[genre]]
