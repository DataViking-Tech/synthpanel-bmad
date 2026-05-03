# Stage 5 — Validate envelope

Inspect the MCP response. Two shapes per spec: a successful `panel_verdict.json`
envelope (spec §2) or a typed error envelope (spec §4).

## Typed-error handling (spec §4)

| Error code | Action |
|---|---|
| `MISSING_DECISION` | Loop back to Stage 2 (frame). User-correctable. |
| `DECISION_TOO_LONG` | Loop back to Stage 2 (frame). User-correctable. |
| `INVALID_TOOL_ARG` | Loop back to Stage 3 (assemble) or Stage 4 (call) depending on the offending field. Not retry-safe. |
| `MODEL_TIMEOUT` | Retry once with the same body. If it fails again, surface and stop — do not auto-retry past one. |
| `INTERNAL_ERROR` | Surface and stop. Not retryable. Include the error's `request_id` (if present) in the surface message. |

Every error envelope carries `schema_version`. Stamp this onto the surface
message so the user can correlate with server logs.

## Success envelope handling (spec §2)

Validate the response shape. Required fields:

- `headline` (string ≤ 140 chars)
- `convergence` (number 0.0–1.0)
- `dissent_count` (integer ≥ 0)
- `top_3_verbatims` (array of `{persona_id, quote}`)
- `flags[]` (closed enum — see below)
- `extension[]` (open escape hatch — agents MUST NOT branch on this; pass through)
- `full_transcript_uri` (string)
- `meta.decision_being_informed` (verbatim echo)
- `schema_version` (string)

If any required field is missing or wrong-typed, treat as `INTERNAL_ERROR`
and surface — do not paper over a malformed envelope.

## Flag handling (closed enum, spec §3)

Flags are warnings, not errors. The workflow continues regardless; the
readback (Stage 7) leads with flags when severity ≥ warn.

| Flag | What it means | Notes |
|---|---|---|
| `low_convergence` | convergence < 0.45 | RP: triggers extend-suggestion at Stage 7. |
| `small_n` | size below the floor for this decision class | Don't suppress — feature, not bug. |
| `out_of_distribution` | requested pack didn't cover the audience | Surface verbatim at readout. |
| `demographic_skew` | realized quotas drifted >15% from requested | Name the cell. |
| `persona_collision` | ≥2 personas near-duplicate (cosine >0.92) | Convergence may be inflated. |
| `refusal_or_degenerate` | model refused or returned degenerate output | Lead with this. |
| `schema_drift` | structured-output retry exhausted (spec §5) | Continue; stamp degradation on card. |

`extension[]` is open and agents MUST NOT branch on it. Persist it for audit;
do not surface it in the readback unless the user asks.

## Schema-version mismatch

If the response's `schema_version` is outside the `1.0.x` band declared in
this skill's frontmatter, treat it as a *workflow-level warning*: stamp the
mismatch in `meta.workflow_warnings` at persist time and continue. The
`schema_drift` flag is for *parsing* drift; this is *contract* drift.

## Exit criteria

Either: (a) typed error surfaced and workflow looped or stopped, or
(b) envelope validated, flags noted, hand off to Stage 6.
