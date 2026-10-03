---
"@id": Source
"@type": Class
bundle: core
---
# Source

A node for one ingested document — the provenance origin other nodes derive from.

**Identity**
A citekey `<first-author-surname>-<publication-year>-<slug-keyword>`, lowercased ASCII, hyphen-separated — `rescorla-2026-tls13`. `anon` where the author is unknown. The same document always yields the same citekey. The document's own title is the separate `title` predicate; the citekey never substitutes for it.

## Requires One
- requiresOne:: [[title]]
- requiresOne:: [[published]]
- requiresOne:: [[abstract]]

## Requires Many
- requiresMany:: [[mentions]]

## Permits One
- permitsOne:: [[url]]
- permitsOne:: [[doi]]

## Permits Any
- permitsMany:: [[author]]
- permitsMany:: [[cites]]
- permitsMany:: [[tags]]
- permitsMany:: [[proposes]]
- permitsMany:: [[raises]]
- permitsMany:: [[generatedThought]]
- permitsMany:: [[analyzes]]
- permitsMany:: [[about]]
- permitsMany:: [[genre]]

## Constraints
- constraint:: Every entity reached by a domain predicate asserted from this document MUST also appear under this node's `mentions`.
