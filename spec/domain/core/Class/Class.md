---
"@id": Class
"@type": Class
bundle: core
---
# Class

A node declaring one type: how its instances are identified, which prose they render by default, which predicates they must and may carry, and what rules they must satisfy that no predicate declaration can express.

**Identity**
The class's UpperCamelCase name, equal to the basename and to its folder name.

- defaultText:: [[description]]

## Requires One
- requiresOne:: [[description]]
- requiresOne:: [[bundle]]

## Permits One
- permitsOne:: [[identityRule]]
- permitsOne:: [[defaultText]]
- permitsOne:: [[subClassOf]]

## Permits Many
- permitsMany:: [[requiresOne]]
- permitsMany:: [[requiresMany]]
- permitsMany:: [[permitsOne]]
- permitsMany:: [[permitsMany]]
- permitsMany:: [[constraint]]

## Constraints
- constraint:: A class name MUST be UpperCamelCase, and is fixed in three places at once: the `@type` value, this node's basename, and the type's folder name.
- constraint:: A class without `subClassOf` MUST carry `identityRule`. A class with `subClassOf` MUST NOT carry `identityRule`: identity is inherited from the root of its chain, never redeclared.
- constraint:: `defaultText`, where present, MUST name a `text`-role predicate that this class or one of its ancestors lists. A class MUST NOT carry `defaultText` when an ancestor already carries one.
- constraint:: The `subClassOf` chain MUST end at a class without `subClassOf`; it MUST NOT contain a cycle.
- constraint:: A predicate MUST be listed under at most one of the four membership blocks, counting the blocks of this class and of all its ancestors. The single exception is tightening: a subclass MAY list under `requiresOne` a predicate an ancestor lists under `permitsOne`, or under `requiresMany` one an ancestor lists under `permitsMany`.
- constraint:: A predicate whose `merge` is single-valued (`immutable`, `firstWriteWin`, `fillIfEmpty`, `lastWriteWin`, `earliest`, `latest`), or whose `role` is `text` with `merge: append`, MUST be listed under `requiresOne` or `permitsOne`. A predicate whose `merge` is `union` MUST be listed under `requiresMany` or `permitsMany`.
- constraint:: An instance of a class is an instance of every ancestor: every inherited membership block and every inherited constraint applies to it unchanged.
