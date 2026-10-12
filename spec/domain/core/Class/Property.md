---
"@id": Property
"@type": Class
bundle: core
---
# Property

A node declaring one predicate: its serialization position, how two contributions to it combine, what its values may be, and what it means.

**Identity**
The predicate's camelCase name, equal to the basename.

- defaultText:: [[description]]

## Requires One
- requiresOne:: [[role]]
- requiresOne:: [[merge]]
- requiresOne:: [[description]]
- requiresOne:: [[bundle]]

## Permits One
- permitsOne:: [[label]]
- permitsOne:: [[aligned]]
- permitsOne:: [[datatype]]

## Permits Many
- permitsMany:: [[oneOf]]
- permitsMany:: [[object]]
- permitsMany:: [[inverseOf]]

## Constraints
- constraint:: A predicate name MUST be camelCase, and MUST be registered exactly once across every bundle in use. Two bundles introducing the same name is a collision, not a merge.
- constraint:: `datatype` MUST be present when `role` is `meta` or `text`, and MUST be absent otherwise.
- constraint:: `oneOf` MUST be present, with at least one value, exactly when `datatype` is `enum`.
- constraint:: `object` and `inverseOf` MAY be present only when `role` is `edge` or `link`.
- constraint:: `label` MAY be present only when `role` is `text` or `link`.
- constraint:: For every predicate `f` named by `inverseOf`, every class that lists this predicate SHOULD also be documented under `f`'s `object`, and every `object` class of this predicate SHOULD list `f`; a backlink whose target cannot carry the forward edge can never be satisfied.
