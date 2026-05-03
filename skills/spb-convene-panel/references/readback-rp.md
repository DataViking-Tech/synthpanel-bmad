# Stage 7 — Readback (RP, full shape)

Render the verdict per the canonical RP shape (DESIGN-2 §8). Lead with the
frame, then convergence band, then dissent count, then top-3 verbatims, then
flags, then "what this panel is and isn't."

## Canonical card template

```
🪧 Verdict — {decision_being_informed}

  Headline: {headline}
  Convergence: {convergence:.2f} ({band})
  Dissent: {dissent_count} persona(s)
  Flags: {flag.code [severity], …  OR  "none"}

Top dissenting voices:
  • [{persona_id}] "{verbatim quote}"
  • [{persona_id}] "{verbatim quote}"
  • [{persona_id}] "{verbatim quote}"

What this panel is and isn't:
  • Synthetic population estimate (n={size}, pack={pack_id}).
  • Not a substitute for talking to {n} real {audience}.
  {if flags.severity ≥ warn: • Flagged: {one-line gloss per flag}.}

Full transcript: {full_transcript_uri}
```

`readback_lede_template` from customize.toml = `"{decision_being_informed}"` —
the frame IS the lede for RP.

## Convergence bands (DESIGN-2 §8)

| Range | Band | Inline gloss |
|---|---|---|
| 0.85–1.00 | very high | "Strong agreement — but check `persona_collision`; if clean, this is signal." |
| 0.65–0.84 | high | "Solid agreement with meaningful texture in the dissent." |
| 0.45–0.64 | moderate | "Real split — exactly the kind of decision a panel earns its cost on." |
| 0.20–0.44 | low | "Low convergence — read the dissent before deciding; consider extending on the cleavage." |
| 0.00–0.19 | very low | "The panel is essentially split. The decision frame may be ambiguous; reframe and re-run." |

Round convergence to two decimals. Never drop a bare float.

## Flag handling at readout

- **Always** name flags before celebrating convergence. If `low_convergence`,
  `small_n`, `out_of_distribution`, or `refusal_or_degenerate` is present,
  the verdict *leads* with that fact.
- **`schema_drift`:** "This is a degraded artifact — the structured-output
  retry exhausted (spec §5). Read with extra skepticism; the panel ran but
  parsing was unreliable."
- **`persona_collision`:** "Two or more personas were near-duplicates
  (cosine >0.92). Treat the agreement as inflated by the collision."
- **`demographic_skew`:** Name the drifted quota cell explicitly and the
  realized vs requested split.
- **Multiple flags:** highest severity wins for tone; all flags still listed.

## Verbatims discipline

The `top_3_verbatims` are **verbatim**. Render the quotes byte-stable. Do not:

- Re-rank or re-pick (the orchestrator's selection is the verdict — tampering
  breaks the audit chain, spec §2).
- Substitute paraphrase. If the quote contains a typo or unusual phrasing,
  that's the data.
- Truncate mid-quote. If a quote runs long, render it in full; the card is
  not a tweet.

## Extend-suggestion (RP-specific, optional)

When `convergence < {workflow.extend_suggestion_threshold}` (default 0.45)
AND `{workflow.suggest_extend_on_low_convergence}` is true, append a single
soft nudge:

> Convergence here was low (0.34). If you want to press the panel on the
> cleavage, run `spb-extend-panel` with `prior_run_id={run_id}` and a
> follow-up question naming the dissent.

Auto-invoke is intentionally NOT done. The user opts in. This is how EP gets
its real-world entry point.

## Things to never do at readout

- Render without the `decision_being_informed` echo. The frame is *part of*
  the verdict, not metadata.
- Drop flags because they "aren't relevant." Severity gates that.
- Re-style the card on user request. Restyling for one user breaks the
  recognition pattern across the cohort.
- Cheerlead ("Great news — the panel loves it!") or hedge ("It seems some
  people might possibly think…").

## Exit criteria

Card rendered. If `{workflow.write_card_to_disk}` is true, also persisted at
`{workflow.readback_card_path}` (Stage 6 handles the write).
