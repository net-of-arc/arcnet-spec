---
"@id": category
"@type": Property
role: meta
merge: immutable
datatype: enum
bundle: core
---
# category

Records John F. Sowa's top-level category:
- Level 1: independent · relative · mediating
- Level 2: physical · abstract
- Level 3: continuant · occurrent
- Level 4 (Leaf): object · process · schema · script · juncture · participation · description · history · structure · situation · reason · purpose

The `category` predicate MUST contain the four decoded words. Allowed combinations, following John F. Sowa's taxonomy:
- `[independent, physical, continuant, object]`
- `[independent, physical, occurrent, process]`
- `[independent, abstract, continuant, schema]`
- `[independent, abstract, occurrent, script]`
- `[relative, physical, continuant, juncture]`
- `[relative, physical, occurrent, participation]`
- `[relative, abstract, continuant, description]`
- `[relative, abstract, occurrent, history]`
- `[mediating, physical, continuant, structure]`
- `[mediating, physical, occurrent, situation]`
- `[mediating, abstract, continuant, reason]`
- `[mediating, abstract, occurrent, purpose]`

## Permits
- permits:: [independent, physical, continuant, object]
- permits:: [independent, physical, occurrent, process]
- permits:: [independent, abstract, continuant, schema]
- permits:: [independent, abstract, occurrent, script]
- permits:: [relative, physical, continuant, juncture]
- permits:: [relative, physical, occurrent, participation]
- permits:: [relative, abstract, continuant, description]
- permits:: [relative, abstract, occurrent, history]
- permits:: [mediating, physical, continuant, structure]
- permits:: [mediating, physical, occurrent, situation]
- permits:: [mediating, abstract, continuant, reason]
- permits:: [mediating, abstract, occurrent, purpose]
