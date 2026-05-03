# Stage 2 — Frame `decision_being_informed`

Coach the user to a frame the panel can answer. Canonical for QP and RP.

## Constraints (spec §1)

- **String, 12–280 characters after trim.**
- **Single line.** Newlines are rejected upstream — do not silently strip them;
  loop back to coaching if the user pasted a multi-line block.
- **UTF-8.**
- **Echoed verbatim** into `panel_verdict.json.meta.decision_being_informed`
  and stamped onto every transcript row. No paraphrase, no summarization.

## The four-part frame (DESIGN-2 §6)

A well-formed `decision_being_informed` answers, implicitly:

1. **What is being chosen?** (option A vs B, or "what to do about X")
2. **For whom?** (audience, customer segment, internal stakeholder)
3. **What is the consequence?** (what changes if we pick wrong)
4. **What is the constraint?** (deadline, budget, reversibility — when relevant)

Aim for parts 1–2 always; add 3 and 4 when they sharpen the question.

## Coaching templates

**Vague intent ("help me with pricing"):**
> "Let's tighten that into a frame the panel can answer. In one sentence under
> 280 chars: what's the choice, who is it for, and what changes if we pick wrong?"

**Label, not a decision (`<12` chars or topic-only):**
> "That reads more like a topic than a decision. What's the *choice* you're
> informing? Try the form: 'choosing [X] for [audience], where the trade-off is [Y]'."

**Over-length (`>280` chars):**
> "The frame is over the 280-char ceiling. The contract rejects this without
> truncation (spec §1: `DECISION_TOO_LONG`, retry-unsafe). Let's cut: which
> sentence carries the least weight if we drop it?"

**Stacked decisions (two questions in one frame):**
> "I'm reading two decisions in there — [A] and [B]. Which one is this panel
> informing? The other is its own panel run."

## Worked frames

| ❌ Weak | ✅ Strong |
|---|---|
| "pricing" | "Choosing launch tier price ($49 or $79) for indie-developer cohort." |
| "what name is best" | "Choosing product name from shortlist of 4, optimizing for indie-builder recall." |
| "thoughts on the docs?" | (Not a panel call — `run_prompt` territory.) |

## Headless mode

If the caller supplied a frame via direct invocation (Quinn routing or
`--headless`), validate it against the constraints above and proceed without
re-coaching. Frame still subject to length / single-line checks; on violation,
loop back to interactive coaching even from headless mode (better than
surfacing `MISSING_DECISION` from the server).

## Exit criteria

`decision_being_informed` is a single-line UTF-8 string between 12 and 280
chars after trim, and the user has confirmed (or headless: validation passed).
