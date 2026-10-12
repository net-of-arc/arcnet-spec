---
"@id": Entity
"@type": Class
bundle: core
---
# Entity

A node for a subject occurring in sources, typed by Sowa category (`category`).

**Identity**
The subject's preferred label (`skos:prefLabel`), human-readable and Title-Cased with spaces. Every other name the sources used is an `aliases` value. A colliding label — including one that collides, case-insensitively, with a schema node such as `Class` or `Object` — is disambiguated by a parenthetical qualifier: `Handshake Protocol (TLS)`, `Class (Programming)`.

- defaultText:: [[text]]

## Requires One
- requiresOne:: [[category]]
- requiresOne:: [[text]]

## Requires Many
- requiresMany:: [[mentionedIn]]

## Permits Many
- permitsMany:: [[aliases]]
- permitsMany:: [[tags]]
- permitsMany:: [[broader]]
- permitsMany:: [[narrower]]
- permitsMany:: [[isPartOf]]
- permitsMany:: [[hasPart]]
- permitsMany:: [[requires]]
- permitsMany:: [[replaces]]
- permitsMany:: [[isReplacedBy]]
- permitsMany:: [[conformsTo]]
- permitsMany:: [[related]]
