# Stage 1 — Trigger check (QP class)

Confirm this is actually a QP-class call. QP wraps `run_quick_poll` for fast,
low-stakes signal; default size 8, range 5–10. Quinn explicitly names this is
**not ship-blocking signal** at readback time (DESIGN-2 §7).

## QP triggers (DESIGN-2 §5)

QP-class decisions are short, tactical, and reversible at low cost:

- **Copy variants** — picking between drafted CTAs, sub-headers, micro-copy.
- **Naming shortlists** — choosing among 3–5 candidate names where the choice
  is mostly aesthetic / phonological, not strategic.
- **Sub-headline phrasing** — A/B-style word-level decisions.
- **Quick gut-checks** — "does this read as too snarky for the cohort?"

## Anti-triggers — route elsewhere

| Symptom | Route to |
|---|---|
| Hard-to-reverse: pricing, hiring, positioning, launch lede | `spb-convene-panel` (RP) |
| ≥12 personas requested | `spb-convene-panel` (RP) — the size IS the signal |
| Follow-up on an existing run | `spb-extend-panel` (EP) |
| Sub-decisional scratch work (`run_prompt`) | Not a panel call — decline politely |
| Purely technical, no audience | Suggest an engineering agent |

## When the user asks for n>10

Stop. Offer to up-route to `spb-convene-panel`:

> "n=12 is past the QP ceiling. For that size I'd run a full panel
> (`spb-convene-panel`) — the readback is differently-shaped (no
> 'not ship-blocking' lede) and the menu code is part of the audit story.
> Switch?"

Do **not** silently up-route. Distinct workflows give distinct trigger
surfaces in `module-help.csv` for a reason.

## When `small_n` will fire anyway

QP at n=5 on a hire-class decision will trip `small_n`. That's a feature, not
a bug — DESIGN-2 §7 floor mapping. Surface the prediction at trigger-check
time so the user can opt to up-route OR accept the flag in the readout.

## Exit criteria

Either: this is a QP-class call and we proceed to Stage 2 (frame), OR we
declined / routed to a sibling workflow with one explanatory sentence.
