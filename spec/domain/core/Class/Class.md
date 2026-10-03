---
"@id": Class
"@type": Class
bundle: core
---
# Class

A node declaring one type: how its instances are identified, which predicates they must and may carry, and what rules they must satisfy that no predicate declaration can express.

**Identity**
The class's UpperCamelCase name, equal to the basename and to its folder name.

## Requires One
- requiresOne:: [[description]]
- requiresOne:: [[identityRule]]

## Permits One
- permitsOne:: [[bundle]]

## Permits Any
- permitsMany:: [[requiresOne]]
- permitsMany:: [[requiresMany]]
- permitsMany:: [[permitsOne]]
- permitsMany:: [[permitsMany]]
- permitsMany:: [[constraint]]

## Constraints
- constraint:: A class name MUST be UpperCamelCase, and is fixed in three places at once: the `@type` value, this node's basename, and the type's folder name.
