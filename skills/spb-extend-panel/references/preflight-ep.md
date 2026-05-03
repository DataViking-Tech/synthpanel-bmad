# Stage 0 — Preflight + resolve `prior_run_id` (EP)

Probe the SynthPanel MCP toolset AND verify the prior run is locally
addressable before any user-facing work.

## What to check

1. **`extend_panel` is reachable.** Required. If absent, stop with the error
   below and offer `spb-convene-panel` as a fresh-start fallback.
2. **`prior_run_id` resolves to a real verdict on disk.**
   - Try `{project-root}/_bmad/synthpanel/verdicts/{prior_run_id}.json`.
   - On miss, list nearest matches (prefix match on `run_id`) and ask the user
     to confirm one.
   - If still no match: surface and stop. Do NOT invoke `extend_panel` with
     an unverified `run_id` — orphan-extension is recoverable but expensive.
3. **Schema version of the prior verdict.** Read `prior_verdict.schema_version`
   from the loaded JSON. If outside the `1.0.x` band, warn the user — extension
   compatibility may be degraded.

## On unreachable MCP

Stop with:

> **SynthPanel MCP server not reachable.** This pack ships only the workflow
> shell — the server is a documented prerequisite. See the README "Setup"
> section for install + host-config instructions, then retry. (DESIGN-5.)

## On missing prior verdict

> Couldn't find verdict for `run_id={prior_run_id}` at
> `{verdict_output_path}`. Either:
>   - Confirm the run_id (it should be the value from `meta.run_id` of the
>     prior run's `panel_verdict.json`), OR
>   - Run a fresh panel via `spb-convene-panel` if the prior verdict is
>     unrecoverable.

## Loaded prior verdict — what to extract

Cache for downstream stages:

- `prior_verdict.meta.decision_being_informed` — echoed to user at Stage 2.
- `prior_verdict.meta.run_id` — passed as `prior_run_id` to the MCP tool.
- `prior_verdict.full_transcript_uri` — server uses this for persona-realization
  replay.
- `prior_verdict.size` and `prior_verdict.pack_id` — Stage 3 inheritance.
- `prior_verdict.convergence` and `dissent_count` — Stage 7 diff.

## Headless mode

Headless callers (Quinn routing, or `--headless`) skip stages 1–2 but still
run preflight AND prior-verdict resolution. A missing prior verdict in
headless mode is a hard surface — don't try to interactively prompt for a
fix.

## Exit criteria

`extend_panel` reachable, prior verdict loaded into memory, run_id confirmed.
