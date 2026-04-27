# System Design Element Templates

JSON snippets for each component used in HelloInterview-style HLDs. All templates use the JSON schema documented in `json-schema.md`. Colors come from `color-palette.md`.

For every template:
- `id`, `seed`, `version`, `versionNonce`, `updated` are placeholders — generate fresh integers for each instance.
- `groupIds` ties multi-shape components (e.g., DB + schema text) together.
- Coordinates (`x`, `y`) are placeholders; the agent positions components per the layout convention.

## Layout Convention

- Left-to-right horizontal flow: Client → Edge (LB / API GW / CDN) → App tier → Data tier.
- Vertical stacking within a column = replicas, shards, or peers of the same tier.
- For globally distributed designs: wrap region-local components in a Region Cluster Container.
- Standard horizontal spacing between tiers: 220px. Standard vertical spacing between peers: 30px.
- DB schema column blocks sit 20px to the right of the DB shape, top-aligned.
- Cost annotations sit 10px below the component, in muted gray (`#999`), font size 12.

## Components

All "box + label" components use upstream's pattern from `element-templates.md`:
1. A `rectangle` element with `boundElements: [{"id": "<text-id>", "type": "text"}]`.
2. A `text` element with `containerId: "<rectangle-id>"`, `textAlign: "center"`, `verticalAlign: "middle"`.

Stroke and background colors come from `color-palette.md` based on each component's semantic tier (edge, app, data, async, etc.).

### Client (web / mobile / CLI)

The leftmost element. Standard rectangle 120×60 with bound text. Label is caller-chosen: `Client`, `Mobile App`, `Web Browser`, `CLI`. Use the "client / external" color from the palette. Optional: a small ellipse marker dot in the top-left if depicting a user-actor specifically.

### Load Balancer

Rectangle 140×60 with bound text label `Load Balancer` (or `LB`, `ALB`, `NLB`, `HAProxy`). Use the "edge tier" color from the palette. Optional VIP marker: a small `ellipse` (12×12) positioned 6px above the rectangle's top edge, centered horizontally, with the same stroke color and `fillStyle: "solid"`. Add the ellipse's id into the rectangle's `boundElements` array AND share `groupIds` so the VIP moves with the LB.

### API Gateway

Rectangle 140×60 with bound text label `API Gateway` (or product names like `Kong`, `Tyk`, `AWS API Gateway`). Same color tier as Load Balancer (edge). Place to the right of the LB if both exist; one or the other is more common.

### App Server / Service

Two variants:

- **Single instance**: rectangle 160×70 with bound text. Label is the service name (e.g., `User Service`, `Order Service`, `Auth Service`). Use the "app tier" color.
- **Stacked instances** (depicting horizontal scale): three `rectangle` elements with offsets `(+0,+0)`, `(+6,+6)`, `(+12,+12)` — the bottom-most drawn first so the top one sits on top. Only the topmost rectangle has a `boundElements` entry pointing to the label text. All three rectangles AND the label share the same `groupIds: ["app-<name>-grp"]`. Use the same fill/stroke color for all three; only the topmost one carries the label.

### Database (with schema columns)

A cylinder-style database is modeled as a rectangle with full roundness (`roundness: { "type": 3 }`), 160×80, with a centered bound text label, PLUS a separate free-floating `text` element 20px to its right that lists schema columns. The DB rectangle, the DB label, and the schema text all share the same `groupIds`.

```json
[
  {
    "type": "rectangle",
    "id": "db-1",
    "x": 600, "y": 200,
    "width": 160, "height": 80,
    "strokeColor": "#1971c2",
    "backgroundColor": "#a5d8ff",
    "fillStyle": "solid",
    "strokeWidth": 2,
    "strokeStyle": "solid",
    "roughness": 0,
    "opacity": 100,
    "angle": 0,
    "seed": 101010,
    "version": 1,
    "versionNonce": 202020,
    "isDeleted": false,
    "groupIds": ["db-users-grp"],
    "boundElements": [{"id": "db-1-label", "type": "text"}],
    "updated": 1,
    "link": null,
    "locked": false,
    "roundness": {"type": 3}
  },
  {
    "type": "text",
    "id": "db-1-label",
    "x": 620, "y": 230,
    "width": 120, "height": 25,
    "text": "users (Postgres)",
    "originalText": "users (Postgres)",
    "fontSize": 16,
    "fontFamily": 3,
    "textAlign": "center",
    "verticalAlign": "middle",
    "strokeColor": "#1971c2",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 1,
    "strokeStyle": "solid",
    "roughness": 0,
    "opacity": 100,
    "angle": 0,
    "seed": 303030,
    "version": 1,
    "versionNonce": 404040,
    "isDeleted": false,
    "groupIds": ["db-users-grp"],
    "boundElements": null,
    "updated": 1,
    "link": null,
    "locked": false,
    "containerId": "db-1",
    "lineHeight": 1.25
  },
  {
    "type": "text",
    "id": "db-1-schema",
    "x": 780, "y": 200,
    "width": 180, "height": 100,
    "text": "users\n─────\nid: bigint\nemail: text\ncreated_at: ts",
    "originalText": "users\n─────\nid: bigint\nemail: text\ncreated_at: ts",
    "fontSize": 14,
    "fontFamily": 3,
    "textAlign": "left",
    "verticalAlign": "top",
    "strokeColor": "#1971c2",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 1,
    "strokeStyle": "solid",
    "roughness": 0,
    "opacity": 100,
    "angle": 0,
    "seed": 505050,
    "version": 1,
    "versionNonce": 606060,
    "isDeleted": false,
    "groupIds": ["db-users-grp"],
    "boundElements": null,
    "updated": 1,
    "link": null,
    "locked": false,
    "containerId": null,
    "lineHeight": 1.25
  }
]
```

