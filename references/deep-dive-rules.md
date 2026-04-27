# Deep Dive Diagram Type — Decision Tree

For each deep dive selected in Phase 6, classify it into ONE diagram type using this tree (evaluate top to bottom; first match wins):

```
1. Does this deep dive describe a TEMPORAL FLOW?
   ("when X happens, A then B then C in order")
   ("the request path through hop-by-hop")
   ("on cache miss, fall back to DB then ...")
   → SEQUENCE DIAGRAM

2. Does this deep dive only ADD A CALLOUT to the existing HLD?
   (a bottleneck warning, an edge-case marker, a numbered note)
   AND introduces NO new components?
   AND does NOT expose internals?
   → ANNOTATED HLD

3. Otherwise (introduces new components, new subsystems,
   exposes internals of an existing component, or shows
   a parallel architecture variant):
   → ZOOM-IN
```

## Examples

### Sequence diagram (type 1)

- "How does a write propagate through the cache and DB?"
- "Walk through the request lifecycle for a posted tweet."
- "What happens on cache miss?"
- "How does the leader election flow work?"

### Annotated HLD (type 2)

- "Where are the bottlenecks at 1M RPS?" → mark hot components on HLD
- "What are the cache stampede risks?" → red callouts on cache
- "Where does cost accumulate?" → cost annotations on relevant components
- "What if the DB goes down?" → red X marks + fallback annotations

### Zoom-in (type 3)

- "How does feed fan-out work?" → push vs pull, celebrity user handling — new internal components
- "How is video transcoding pipelined?" → upload → encode tiers → CDN — internal pipeline
- "Multi-region replication strategy" → topology not shown in HLD
- "Search indexing pipeline internals" — opens up the search component

## Tie-breakers

- A deep dive can mention timing AND introduce new components. The tree's first-match rule sends it to SEQUENCE; that is correct — temporal flow is the dominant feature.
- If unclear between annotated and zoom-in, count: does the diagram require any NEW shapes that don't appear on the HLD? If yes → zoom-in. If only labels/markers → annotated.
- Never use TWO types for the same deep dive. One topic, one diagram.
