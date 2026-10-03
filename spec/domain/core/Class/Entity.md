---
"@id": Entity
"@type": Class
bundle: core
---
# Entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.2).

**Identity**
The subject's preferred label (`skos:prefLabel`), human-readable and Title-Cased with spaces. Every other name the sources used is an `aliases` value. A colliding label is disambiguated by a parenthetical qualifier — `Handshake Protocol (TLS)`.

## Requires One
- requiresOne:: [[category]]
- requiresOne:: [[text]]

## Requires Many
- requiresMany:: [[mentionedIn]]

## Permits Any
- permitsMany:: [[aliases]]
- permitsMany:: [[tags]]
- permitsMany:: [[concerns]]
- permitsMany:: [[broader]]
- permitsMany:: [[narrower]]
- permitsMany:: [[isPartOf]]
- permitsMany:: [[hasPart]]
- permitsMany:: [[requires]]
- permitsMany:: [[replaces]]
- permitsMany:: [[isReplacedBy]]
- permitsMany:: [[conformsTo]]
- permitsMany:: [[related]]
- permitsMany:: [[referencedBy]]
- permitsMany:: [[secures]]
- permitsMany:: [[verifies]]

## Constraints
- constraint:: `category` MUST carry four values decoding one legal Sowa combination — one from each of the four levels listed by the `category` predicate's own `permits` set. Combinations outside that list are not conforming.
