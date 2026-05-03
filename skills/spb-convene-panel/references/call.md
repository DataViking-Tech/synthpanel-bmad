# Stage 4 — Call MCP tool

Construct the request body strictly per spec §1 against the `inputSchema`
declared in this skill's frontmatter, then invoke the MCP tool.

## Request shape

For `run_panel`:

```json
{
  "decision_being_informed": "<single-line UTF-8, 12–280 chars>",
  "pack_id": "<optional pack id>",
  "size": <integer 12–40, default 16>,
  "quotas": {"<cell_id>": <weight>, ...},
  "seed": <optional integer>
}
```

For `run_quick_poll`: same shape, `size` range 5–10 (default 8), no `seed`.

For `extend_panel`:

```json
{
  "prior_run_id": "<run_id resolved at Stage 0>",
  "follow_up_question": "<single-line UTF-8, 12–280 chars>",
  "additional_size": <integer 0–12, default 0>
}
```

`decision_being_informed` is **NOT** a request field for `extend_panel`
(workflows.md §2 open clarification — inherited from prior run via the
transcript URI; if the server requires it on `extend_panel`, echo
`prior_verdict.meta.decision_being_informed` verbatim).

## Required pre-call validation

Before invoking the tool, validate the constructed body against the
`inputSchema` in this skill's frontmatter. Request-side validation is a cheap
reject (spec §6) — surfacing a typed error from the server costs a network
round-trip we can avoid.

Specifically:

- `decision_being_informed` (or `follow_up_question`): single-line, 12–280
  chars after trim, UTF-8.
- `size` / `additional_size`: integer within the workflow's allowed range.
- `quotas` keys are strings; values are numeric.
- No additional properties (per `additionalProperties: false`).

If validation fails: loop back to the relevant stage. Do not paper over.

## Invocation

Call the MCP tool by its registered name:

| Workflow | Tool | Server |
|---|---|---|
| `spb-convene-quick-poll` | `run_quick_poll` | `synthpanel` |
| `spb-convene-panel` | `run_panel` | `synthpanel` |
| `spb-extend-panel` | `extend_panel` | `synthpanel` |

The host invokes the tool. The response is a `panel_verdict.json` envelope
(spec §2), or a typed error envelope (spec §4) — both flow into Stage 5.

## Exit criteria

Tool invoked, response received (envelope or typed error). Hand off to Stage 5.
