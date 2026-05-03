# Stage 3 — Assemble (EP)

EP inherits panel composition from the prior run by default. The point of EP
is persona continuity — deviating defeats the purpose.

## additional_size

- **Default `additional_size` = `{workflow.default_additional_size}` (0).**
- Allow override within `{workflow.allowed_additional_size_range}` (0–12).
- `additional_size = 0` re-polls the same realized panel with the new
  question. This is the canonical EP shape.
- `additional_size > 0` broadens the panel — useful when the prior run was
  narrow (n=8 QP that the user wants to bolster) or when the follow-up
  warrants new perspectives.

## What is NOT user-overridable here (by default)

`{workflow.inherit_pack_from_parent}` and `{workflow.inherit_quotas_from_parent}`
default to `true` — pack and quotas come from the prior verdict. Overriding
either at the team/user customize.toml level requires an audited reason: a
different pack means different personas, which means the persona-realization
replay is meaningless.

If the user explicitly asks for a different pack, route them to
`spb-convene-panel` instead — they want a fresh run, not an extension.

## small_n at extension time

If the prior run was small (n<12) and `additional_size = 0`, the resulting
extended verdict will inherit the small-n risk. Surface it:

> "Heads up: extending an n=8 prior run with `additional_size=0` re-polls
> the same 8 personas. The `small_n` flag will fire on the extension too.
> Set `additional_size>=10` to broaden, or accept the flag in the readout."

## Headless mode

Inputs come from the call site. Apply customize.toml defaults; skip
confirmation prompts. `small_n` and `out_of_distribution` are stamped into
the extension verdict regardless.

## Exit criteria

`{additional_size}` resolved. Pack and quotas are sourced from the prior
verdict (no further user input). Hand off to Stage 4 (call).
