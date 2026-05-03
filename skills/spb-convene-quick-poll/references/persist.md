# Stage 6 — Persist verdict + card

Write two artifacts under `{project-root}/_bmad/synthpanel/`:

1. `verdicts/{run_id}.json` — the raw `panel_verdict.json` envelope. Source of
   truth for re-rendering (`spb-render-readback`) and for follow-ups
   (`spb-extend-panel`).
2. `cards/{run_id}.md` — the rendered markdown card (the "hero artifact" per
   launch-plan). Screenshottable, pasteable into a PR.

Paths come from `customize.toml`:

- `{workflow.verdict_output_path}` (default `…/verdicts/{run_id}.json`)
- `{workflow.readback_card_path}` (default `…/cards/{run_id}.md`)

## `run_id` resolution

`run_id` comes from the MCP server response (typically `meta.run_id` or
`extension[].run_id`). If absent — schema_drift, older server, etc. —
generate a UUIDv7 client-side and stamp it into `meta.run_id` before save.
This guarantees the verdict survives multiple workflow runs in one session
and is addressable by `spb-extend-panel` and `spb-render-readback`.

## What to persist on the verdict JSON

Save the envelope **verbatim** — no reformatting, no field reordering,
no paraphrase. The audit chain (spec §2) depends on byte-stable persistence.

Add a `meta.workflow_warnings` array (workflow-level only — never inside
`extension[]`) for:

- Schema-version mismatch noted at Stage 5.
- `extend_panel` orphan-extension warning (EP only — see persist-ep.md).
- Any pre-call warnings noted at Stage 3 (audience mismatch, small_n
  predicted but accepted).

If `seed` was supplied to `run_panel`, persist it on the verdict (the server
typically echoes it; if not, stamp it from the request body). This lets
`spb-extend-panel` replay persona realization.

## What to persist on the card

The rendered markdown card from Stage 7 (readback). Atomic write — render in
memory, then write once. Never partial-write a card.

`{workflow.write_card_to_disk}` from Quinn's customize.toml may disable disk
write at the agent level; in that case, render to terminal only. Default
behavior is to write.

## Directory creation

Create parent directories if they don't exist (`mkdir -p` semantics). Do not
fail the workflow on directory creation issues — surface and continue to
Stage 7 with terminal-only readback.

## On_complete hook

If `{workflow.on_complete}` is non-empty, fire it after both artifacts are
written. Pass the `run_id` and resolved paths as environment variables:
`SPB_RUN_ID`, `SPB_VERDICT_PATH`, `SPB_CARD_PATH`. Hook failure does NOT fail
the workflow — log and continue.

## Exit criteria

Both artifacts on disk (or surfaced as terminal-only if writes failed). Hand
off to Stage 7 (readback).
