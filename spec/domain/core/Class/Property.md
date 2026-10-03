---
"@id": Property
"@type": Class
bundle: core
---
# Property

A node declaring one predicate: its serialization position, how two contributions to it combine, what its values may be, and what it means.

**Identity**
The predicate's camelCase name, equal to the basename.

## Requires One
- requiresOne:: [[role]]
- requiresOne:: [[merge]]
- requiresOne:: [[description]]

## Permits One
- permitsOne:: [[label]]
- permitsOne:: [[aligned]]
- permitsOne:: [[datatype]]
- permitsOne:: [[bundle]]

## Permits Any
- permitsMany:: [[permits]]
- permitsMany:: [[object]]

## Constraints
- constraint:: A predicate name MUST be camelCase, and MUST be registered exactly once across every bundle in use. Two bundles introducing the same name is a collision, not a merge.
