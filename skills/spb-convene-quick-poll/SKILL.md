---
name: spb-convene-quick-poll
type: mcp-tool-wrapper
description: Convenes a fast 5–10-persona quick-poll for low-stakes copy/naming/shortlist decisions via the SynthPanel run_quick_poll MCP tool. Use when the user says 'quick poll', 'fast signal on copy', 'pick between these names', or 'sanity-check this headline'.
mcp:
  server: synthpanel
  tool: run_quick_poll
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
      description: Single-line UTF-8 frame for the decision this poll informs. Newlines rejected upstream (spec §1).
    pack_id:
      type: string
      description: Persona pack identifier. Resolves from customize.toml.default_pack when omitted.
    size:
      type: integer
      minimum: 5
      maximum: 10
      default: 8
      description: Number of personas. QP class floor 5, ceiling 10. n>10 must up-route to spb-convene-panel.
    quotas:
      type: object
      additionalProperties:
        type: number
      description: Optional quota cells over the persona pack.
sha256: f6884c8b7835b67605da661156ac9ff25d9e1e7fb62b125f401cd48f62543a2f
---

# Convene Quick Poll

Wraps the SynthPanel MCP tool `run_quick_poll` for fast, low-stakes signal
(copy variants, naming shortlists, sub-headline phrasing). Default size 8,
range 5–10. Workflow shape canonical in `design-notes/workflows.md` §3.

## Stages

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

Activation routes by mode (autonomous / yolo / guided). Headless mode
short-circuits to stages 4–7. Readback explicitly names "not ship-blocking"
up front.

## MCP tool contract

Stage 4 constructs a request body strictly per spec §1 against the
`inputSchema` declared in this skill's frontmatter. The agent MUST validate
the request against `inputSchema` before invoking `run_quick_poll`; servers
may also reject (typed errors `MISSING_DECISION`, `DECISION_TOO_LONG`,
`INVALID_TOOL_ARG` per spec §4).

Response envelope (`panel_verdict.json`) is validated per spec §2; the
closed-enum `flags[]` (spec §3) drives Stage 7 readback shape.

## Customization

`customize.toml` overlays QP-specific scalars:

```toml
default_size = 8
allowed_size_range = [5, 10]
readback_lede_template = "Quick-poll (n={size}, pack={pack_id}) — not ship-blocking. {headline}"
small_n_floor_class = "qp"
```

## Failure modes

- User asks for n>10 → Stage 3 stops and offers `spb-convene-panel`.
- `small_n` flag fires at QP n=5 on a hire-class decision → surface, do not
  suppress (DESIGN-2 §7 floor mapping).