Note the schema text uses `\n` line breaks and a horizontal rule (`─────`) under the table name. Replace the `users / id: bigint / ...` content with the actual schema for the design. Color values shown (`#1971c2`, `#a5d8ff`) are illustrative — pull the real "data tier" colors from `color-palette.md`.

### Cache (Redis / Memcached style)

Rectangle 140×60 with rounded corners using `roundness: { "type": 3, "value": 25 }` (more rounded than a normal rectangle but not a full cylinder). Bound text label `Cache`, `Redis`, or `Memcached`. Use the "cache tier" color from the palette (typically a warm color to distinguish from primary DBs).

### Message Queue (Kafka / SQS style)

A long rectangle 200×50 (the "queue body") with 3 internal vertical lines suggesting partitioned messages. The vertical lines are separate `line` elements positioned at `x = body.x + 50, 100, 150` (evenly dividing the body), `y = body.y + 5` to `y = body.y + 45`. The body rectangle has a `boundElements` entry for its label text. The body, the 3 lines, and the label all share `groupIds: ["mq-<name>-grp"]`.

Label text sits 6px below the body (free-floating text, not bound) with format `Queue: <name>` or `Topic: <name>`. Use the "async / messaging" color from the palette.

### Object Storage / CDN

Rectangle 140×60 with bound text label `S3`, `Object Storage`, `GCS`, or `CDN`. Use the "storage" color.

To distinguish a CDN from plain object storage: add a small `ellipse` (16×16) in the top-left corner of the rectangle as a "globe" marker. Stroke matches the rectangle, `backgroundColor: "transparent"`. Add the ellipse to the rectangle's `boundElements` and share `groupIds`.

### Worker / Background Job

Rectangle 140×60 with bound text label `Worker: <name>` (e.g., `Worker: image-resize`, `Worker: email-sender`). Add a small "gear" marker in the top-right corner — model as a tiny `ellipse` (10×10) or a 5-pointed `polygon` (if available; otherwise an ellipse is fine). Use the "async / messaging" color tier.

### Search Index

Rectangle 140×60 with bound text label `Elasticsearch`, `OpenSearch`, or `Search Index`. Use a distinct "search" color from the palette (commonly green or teal in HelloInterview diagrams).

### Pub/Sub Broker

Rectangle 140×60 with bound text label `Pub/Sub`, `Kafka`, or `SNS`. To suggest fan-out: include two short `arrow` elements pre-drawn out the right side of the broker, both bound on the start side via `startBinding: { "elementId": "<broker-id>", "focus": 0, "gap": 2 }` with `endBinding: null` and a 40px length. Place the arrows at `y = broker.y + 15` and `y = broker.y + 45`. The broker, label, and both arrows share `groupIds: ["pubsub-<name>-grp"]`.

### Region Cluster Container

A dashed-border rectangle that visually wraps several components inside a single geographic region. The container has NO fill (`backgroundColor: "transparent"`) and a top-left label inside it.

```json
[
  {
    "type": "rectangle",
    "id": "region-1",
    "x": 100, "y": 150,
    "width": 520, "height": 320,
    "strokeColor": "#666",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 2,
    "strokeStyle": "dashed",
    "roughness": 0,
    "opacity": 100,
    "angle": 0,
    "seed": 707070,
    "version": 1,
    "versionNonce": 808080,
    "isDeleted": false,
    "groupIds": ["region-tokyo-grp"],
    "boundElements": null,
    "updated": 1,
    "link": null,
    "locked": false,
    "roundness": {"type": 3}
  },
  {
    "type": "text",
    "id": "region-1-label",
    "x": 120, "y": 160,
    "width": 280, "height": 20,
    "text": "Region: ap-northeast-1 (Tokyo)",
    "originalText": "Region: ap-northeast-1 (Tokyo)",
    "fontSize": 14,
    "fontFamily": 3,
    "textAlign": "left",
    "verticalAlign": "top",
    "strokeColor": "#666",
    "backgroundColor": "transparent",
    "fillStyle": "solid",
    "strokeWidth": 1,
    "strokeStyle": "solid",
    "roughness": 0,
    "opacity": 100,
    "angle": 0,
    "seed": 909090,
    "version": 1,
    "versionNonce": 111213,
    "isDeleted": false,
    "groupIds": ["region-tokyo-grp"],
    "boundElements": null,
    "updated": 1,
    "link": null,
    "locked": false,
    "containerId": null,
    "lineHeight": 1.25
  }
]
```

