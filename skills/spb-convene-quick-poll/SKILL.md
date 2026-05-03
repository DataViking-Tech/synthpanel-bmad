---
name: spb-convene-quick-poll
description: Convenes a fast 5–10-persona quick-poll for low-stakes copy/naming/shortlist decisions via the SynthPanel run_quick_poll MCP tool. Use when the user says 'quick poll', 'fast signal on copy', 'pick between these names', or 'sanity-check this headline'.
---

# Convene Quick Poll

> **Status:** stub. Filled in by IMPL-4 (`spb-impl-4`).
>
> Workflow shape and stages canonical in `design-notes/workflows.md` §3 (DESIGN-3).

Wraps the SynthPanel MCP tool `run_quick_poll` for fast, low-stakes signal
(copy variants, naming shortlists, sub-headline phrasing). Default size 8,
range 5–10.

Stages (per workflows.md §3):

| # | Stage | Prompt |
|---|---|---|
| 0 | Preflight | `references/preflight.md` |
| 1 | Trigger check (QP class) | `references/trigger-check-qp.md` |
| 2 | Frame `decision_being_informed` | `references/frame.md` |
| 3 | Assemble (size/pack/quotas) | `references/assemble-qp.md` |
| 4 | Call `run_quick_poll` | `references/call.md` |
| 5 | Validate envelope | `references/validate.md` |
| 6 | Persist verdict + card | `references/persist.md` |
| 7 | Readback (QP-flavored) | `references/readback-qp.md` |

Activation routes by mode (autonomous / yolo / guided). Headless short-circuits
to stages 4–7. Readback explicitly names "not ship-blocking" up front.

`customize.toml` overlays QP-specific scalars (`default_size`,
`allowed_size_range`, `readback_lede_template`, `small_n_floor_class`).
