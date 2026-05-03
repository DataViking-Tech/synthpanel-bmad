# Stage 1 — Trigger check (RP class)

Confirm this is actually an RP-class call. RP wraps `run_panel` for
hard-to-reverse, audience-bearing decisions; default size 16, range 12–40.

## RP triggers (DESIGN-2 §5)

At least ONE of:

- Phrasing names a binary or N-way choice ("$49 or $79", "ship X or Y").
- Phrasing weighs ≥2 options for a non-trivial outcome.
- Decision is **hard to reverse**: hiring class, pricing tier, product naming,
  strategic positioning, launch lede.
- Stakeholder breadth: users, customers, hiring market, investors.
- User is drafting copy, naming, or positioning **for an external reader**.
- User explicitly asks "what would people think" / "how would users react?".

## Anti-triggers — route elsewhere

| Symptom | Route to |
|---|---|
| Low-stakes copy / naming shortlist, fast signal sought | `spb-convene-quick-poll` (QP) |
| Follow-up on an existing run, persona continuity wanted | `spb-extend-panel` (EP) |
| Sub-decisional scratch work (`run_prompt` territory) | Not a panel call — decline politely |
| Purely technical, no audience (which DB index?) | Suggest an engineering agent |
| Reversible at zero cost ("what should I have for lunch?") | Not a panel call |

## When to up-route from QP

If a user invoked QP but the request matches RP class (hire-class decision,
external-audience copy at scale, ≥12 personas wanted), this skill should
**not** silently absorb it — Stage 3 of QP stops and offers RP. Mirror that
courtesy here: if the user invoked RP for what is clearly a QP-class call
(naming shortlist of 4, n=8 wanted), suggest QP and proceed only if confirmed.

## Exit criteria

Either: this is an RP-class call and we proceed to Stage 2 (frame), OR we
declined / routed to a sibling workflow with one explanatory sentence.
