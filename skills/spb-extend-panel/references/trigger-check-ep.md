# Stage 1 — Trigger check (EP)

Confirm this is a follow-up, not a fresh decision. EP wraps `extend_panel` to
preserve persona continuity — same realized panel, new question.

## EP triggers (DESIGN-3 §5)

The user wants to:

- **Press the panel on the dissent.** A prior RP run had `low_convergence` or
  notable `dissent_count`; user wants to ask the same panel a sharper version
  of the question.
- **Follow-up sub-question.** A prior verdict raised a flag or surfaced a
  nuance the user didn't anticipate; new question is a *child* of the original
  decision, not a new one.
- **Probe a specific cleavage.** "What did the dissenters think the cost
  was?" — same panel, scoped follow-up.
- **Validate a tentative direction.** Prior panel pointed toward option A;
  user wants to gut-check a refinement of A with the same cohort.

## Anti-triggers — route elsewhere

| Symptom | Route to |
|---|---|
| User is reframing the original question (fundamentally different decision) | `spb-convene-panel` (RP) — extension preserves continuity, reframing breaks it |
| Prior run was QP-class and user wants RP-grade follow-up | `spb-convene-panel` (RP) — don't extend a QP into RP territory |
| User wants a different audience (different pack) | `spb-convene-panel` (RP) — different pack defeats EP's point |
| No prior `run_id` available (lost the verdict) | `spb-convene-panel` (RP) — re-realize from scratch |

## How to detect "reframe vs follow-up"

The follow-up question should be **answerable by the prior panel**. If the
new question requires personas with different priors, different demographics,
or different domain expertise than the prior run convened, it's a reframe.

Concrete test: read the prior `decision_being_informed` aloud, then the
proposed follow-up. If the follow-up's answer would be *informed by* the
prior verdict, it's an extension. If the answer would *replace* the prior
verdict, it's a reframe — route to RP.

## Headless mode

Headless callers are assumed to know which workflow they're in. Don't
re-litigate the reframe-vs-follow-up call in headless mode unless the
follow-up question itself trips a heuristic (e.g., explicitly names a
different audience).

## Exit criteria

Either: this is a follow-up to the loaded prior verdict and we proceed to
Stage 2 (frame), OR we declined / routed to `spb-convene-panel` with one
explanatory sentence.
