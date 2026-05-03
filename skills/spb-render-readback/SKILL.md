---
name: spb-render-readback
description: Re-renders a saved SynthPanel panel_verdict.json as the markdown verdict card. Use when the user says 'render that verdict again', 'show me the card for run X', 'redo the readback', or 'regenerate the panel card'.
---

# Render Readback

> **Status:** stub. Filled in by IMPL-4 (`spb-impl-4`).
>
> Workflow shape and stages canonical in `design-notes/workflows.md` §6 (DESIGN-3).

Local-only workflow. Re-renders a saved `panel_verdict.json` as the markdown
card. No MCP call. Useful for stale terminals, sharing a verdict from a prior
session, or regenerating after a `customize.toml` template change.

Stages (per workflows.md §6):

| # | Stage | Prompt |
|---|---|---|
| 0 | (skip — no MCP) | |
| 1 | Resolve `run_id` or path | `references/resolve-rb.md` |
| 2 | Render against current `readback_card_path` template | `references/render-rb.md` |
| 3 | Persist (optional `--write`) | `references/persist-rb.md` |

No `customize.toml` of its own; reads the panel-running workflow's
`readback_card_path` template from `{project-root}/_bmad/config.yaml`.

Surfaces a schema-version mismatch if the card template expects newer fields
than the saved verdict has — the user is told what's missing rather than
silently rendering an incomplete card.
