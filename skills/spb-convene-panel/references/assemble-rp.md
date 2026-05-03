# Stage 3 — Assemble (RP)

Pick panel size, pack, and quotas. Defaults must be defendable at readout time.

## Size

- **Default `size` = `{workflow.default_size}` (16).**
- Allow override within `{workflow.allowed_size_range}` (12–40).
- If the user requests `size < 12`, stop and offer `spb-convene-quick-poll` —
  do NOT silently down-route. The menu code is part of the audit story.
- If the user requests `size > 40`, accept but warn that latency and
  per-persona quality may degrade past the suggested ceiling.

## Pack selection (DESIGN-2 §7 rules)

1. **Audience ≠ panel.** Pick the pack that matches the *audience*, not the
   asker. If the user is an indie dev choosing pricing for indie devs, pack
   is `developer` / `indie-builders`, not "self." Self-cohort selection is
   the most common failure mode — flag it explicitly when detected.
2. **Mismatch surfaces, doesn't substitute.** If no pack covers the audience,
   run anyway with `out_of_distribution` flag (spec §3) — do NOT silently
   substitute a closer-but-wrong pack.
3. **Quota drift gets named.** If realized quotas drift >15% from requested,
   `demographic_skew` flag fires (spec §3). Name the drifted cell at readback.

`pack_id` resolution order:
1. Explicit user argument.
2. `{workflow.default_pack}` from customize.toml.
3. Quinn's audience-derived pick (interactive only — ask the user to confirm).

## Quotas

Optional `quotas` object: `{cell_id: weight}` over the persona pack. Use
templates from `{workflow.panel_template}` if provided (a path to a
`panelists.yaml` shape — DESIGN-3 §2). Empty quotas = pack defaults.

## small_n floor (RP class)

`{workflow.small_n_floor_class}` is `"rp"`. Quinn's facilitator skill maps
decision class to a floor (see `bmad-agent-facilitator/references/sizing-and-packs.md`).
For a hire-class decision Quinn names the floor *before* the run:

> "For a hiring-class decision I'd run n≥18 or you'll trip the `small_n` flag
> — want to bump the size or accept the flag in the readout?"

The flag is not a failure; it is a documented limitation. Either choice is fine
as long as the user is informed.

## Headless mode

Inputs come from the call site (frame already framed at Stage 2). Apply the
same defaults from customize.toml; skip the audience-confirmation prompt.
`out_of_distribution` and `small_n` are stamped into the verdict regardless.

## Exit criteria

`{size, pack_id, quotas?}` resolved and ready to pass to Stage 4. Any flags
known *before* the call (audience mismatch, small_n likely) noted for readback.
