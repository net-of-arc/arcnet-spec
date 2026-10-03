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
- requiresMany:: [[cites]]

## Constraints
- constraint:: A `Timeline` node is filed under `timeline/yearly/` or `timeline/monthly/` according to its `@id` shape, not flat in a type folder. It is an index of document production dates, never a change log — that is git.
