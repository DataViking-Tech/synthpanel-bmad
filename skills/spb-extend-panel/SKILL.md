---
name: spb-extend-panel
description: Extends a prior SynthPanel run with a follow-up question; persona continuity preserved via the extend_panel MCP tool. Use when the user says 'extend that panel', 'follow up with the same panel', 'ask the panel one more thing', or 'press the panel on the dissent'.
---

# Extend Panel

> **Status:** stub. Filled in by IMPL-4 (`spb-impl-4`).
>
> Workflow shape and stages canonical in `design-notes/workflows.md` §5 (DESIGN-3).

Follow-up workflow on an existing `panel_verdict.json`. Wraps the SynthPanel
MCP tool `extend_panel`, preserving persona realization from the prior run.
Default `additional_size=0` (re-poll same panel with new question), range 0–12.

Stages (per workflows.md §5):

| # | Stage | Prompt |
|---|---|---|
| 0 | Preflight + resolve `prior_run_id` | `references/preflight-ep.md` |
| 1 | Trigger check (follow-up, not reframe) | `references/trigger-check-ep.md` |
| 2 | Frame `follow_up_question` | `references/frame-ep.md` |
| 3 | Assemble (additional_size, same pack/quotas) | `references/assemble-ep.md` |
| 4 | Call `extend_panel` | `references/call.md` (shared shape) |
| 5 | Validate (incl. orphan-extension fallback) | `references/validate.md` (shared shape) |
| 6 | Persist as `verdicts/{run_id}-ext-{n}.json` + index | `references/persist-ep.md` |
| 7 | Readback (extension-flavored, diff vs parent) | `references/readback-ep.md` |

If the user is reframing the original question, route to `spb-convene-panel`
instead — extension preserves persona continuity, reframing breaks it.

`customize.toml` overlays EP-specific scalars (`default_additional_size`,
`allowed_additional_size_range`, `readback_show_diff_vs_parent`).

> **Open clarification (workflows.md §2):** spec §1 lists `extend_panel` in
> the "Lives on" row for `decision_being_informed`. This workflow treats it
> as **inherited from the prior run, not re-supplied**. If the server actually
> requires a fresh value, Stage 4 echoes the prior verdict's
> `meta.decision_being_informed` verbatim.
