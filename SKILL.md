---
name: system-design
description: Use when the user asks to design a system, architect or scale a service, or asks "how would you design X" / "design a Y system". Runs the HelloInterview 6-phase Delivery framework end-to-end and produces Excalidraw diagrams (1 HLD + 2-3 deep dives) with a Playwright render-and-fix validation loop. Output goes to `<cwd>/system-designs/<slug>/`.
---

# System Design Skill

A full-auto system-design responder. Given a system-design prompt, runs all 6 phases of the HelloInterview Delivery framework and produces a `design.md` plus Excalidraw diagrams in `<cwd>/system-designs/<slug>/`.

## Setup (one-time)

If `references/.venv/` does not exist, run before first use:

```bash
cd .claude/skills/system-design/references
uv sync
uv run playwright install chromium
```

## Inputs

The skill triggers on prompts like:
- "Design a Twitter feed"
- "How would you design a URL shortener?"
- "Design the backend for an app like Setlog"
- "幫我設計一個 X"

Derive a kebab-case `<slug>` from the prompt's main subject (e.g., "Twitter feed" → `twitter-feed`).

## Workflow

For every invocation, execute these steps in order:

1. **Determine slug and create output directory**

   ```bash
   mkdir -p <cwd>/system-designs/<slug>
   ```

2. **Read references** — load `references/hellointerview-framework.md`, `references/element-templates.md`, `references/deep-dive-rules.md`, and the worked example at `examples/twitter-feed/design.md` (and skim the example's `.excalidraw` files to learn the JSON style).

3. **Run Phase 1: Requirements** per `hellointerview-framework.md`. Append to `<cwd>/system-designs/<slug>/design.md`.

4. **Run Phase 2: Core Entities.** Append.

5. **Run Phase 3: API / System Interface.** Append.

6. **Run Phase 4: Data Flow** — only if the system is data-processing per the skip criteria in the framework. Append or skip section entirely.

7. **Run Phase 5: High-Level Design.**
   - Build `hld.excalidraw` using `element-templates.md` JSON templates.
   - Apply layout convention (left-to-right tiers, vertical stacking for peers, region cluster containers if global, schema-adjacent for DBs, cost annotations if Phase 1 surfaced cost as a driver).
   - Run the render-and-fix loop (see below).
   - Append narrative section to `design.md` referencing `hld.png`.

8. **Run Phase 6: Deep Dives.**
   - Pick 2-3 deep dives from the non-functional requirements (max 3 hard cap).
   - For each, classify the diagram type using `deep-dive-rules.md`.
   - Generate the `.excalidraw` file using the appropriate filename:
     - Annotated HLD: `hld-annotated-<dive-slug>.excalidraw`
     - Zoom-in: `deep-dive-<dive-slug>.excalidraw`
     - Sequence: `sequence-<dive-slug>.excalidraw`
   - Run render-and-fix loop on each.
   - Append `### <Topic>` section under `## Deep Dives` in `design.md`.

9. **Print summary in chat** — list `design.md` path, every `.excalidraw` + `.png` produced, and the deep-dive topics chosen.

## Render-and-Fix Loop

For every diagram produced in Phase 5 and Phase 6:

1. Write `.excalidraw` JSON to its target path.
2. Render to PNG:

   ```bash
   cd .claude/skills/system-design/references
   uv run python render_excalidraw.py <input>.excalidraw --output <output>.png
   ```

3. Read the PNG using the Read tool.
4. Inspect against this checklist:
   - **Text overflow**: any label extending beyond its container's bounding box?
   - **Arrow endpoint detached**: any arrow not visually connecting to its source/target shape?
   - **Overlapping elements**: any two shapes intersecting unintentionally?
   - **Schema adjacency**: every DB element has its schema columns block immediately to the right?
   - **Region container fit**: every region cluster container fully wraps its child components with ~20px padding?
   - **Unbalanced spacing**: gaps between tiers wildly inconsistent?
   - **Illegible labels**: text size too small or low contrast?
5. If issues found AND iteration count < 3: patch the JSON, re-render, re-inspect.
6. If iteration == 3 AND issues remain: deliver current state and append a "Known render issues" note to the relevant section of `design.md`.

## Output Structure

```
<cwd>/system-designs/<slug>/
  design.md                                  # all 6 phases as markdown
  hld.excalidraw + hld.png                   # Phase 5
  hld-annotated-<dive>.excalidraw + .png     # Phase 6 (if annotated type chosen)
  deep-dive-<dive>.excalidraw + .png         # Phase 6 (if zoom-in type chosen)
  sequence-<dive>.excalidraw + .png          # Phase 6 (if sequence type chosen)
```

## Reference Files

- `references/hellointerview-framework.md` — phase specs, time budgets, skip criteria
- `references/element-templates.md` — system-design component JSON, layout conventions
- `references/deep-dive-rules.md` — diagram-type decision tree
- `references/json-schema.md` — Excalidraw JSON format
- `references/color-palette.md` — colors
- `references/render_excalidraw.py` — Playwright renderer
- `examples/twitter-feed/` — fully worked example covering all 3 deep-dive diagram types
