---
name: spb-convene-panel
description: Convenes a 12–20-persona structured deliberation panel for hard-to-reverse product decisions (pricing, positioning, hiring class, launch lede) via the SynthPanel run_panel MCP tool. Use when the user says 'convene a panel', 'run a deliberation', 'panel this decision', or 'I need cohort signal on a hard call'.
---

# Convene Panel

> **Status:** stub. Filled in by IMPL-4 (`spb-impl-4`).
>
> Workflow shape and stages canonical in `design-notes/workflows.md` §4 (DESIGN-3).

The flagship workflow. Wraps the SynthPanel MCP tool `run_panel` for structured
deliberation on pricing, positioning, naming for product-level scope, launch
lede, hiring class. Default size 16, range 12–40.

Stages (per workflows.md §4):

| # | Stage | Prompt |
|---|---|---|
| 0 | Preflight | `references/preflight.md` |
| 1 | Trigger check (RP class) | `references/trigger-check-rp.md` |
| 2 | Frame `decision_being_informed` | `references/frame.md` |
| 3 | Assemble (size/pack/quotas) | `references/assemble-rp.md` |
| 4 | Call `run_panel` | `references/call.md` |
| 5 | Validate envelope | `references/validate.md` |
| 6 | Persist verdict + card | `references/persist.md` |
| 7 | Readback (full RP shape) | `references/readback-rp.md` |

Activation routes by mode. Headless short-circuits to stages 4–7. When
convergence < 0.45, Stage 7 offers `extend-panel` as a soft next step
(`suggest_extend_on_low_convergence` in `customize.toml`).

`customize.toml` overlays RP-specific scalars (`default_size`,
`allowed_size_range`, `readback_lede_template`, `small_n_floor_class`,
`suggest_extend_on_low_convergence`).
