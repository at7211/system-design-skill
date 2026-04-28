# System Design Skill

A Claude Code skill that takes a system-design prompt and runs the [HelloInterview Delivery framework](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery) end-to-end, producing a structured `design.md` plus Excalidraw diagrams (1 HLD + 2–3 deep dives) with a Playwright render-and-fix validation loop.

Forked from [`coleam00/excalidraw-diagram-skill`](https://github.com/coleam00/excalidraw-diagram-skill) — the upstream skill is a generic diagram generator; this fork specializes it for system design.

## What's different from upstream

- **Domain-specific workflow.** Runs the HelloInterview 6-phase framework: Requirements → Core Entities → API → (Data Flow) → High-Level Design → Deep Dives. Phase 4 (Data Flow) is auto-skipped for request/response systems.
- **System-design element templates.** Load Balancer, API Gateway, App Server (stacked instances), Database with adjacent schema columns, Cache, Message Queue, Object Storage / CDN, Worker, Search Index, Pub/Sub Broker, Region Cluster Container (for global designs), Cost Annotation (when capacity surfaces cost as a design driver).
- **Three deep-dive diagram types with a decision tree.** SEQUENCE for temporal flows, ANNOTATED HLD for callouts on existing components, ZOOM-IN for new subsystems.
- **Worked Twitter-feed example** in `examples/twitter-feed/` covers all three deep-dive types and serves as a few-shot reference.
- **Excalifont by default** to match Excalidraw's hand-drawn aesthetic (upstream defaulted to Cascadia monospace).

Inherited from upstream: the Playwright render-and-fix loop, the color palette customization model, the Excalidraw JSON schema reference.

## Sample output

High-level design (Phase 5):

![HLD](examples/twitter-feed/hld.png)

Sequence-type deep dive (temporal flow):

![Sequence diagram](examples/twitter-feed/sequence-tweet-post.png)

Annotated-HLD deep dive (callouts, no new components):

![Annotated HLD](examples/twitter-feed/hld-annotated-stampede.png)

Zoom-in deep dive (new subsystem):

![Zoom-in](examples/twitter-feed/deep-dive-fanout.png)

## Installation

Drop the skill into your project's `.claude/skills/` directory:

```bash
cd <your-project>
mkdir -p .claude/skills
git clone https://github.com/at7211/system-design-skill.git .claude/skills/system-design
```

## Setup

One-time, to install the render pipeline:

```bash
cd .claude/skills/system-design/references
uv sync
uv run playwright install chromium
```

Or just tell your agent: *"Set up the system-design skill by following the instructions in SKILL.md."*

## Usage

In Claude Code, ask any of:

> "Design a Twitter feed"
>
> "How would you design a URL shortener?"
>
> "Design the backend for an app like X with 1M DAU across regions"

The skill auto-triggers, runs all 6 phases, and writes output to `<cwd>/system-designs/<slug>/`:

```
system-designs/<slug>/
  design.md                                # all 6 phases as markdown
  hld.excalidraw + hld.png                 # high-level design (Phase 5)
  sequence-<dive>.excalidraw + .png        # temporal-flow deep dive
  hld-annotated-<dive>.excalidraw + .png   # callout deep dive
  deep-dive-<dive>.excalidraw + .png       # zoom-in deep dive
```

The render-and-fix loop iterates each diagram up to 3 times — Claude reads its own rendered PNG, checks for layout issues (text overflow, detached arrows, overlapping shapes, missing schema-adjacent placement on DBs), and patches the JSON before delivering.

## Customize

- **Colors / brand**: edit `references/color-palette.md`. The "System-Design Tier → Palette Mapping" section maps tier names (edge / app / data / cache / async / etc.) to palette entries — adjust the mapping or swap the palette wholesale.
- **Framework methodology**: edit `references/hellointerview-framework.md` to change phase semantics, time budgets, or skip criteria.
- **Element templates**: edit `references/element-templates.md` to add new components or adjust JSON conventions.
- **Deep-dive classification**: edit `references/deep-dive-rules.md` to tune the decision tree between sequence / annotated / zoom-in.

## File structure

```
system-design/
  SKILL.md                              # trigger description + 6-phase workflow
  references/
    hellointerview-framework.md         # 6-phase framework spec
    element-templates.md                # system-design components + inline JSON
    deep-dive-rules.md                  # diagram-type decision tree
    json-schema.md                      # Excalidraw JSON format reference
    color-palette.md                    # brand colors + tier mapping
    render_excalidraw.py                # Playwright renderer
    render_template.html                # renderer HTML shell (Excalidraw 0.18.0 pinned)
    pyproject.toml                      # Python deps (Playwright)
  examples/
    twitter-feed/                       # worked example w/ all 3 deep-dive types
      design.md
      hld.excalidraw + hld.png
      deep-dive-fanout.excalidraw + .png
      hld-annotated-stampede.excalidraw + .png
      sequence-tweet-post.excalidraw + .png
```

## Credits

Upstream: [coleam00/excalidraw-diagram-skill](https://github.com/coleam00/excalidraw-diagram-skill) — render pipeline, Excalidraw JSON schema reference, color palette structure.

Framework: [HelloInterview — System Design In a Hurry: Delivery](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery).
