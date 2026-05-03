# Stage 2 — Frame `follow_up_question` (EP)

Coach the user to a follow-up question the prior panel can answer.

## Constraints

Same as `decision_being_informed` (spec §1):

- **String, 12–280 characters after trim.**
- **Single line.** Newlines rejected upstream.
- **UTF-8.**

## What's NOT happening here

EP does **NOT** re-coach a `decision_being_informed`. The original frame is
inherited from the prior run (workflows.md §2 open clarification). Re-coaching
mid-extension would break the audit chain — the verdict's
`meta.decision_being_informed` would no longer match the panel's prior frame.

If the user wants a new frame, route them to `spb-convene-panel` at trigger
check. By Stage 2 we're committed to extension.

## Echo the prior frame first

Before coaching the follow-up, surface the prior frame so the user sees what
they're extending:

> Prior frame: `{prior_verdict.meta.decision_being_informed}`
> Prior result: convergence {prior_verdict.convergence:.2f},
> dissent {prior_verdict.dissent_count} persona(s),
> flags {prior_verdict.flags or "none"}.
>
> What's the follow-up question?

This anchors the user in the panel's context. Without it, follow-up questions
drift into reframes more often than not.

## Coaching templates

**Vague follow-up ("ask them more"):**
> "Let's tighten that. The prior panel converged at 0.62 with 4 dissenters —
> what specifically do you want to press on? The dissent? A new sub-option?
> A consequence the prior frame didn't name?"

**Reframe disguised as follow-up:**
> "I'm reading that as a different decision than the prior frame. The prior
> panel was framed around `[X]`; you're now asking about `[Y]`. The personas
> realized for `[X]` won't necessarily inform `[Y]`. Do you want to extend the
> existing panel with a tighter question, or convene a fresh panel for `[Y]`?"

**Over-length:**
> "The follow-up is over the 280-char ceiling. Same constraint as the original
> frame (spec §1: `DECISION_TOO_LONG`, retry-unsafe). Cut the part that carries
> the least weight."

## Worked follow-ups

| Prior frame | ❌ Weak follow-up | ✅ Strong follow-up |
|---|---|---|
| "Choosing launch tier price ($49 or $79) for indie devs." | "more thoughts" | "Among the $79 dissenters: what would they pay if onboarding included a 30-day money-back guarantee?" |
| "Hiring senior vs junior IC for the platform team." | "what about staff?" | "Among the dissenters: what specific platform-team risk is the senior hire mitigating that the junior isn't?" |

## Headless mode

Same constraints (length, single-line) apply. Validate before passing to
Stage 3; loop back to interactive coaching on violation.

## Exit criteria

`follow_up_question` is a single-line UTF-8 string between 12 and 280 chars
after trim. The prior `decision_being_informed` remains inherited (not
re-supplied to the MCP tool).
