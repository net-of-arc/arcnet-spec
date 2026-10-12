---
"@id": Source
"@type": Class
bundle: core
---
# Source

A node for one ingested document — the provenance origin other nodes derive from.

**Identity**
A citekey `<first-author-surname>-<publication-year>-<slug-keyword>`, lowercased ASCII, hyphen-separated — `rescorla-2026-tls13`. `anon` where the author is unknown. The same document always yields the same citekey. Two different documents that would yield the same citekey are distinguished by choosing a different slug keyword; a parenthetical or numeric suffix is never used, because which document received the plain citekey would then depend on ingestion order. The document's own title is the separate `title` predicate; the citekey never substitutes for it.

- defaultText:: [[abstract]]

## Requires One
- requiresOne:: [[title]]
- requiresOne:: [[published]]
- requiresOne:: [[abstract]]

## Requires Many
- requiresMany:: [[mentions]]

## Permits One
- permitsOne:: [[url]]
- permitsOne:: [[doi]]
- permitsOne:: [[genre]]

## Permits Many
- permitsMany:: [[author]]
- permitsMany:: [[tags]]
- permitsMany:: [[about]]
- permitsMany:: [[cites]]
- permitsMany:: [[supports]]
- permitsMany:: [[disputes]]
- permitsMany:: [[sameAs]]

## Constraints
- constraint:: Every `Entity` or `Resource` node carried by this document's patch, and every `Entity` this node reaches by `about`, MUST also appear under this node's `mentions`.
- constraint:: Every target of `mentions` MUST carry `mentionedIn` back to this node.
- constraint:: When this document is ingested, every existing `Reference` to the same work — equal `doi`, else equal `url`, else equal `title` — MUST be linked to this node by `sameAs`, and so MUST every `Reference` to it created later. The `Reference` is kept, never renamed or merged: the edges that cite it keep resolving.
