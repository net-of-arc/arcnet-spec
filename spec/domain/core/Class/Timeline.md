---
"@id": Timeline
"@type": Class
bundle: core
---
# Timeline

A production-date index of ingested documents.

**Identity**
The period code — `YYYY` for a yearly bucket, `YYYY-MM` for a monthly one.

## Requires Many
- requiresMany:: [[spanContains]]

## Constraints
- constraint:: A `Timeline` node is filed under `timeline/yearly/` or `timeline/monthly/` according to its `@id` shape, not flat in a type folder. It is an index of document production dates, never a change log — that is git.
- constraint:: Every `spanContains` target's `published` date MUST fall within this node's period, and every `Source` MUST be contained by every bucket whose period contains its `published` date at a precision that date supports.
- constraint:: A `Timeline` node is derived data: it MUST be reproducible from the `Source` nodes alone, and is regenerated, never textually merged, when branches are combined (CORE §9.4).
