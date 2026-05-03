# Stage 7 — Readback (QP-flavored)

Render the verdict per the QP shape: **lede the limit first**, then the
standard convergence/dissent/flags shape (DESIGN-2 §8).

## QP-flavored card template

```
🪧 {readback_lede_template_resolved}

  Frame: {decision_being_informed}
  Convergence: {convergence:.2f} ({band})
  Dissent: {dissent_count} persona(s)
  Flags: {flag.code [severity], …  OR  "none"}

Top dissenting voices:
  • [{persona_id}] "{verbatim quote}"
  • [{persona_id}] "{verbatim quote}"
  • [{persona_id}] "{verbatim quote}"

What this poll is and isn't:
  • Quick-poll synthetic signal (n={size}, pack={pack_id}).
  • Not ship-blocking. For load-bearing decisions, run a full panel
    (spb-convene-panel, n=12+).
  {if flags.severity ≥ warn: • Flagged: {one-line gloss per flag}.}

Full transcript: {full_transcript_uri}
```

`{workflow.readback_lede_template}` resolves to:
> `Quick-poll (n=8, pack=indie-builders) — not ship-blocking. {headline}`

The "not ship-blocking" phrase is non-negotiable in the QP lede. It's the
honest framing that makes QP usable without users mistaking it for RP.

## Convergence bands

Same bands as RP (see `../../spb-convene-panel/references/readback-rp.md`).
At n=8 the convergence metric is noisier — the user should treat any band
classification as a hint, not a measurement.

## Flag handling at readout

Same rules as RP (see `references/validate.md`):

- Lead with `low_convergence`, `small_n`, `out_of_distribution`,
  `refusal_or_degenerate` if any are present.
- `small_n` at QP n=5 on a hire-class decision: surface, don't suppress
  (DESIGN-2 §7 floor mapping). The flag is the design.
- `schema_drift`: render the card with the degradation stamped.

## Verbatims discipline

Verbatims are **verbatim** — same rules as RP. No re-rank, no paraphrase,
no truncation. The audit chain (spec §2) is identical at QP and RP sizes.

## QP-specific things to never do

- Drop the "not ship-blocking" lede. That's the contract with the user.
- Suggest `spb-extend-panel` from a QP run. EP is for RP follow-ups; if QP
  raised a question worth extending, the right move is a fresh
  `spb-convene-panel`, not extending an n=8 poll.
- Cheerlead high QP convergence. n=8 convergence at 0.92 is noisier than
  n=16 convergence at 0.78. Voice stays calm, calibrated.

## Exit criteria

Card rendered. If `{workflow.write_card_to_disk}` is true, also persisted at
`{workflow.readback_card_path}` (Stage 6 handles the write).
