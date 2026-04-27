# HelloInterview Delivery Framework — 6 Phases

The skill's workflow runs these phases in order. Each phase has a model-effort budget, required artifacts, and skip criteria. Source: https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery

## Phase 1 — Requirements (~5 min effort)

**Output:** prioritized lists in `design.md` under `## Requirements`.

**Functional requirements:** "Users should be able to..." statements identifying core features. List 4-7. Order by importance.

**Non-functional requirements:** system qualities. Pick the 3-5 that actually constrain the design from this menu: availability, scalability, latency, durability, security, fault tolerance, consistency, compliance.

**Capacity estimation:** include ONLY when numbers will materially change downstream design choices. Examples that warrant estimation:
- RPS that determines sharded vs single-node SQL
- Storage volume that determines hot/warm/cold tier split
- Bandwidth that determines whether CDN is required
- Concurrent connection count that determines push vs pull

When estimation surfaces **cost** as a design driver (e.g., archive at PB scale), record per-component cost rates (`$/GB/mo`, `$/M req`, instance class hourly) so they can be rendered as cost annotations on the HLD or annotated deep-dive diagrams.

Skip estimation if the answer is "any reasonable architecture handles this load."

## Phase 2 — Core Entities (~2 min effort)

**Output:** bulleted list in `design.md` under `## Core Entities`.

The nouns satisfying the functional requirements (User, Tweet, Follow, Feed, etc.). Names only — no schemas. Schemas appear adjacent to database elements in Phase 5.

## Phase 3 — API / System Interface (~5 min effort)

**Output:** REST endpoints in `design.md` under `## API`.

- HTTP verb + path + request body + response.
- Plural resource names: `POST /tweets`, `GET /feed`, not `POST /createTweet`.
- Authenticate via headers, never request bodies. Note this in the section text.
- Cover the functional requirements end-to-end. Don't list trivial endpoints (login/logout) unless they're the focus.

## Phase 4 — Data Flow (~5 min effort, OPTIONAL)

**Output:** numbered steps in `design.md` under `## Data Flow` (omit section entirely if skipped).

**Include only if** the system is data-processing (analytics pipeline, ML inference, ETL, video transcoding, search indexing). For request/response systems (e.g., URL shortener, Twitter feed UI), skip.

Format:

```
1. Client uploads raw <input> to <component>.
2. <component> writes to <queue>.
3. Worker consumes queue, transforms data via <step>.
4. Result written to <storage>.
5. Downstream <consumer> reads from <storage>.
```

## Phase 5 — High-Level Design (~10-15 min effort)

**Output:** `hld.excalidraw` + `hld.png` + `design.md` section `## High-Level Design`.

- Build the diagram using element templates from `element-templates.md`.
- Layout: left-to-right is client → edge → app tier → data tier. Vertical stacking represents replicas/peers within the same tier.
- DB elements MUST display schema columns in an adjacent text block to the right side.
- For globally distributed systems, wrap region-local components in a region cluster container (dashed border + region label like `Region: ap-northeast-1 (Tokyo)`).
- When cost is a design driver (recorded in Phase 1), attach cost annotations below relevant components.
- Run the render-and-fix loop (max 3 iterations) — see SKILL.md.

The `design.md` section narrates the diagram in prose: what each component does, why each tier exists, the request path through the system.

## Phase 6 — Deep Dives (~10 min effort)

**Output:** 2-3 additional diagrams + `design.md` section `## Deep Dives`.

- Pick deep dives from the Phase 1 non-functional requirements. Each non-functional requirement that materially affects architecture should be addressed.
- Maximum 3 deep dives (hard cap).
- For each, classify the diagram type using `deep-dive-rules.md`.
- Diagram filenames per type:
  - Annotated HLD: `hld-annotated-<slug>.excalidraw` + `.png`
  - Zoom-in: `deep-dive-<slug>.excalidraw` + `.png`
  - Sequence: `sequence-<slug>.excalidraw` + `.png`
- Run the same render-and-fix loop per diagram.
- Each deep dive gets a subsection `### <Topic>` in `design.md` with prose analysis and a reference to the diagram file.
