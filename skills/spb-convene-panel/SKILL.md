---
name: spb-convene-panel
type: mcp-tool-wrapper
description: Convenes a 12–20-persona structured deliberation panel for hard-to-reverse product decisions (pricing, positioning, hiring class, launch lede) via the SynthPanel run_panel MCP tool. Use when the user says 'convene a panel', 'run a deliberation', 'panel this decision', or 'I need cohort signal on a hard call'.
mcp:
  server: synthpanel
  tool: run_panel
  schema_version: 1.0.0
inputSchema:
  type: object
  required:
    - decision_being_informed
  additionalProperties: false
  properties:
    decision_being_informed:
      type: string
      minLength: 12
      maxLength: 280
      description: Single-line UTF-8 frame for the decision this panel informs. Newlines rejected upstream (spec §1). Verbatim-echoed into panel_verdict.json.meta.decision_being_informed.
    pack_id:
      type: string
      description: Persona pack identifier. Resolves from customize.toml.default_pack when omitted.
    size:
      type: integer
      minimum: 12
      maximum: 40
      default: 16
      description: Number of personas. RP class floor 12, suggested ceiling 40.
    quotas:
      type: object
      additionalProperties:
        type: number
      description: Optional quota cells over the persona pack.
    seed:
      type: integer
      description: Seed for persona realization. Persisted on the verdict so spb-extend-panel can replay the same realization.
sha256: 00aa51692cbb762e583e0436844168183f984e27c6d794561b67a67662e46a1f
---

# Convene Panel

The flagship workflow. Wraps the SynthPanel MCP tool `run_panel` for
structured deliberation on pricing, positioning, naming for product-level
scope, launch lede, hiring class. Default size 16, range 12–40. Workflow
shape canonical in `design-notes/workflows.md` §4.

## Stages

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

Activation routes by mode. Headless short-circuits to stages 4–7.

## MCP tool contract

Stage 4 constructs a request body strictly per spec §1 against the
`inputSchema` declared in this skill's frontmatter. The agent MUST validate
the request against `inputSchema` before invoking `run_panel` — request-side
validation is a cheap reject (spec §6).

Response envelope (`panel_verdict.json`) is validated per spec §2. On
`flags[].code == "schema_drift"` (spec §5) the workflow continues to readback
but stamps the degradation in the card; on typed errors (spec §4) it loops
back to Stage 2 (`MISSING_DECISION` / `DECISION_TOO_LONG`), retries once
(`MODEL_TIMEOUT`), or surfaces and stops.

## Customization

`customize.toml` overlays RP-specific scalars:

```toml
default_size = 16
allowed_size_range = [12, 40]
readback_lede_template = "{decision_being_informed}"
small_n_floor_class = "rp"
suggest_extend_on_low_convergence = true
```

When `convergence < 0.45` AND `suggest_extend_on_low_convergence = true`,
Stage 7 ends with a soft nudge offering `spb-extend-panel` on the cleavage.
Auto-invoke is intentionally not done; the user opts in.
