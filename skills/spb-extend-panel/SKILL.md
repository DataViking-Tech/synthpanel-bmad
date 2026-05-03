---
name: spb-extend-panel
type: mcp-tool-wrapper
description: Extends a prior SynthPanel run with a follow-up question; persona continuity preserved via the extend_panel MCP tool. Use when the user says 'extend that panel', 'follow up with the same panel', 'ask the panel one more thing', or 'press the panel on the dissent'.
mcp:
  server: synthpanel
  tool: extend_panel
  schema_version: 1.0.0
inputSchema:
  type: object
  required:
    - prior_run_id
    - follow_up_question
  additionalProperties: false
  properties:
    prior_run_id:
      type: string
      description: Run ID of the panel to extend. Resolves locally to verdicts/{run_id}.json and to a server-side full_transcript_uri for persona-realization replay.
    follow_up_question:
      type: string
      minLength: 12
      maxLength: 280
      description: Single-line UTF-8 follow-up. NOT a fresh decision_being_informed — the original frame is inherited from the prior run.
    additional_size:
      type: integer
      minimum: 0
      maximum: 12
      default: 0
      description: Personas to add beyond the prior panel. Default 0 re-polls the same realized panel.
sha256: 0d72d505302db8eecde6fe5d6e800b3ce75fa2a7076e641d796a3790b658f482
---

# Extend Panel

Follow-up workflow on an existing `panel_verdict.json`. Wraps the SynthPanel
MCP tool `extend_panel`, preserving persona realization from the prior run.
Default `additional_size=0` (re-poll same panel with new question), range
0–12. Workflow shape canonical in `design-notes/workflows.md` §5.

## Stages

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

## MCP tool contract

Stage 4 constructs the request body strictly per spec §1 against the
`inputSchema` in this skill's frontmatter. `decision_being_informed` is
**not** a request field for `extend_panel` — it is inherited from the prior
run via the resolved transcript URI (workflows.md §2 open clarification).

Response envelope is validated per spec §2; the orphan-extension case
(prior_run_id resolves locally but server reports unknown run) is surfaced
explicitly per workflows.md §5 — persona continuity is no longer guaranteed,
user opts into either re-`spb-convene-panel` or a re-realized panel stamped
with a warning in the readback.

## Customization

`customize.toml` overlays EP-specific scalars:

```toml
default_additional_size = 0
allowed_additional_size_range = [0, 12]
readback_show_diff_vs_parent = true
```
