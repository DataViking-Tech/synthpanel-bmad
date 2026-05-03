# Stage 3 — Assemble (QP)

Pick poll size, pack, and quotas. Defaults must be defendable at readout time.

## Size

- **Default `size` = `{workflow.default_size}` (8).**
- Allow override within `{workflow.allowed_size_range}` (5–10).
- If the user requests `size > 10`, stop and offer `spb-convene-panel`
  (see trigger-check-qp.md "When the user asks for n>10"). Do NOT silently
  up-route.
- If the user requests `size < 5`, decline — below n=5 the convergence
  metric is statistically uninformative.

## Pack selection (DESIGN-2 §7 rule 1)

1. **Audience ≠ panel.** Pick the pack that matches the *audience*, not the
   asker. For QP-class calls (copy / naming) the audience is usually the
   reader of the copy: end-users, customers, prospects.
2. **Warn if asker == audience cohort.** Self-cohort selection is the most
   common failure mode and the most invisible at small n. Surface it
   explicitly when detected.
3. **Mismatch surfaces, doesn't substitute.** If no pack covers the audience,
   run anyway with `out_of_distribution` flag (spec §3) — do NOT silently
   substitute a closer-but-wrong pack.

`pack_id` resolution order:
1. Explicit user argument.
2. `{workflow.default_pack}` from customize.toml.
3. Quinn's audience-derived pick (interactive only — ask the user to confirm).

## Quotas

Optional `quotas` object: `{cell_id: weight}` over the persona pack. QP-class
runs typically don't bother with quotas — at n=8 quota integrity is fragile.
If the user supplies quotas, accept and pass through; expect
`demographic_skew` to fire more often than at RP sizes.

## small_n floor (QP class)

`{workflow.small_n_floor_class}` is `"qp"`. Quinn's facilitator skill maps
decision class to a floor. For the QP class, hire/fire and pricing-tier
decisions WILL trip `small_n` — that's the design. Surface the prediction at
convene time:

> "Heads up: at n=8 a hire-class decision will trip the `small_n` flag in the
> readout. If you want a clean signal, switch to `spb-convene-panel` at n≥18.
> Otherwise we run with the flag — your call."

## Headless mode

Inputs come from the call site. Apply the same defaults from customize.toml;
skip the audience-confirmation prompt. `out_of_distribution` and `small_n`
are stamped into the verdict regardless.

## Exit criteria

`{size, pack_id, quotas?}` resolved and ready to pass to Stage 4. Any flags
known *before* the call (audience mismatch, small_n likely) noted for readback.
