# Frame Coaching — Shaping `decision_being_informed`

Loaded by Quinn (`bmad-agent-facilitator`) at convene time and when running
the inline `FR` (frame coach) capability.

The `decision_being_informed` field is the most important sentence the user
writes when running a panel. It's the only thing that makes the verdict
legible six months later. Quinn's job is to coach the user to a frame that
survives that test.

---

## 1. Hard constraints (from spec §1)

- **String, 12–280 characters after trim.**
- **Single-line.** No `\n`, no `\r`. UTF-8.
- **Required** on `run_panel` / `run_quick_poll` / `extend_panel`.
- **NOT** sent on `run_prompt` (sub-decisional, no verdict).
- **Echoed verbatim** into `panel_verdict.json` at `meta.decision_being_informed`
  and stamped onto every transcript row.

The contract enforces these. Quinn coaches the user toward them BEFORE the
call so we never see `MISSING_DECISION` or `DECISION_TOO_LONG` from the
server. (Both errors are retry-unsafe — the user must re-frame, not retry.)

---

## 2. The four-part frame

A well-formed `decision_being_informed` answers, implicitly:

1. **What is being chosen?** (option A vs option B, or "what to do about X")
2. **For whom?** (audience, customer segment, internal stakeholder)
3. **What is the consequence?** (what changes if we pick wrong)
4. **What is the constraint?** (deadline, budget, reversibility — when relevant)

Aim for **1 and 2 always**; add 3 and 4 when they sharpen the question. You
will rarely fit all four in 280 chars.

---

## 3. Coaching templates (Quinn's inline prompts)

### When the user gives a vague intent

> "Let's tighten that into a frame the panel can answer. In one sentence
> under 280 chars: what's the choice, who is it for, and what changes if we
> pick wrong?"

### When the user gives a label, not a decision (<12 chars or topic-only)

> "That reads more like a topic than a decision. What's the *choice* you're
> informing with this panel? Try the form: 'choosing [X] for [audience],
> where the trade-off is [Y]'."

### When the user runs long (>280 chars)

> "The frame is over the 280-char ceiling. The contract rejects this without
> truncation (`DECISION_TOO_LONG`, retry-unsafe). Let's cut: which sentence
> carries the least weight if we drop it?"

### When the frame contains multiple decisions stacked

> "I'm reading two decisions in there — [A] and [B]. Which one is this panel
> informing? The other is its own panel run."

### When the user supplies whitespace-only or pure newlines

> "The contract treats whitespace-only as missing — that'll come back as
> `MISSING_DECISION`. What's the actual choice?"

---

## 4. Worked frames — weak vs strong

| ❌ Weak | ✅ Strong |
|---|---|
| "pricing" (4 chars, label) | "Choosing launch tier price ($49 or $79) for indie-developer cohort." (66 chars) |
| "should we hire the senior eng or the junior one and how do we onboard them and also what's the comp band" (>280) | "Hiring senior vs junior IC for the platform team — informs comp band and Q3 ramp timing." (89 chars) |
| "what name is best" (close to label) | "Choosing product name from shortlist of 4, optimizing for indie-builder recall." (79 chars) |
| "thoughts on the docs?" (sub-decisional) | (Not a panel call — this is `run_prompt` per spec §1.) |
| "$49 or $79\nfor indies" (multi-line) | "Choosing $49 vs $79 launch price for indie devs." (51 chars, single line) |

---

## 5. The FR (frame coach) inline capability

When the user invokes `FR` directly, Quinn:

1. **Reads the user's draft frame** (whatever they supply — could be a phrase,
   a paragraph, or just a topic).
2. **Diagnoses** which constraint or part of the four-part frame is missing
   or off:
   - too short → label, not decision
   - too long → over 280
   - multi-line → contract violation
   - vague → "what's the choice / for whom / consequence?"
   - stacked → split into multiple panels
3. **Proposes a tightened frame** — verbatim, with character count noted.
   Quinn's draft is a starting point, not the final answer; the user owns
   the final string.
4. **Returns the frame** without convening. FR is coaching-only — convening
   is a separate action (QP, RP, EP).

If the user then says "run it," Quinn passes the agreed frame to the
appropriate convene-* workflow at Stage 4.

### FR session shape (terse)

> User: "Frame this for a panel: pricing for the launch."
>
> Quinn: "Too short — that reads as a topic, not a decision. The four-part
> frame wants: what choice / for whom / what consequence. Try:
> *'Choosing launch tier price ($X vs $Y) for [your audience], where the
> trade-off is [easy-yes vs headroom / accessibility vs revenue / …]'.*
>
> Fill in the bracketed parts and we'll have a frame that's ~80–100 chars
> and answers what the panel needs."

---

## 6. Frame quality at readout time

The frame is part of the verdict. At readback, Quinn:

- Echoes the frame **verbatim** as the lede of the markdown card.
- Never paraphrases. If the frame was bad, the verdict is bad — that's a
  signal to re-run with a tighter frame, not to launder the lede.
- Stamps the frame on every transcript row alongside the persona quote, per
  spec §2 — so anyone reading the transcript six months later can reconstruct
  what the panel was actually deliberating.

If the frame was over 280 chars (which means the contract rejected it and
nothing ran), there is no verdict to read back — the FR coaching loop
restarts.

---

## 7. What FR does NOT do

- Convene a panel. FR is read-only and inline.
- Decide for the user what the right frame is. Quinn proposes; user owns.
- Modify the frame on a saved verdict. The frame in `meta.decision_being_informed`
  is immutable post-run; rewriting it would break the audit chain.
- Coach `follow_up_question` for `extend_panel` — that's a different field
  with different rules (load `references/triggers.md` and the EP workflow).
