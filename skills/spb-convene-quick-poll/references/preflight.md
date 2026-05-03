# Stage 0 — Preflight (QP)

Probe the SynthPanel MCP toolset. Cheap fail-fast before any user-facing work.

## What to check

1. **`run_quick_poll` is reachable.** Required. If absent, stop with the error
   below.
2. **Schema version.** If the server advertises a `schema_version` outside the
   `1.0.x` band declared in this skill's frontmatter (`mcp.schema_version`),
   record the mismatch on the verdict (Stage 6 stamps it onto `meta`) and
   continue — the response-side validator will catch real drift via the
   `schema_drift` flag (spec §5).

## On unreachable MCP

Stop with:

> **SynthPanel MCP server not reachable.** This pack ships only the workflow
> shell — the server is a documented prerequisite. See the README "Setup"
> section for install + host-config instructions, then retry. (DESIGN-5.)

Do NOT try to install or auto-configure the server (DESIGN-5 §"Why not bundle
setup"). Surface and exit.

## Headless mode

Headless callers (Quinn invoking via routing, or `--headless`) skip stages 1–3
but still run preflight. A headless caller assumed to have a frame is **not**
assumed to have a server.

## Exit criteria

`run_quick_poll` reachable. Note any warnings on a scratch buffer for the
readback.