Components placed inside the region are NOT added to the container's `groupIds` — they're positioned visually within the rectangle's bounds with 20px padding. Resize the container's `width` and `height` to fit N inner components (the example above sizes 520×320 to wrap roughly 2 columns × 3 rows of standard 140×60 boxes with 20px gaps).

Example label values:
- `Region: ap-northeast-1 (Tokyo)`
- `Region: ap-northeast-2 (Seoul)`
- `Region: us-east-1 (Virginia)`
- `Region: eu-west-1 (Ireland)`

### Cost Annotation

A single free-floating `text` element placed 10px below a parent component's bounding box, in muted gray.

```json
{
  "type": "text",
  "id": "cost-anno-1",
  "x": 600, "y": 290,
  "width": 200, "height": 18,
  "text": "$0.023/GB/mo · 50TB",
  "originalText": "$0.023/GB/mo · 50TB",
  "fontSize": 12,
  "fontFamily": 3,
  "textAlign": "left",
  "verticalAlign": "top",
  "strokeColor": "#999",
  "backgroundColor": "transparent",
  "fillStyle": "solid",
  "strokeWidth": 1,
  "strokeStyle": "solid",
  "roughness": 0,
  "opacity": 100,
  "angle": 0,
  "seed": 141516,
  "version": 1,
  "versionNonce": 171819,
  "isDeleted": false,
  "groupIds": [],
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false,
  "containerId": null,
  "lineHeight": 1.25
}
```

Format strings:
- Storage: `$0.023/GB/mo · 50TB`
- Requests: `$0.40/M req · 200M req/mo`
- Instance: `c5.xlarge · 12 instances · $1,200/mo`
- Bandwidth: `$0.09/GB egress · 8TB/mo`

Used when Phase 1 capacity estimation flagged cost as a design driver. Skip cost annotations on diagrams where cost is not a key concern — they add visual noise.

## Arrow Styles

| Style | Excalidraw config | Meaning |
|---|---|---|
| Solid | `strokeStyle: "solid"`, `endArrowhead: "arrow"` | Synchronous request/response |
| Dashed | `strokeStyle: "dashed"`, `endArrowhead: "arrow"` | Asynchronous / fire-and-forget |
| Double-line | Two parallel solid arrows (two `arrow` elements offset 4px on the perpendicular axis) | Replication / data sync between peers |

All arrows MUST bind to source and target via `startBinding` / `endBinding` (referencing the target element's `id` with `focus: 0, gap: 2`) so they reconnect when components move during the render-and-fix loop. Arrow color typically matches the source element's stroke from the palette; use the "async" color (often muted) for dashed async arrows regardless of source.

For double-line replication arrows, both arrow elements share the same `groupIds` so they move together.

## Sequence Diagram Primitives

For Phase 6 type D (sequence diagrams):

- **Lifeline header**: rectangle 100×40 at the top of the canvas, labeled with actor/component name (bound text, centered). Place lifeline headers horizontally spaced 180px apart.
- **Lifeline**: vertical `line` element below each header. `strokeStyle: "dashed"`, `strokeWidth: 1`, length determined by the number of messages (typically 30px per message slot). `points: [[0, 0], [0, <length>]]`. Share `groupIds` with the header so the pair moves together.
- **Message arrow**: horizontal `arrow` between two lifelines, bound on both ends to the respective lifeline `line` elements (or to invisible activation bars — see below).
  - Solid arrow with `endArrowhead: "arrow"` = synchronous request
  - Dashed arrow with `endArrowhead: "arrow"` = asynchronous
  - Dashed arrow with `endArrowhead: "triangle"` (open arrowhead) = response / return value
- **Activation bar**: thin `rectangle` 8px wide, overlaid on the lifeline at the x-coordinate of the lifeline. Height spans the time range during which the component is active. `fillStyle: "solid"`, color from palette `activation` (typically a light tinted color matching the component's tier). Share `groupIds` with the lifeline.
- **Time flows top to bottom.** Earliest message at the top; vertical position = time order. Standard vertical spacing between consecutive messages: 40px.

Label each message arrow with a free-floating `text` element 4px above the arrow midpoint, fontSize 12, format `<verb> <noun>` (e.g., `POST /orders`, `publish OrderCreated`, `200 OK`).
