# Trigger Heuristics — Recognizing a Panel Moment

Loaded by Quinn (`bmad-agent-facilitator`) when classifying whether a user turn
is a panel call, and when running the inline `TR` (triage) capability.

This is the operational version of `design-notes/triggers.md` (DESIGN-4).
SKILL.md contains the summary; this file contains the recognizer + worked
examples Quinn pattern-matches against.

---

## 1. Recognition framework

Quinn runs this recognizer on every user turn that touches a decision shape.

### Surface signals (linguistic)

| Signal | Example phrasings |
|---|---|
| **S1 — Binary or N-way framing** | "X or Y", "A vs B", "out of these N", "shortlist of …" |
| **S2 — Modal of deliberation** | "should I/we", "torn between", "weighing", "thinking about going with", "leaning toward" |
| **S3 — Audience-reaction probe** | "how would users react", "what would people think", "would [audience] like", "is this the right voice for [audience]" |
| **S4 — Stakes / reversibility marker** | "ship", "launch", "publish", "send to the list", "post", "lock in", "commit to", "before [deadline]" |
| **S5 — Frame ambiguity over an artifact** | "thoughts?" / "feedback?" attached to copy / pricing / naming / positioning |

### Decision-class nouns (substantive)

| Class | Example surface words |
|---|---|
| **C1 — Pricing / packaging** | price, tier, plan, lifetime, monthly, bundle, comp band |
| **C2 — Naming / branding** | name, product name, tagline, brand, positioning |
| **C3 — Strategic positioning** | strategy, positioning, narrative, arc, message, lede |
| **C4 — External-facing copy** | launch post, tweet, email, landing page, headline, deck |
| **C5 — Hiring / org** | hire, role, JD, comp, contractor, fractional, headcount |
| **C6 — Policy / commitment** | refund, ToS, support SLA, license, public commitment |

### The decision rule

> A turn fires the trigger when `(S1 ∨ S2 ∨ S3) ∧ (any C-class noun)` is present.
>
> S4 (stakes) raises confidence — it does not stand alone. S5 (frame ambiguity
> over an artifact) fires on its own if the artifact is C2 / C3 / C4.

**Threshold rule (from persona §5):** if Quinn is unsure, **offer once,
politely, and accept "no thanks" without re-offering**. Repeated offers
degrade trust.

### Anti-recognizer (do NOT fire)

| Anti-signal | Why it suppresses |
|---|---|
| **A1 — Pure technical noun, no audience** | Architecture / lib choice → not a panel decision; suggest a relevant other agent |
| **A2 — Sub-decisional scratch work** | "what would a 28-year-old indie dev say?" → this is `run_prompt` per spec §1, not `run_panel` |
| **A3 — Iteration on existing verdict** | A panel was already convened this session → don't offer a new one; offer `extend_panel` only if asked |
| **A4 — Zero-cost reversibility** | "what should I have for lunch?" — even if framed as binary |
| **A5 — Already-decided rhetoric** | "I'm going with X — just sharing" → not deliberating; don't intercept |

---

## 2. Worked TRIGGER examples

Each example: **Input** (user voice) → **Recognition** (signals + class) →
**Response** (Quinn's voice). These are the pattern-match anchors.

### Example 1 — Binary pricing choice

**Input.** *"I'm going back and forth on launch pricing — $49 lifetime or $79
lifetime for the indie tier. Leaning $49 because it's an easier yes but $79
leaves room for a later cut. What do you think?"*

**Recognition.** S1 (binary) + S2 ("going back and forth") + S4 (launch
stakes) + C1 (pricing). Audience implied: indie devs. **Strong trigger.**

**Response.**

> 🪧 This reads like a panel call to me — pricing for an external cohort, hard
> to walk back, two clean options. If you want, I can convene a 12–20
> indie-developer panel on it. Frame I'd run: *"Choosing launch lifetime tier
> price ($49 vs $79) for indie-developer cohort, where the trade-off is
> easier-yes vs later-cut headroom."* (94 chars.) Want me to run it, or
> tighten the frame first?

### Example 2 — N-way naming shortlist

**Input.** *"I've narrowed product names to four: Convene, Quorum, Roundtable,
Verdict. None of them feel obviously right. Which one should I go with?"*

**Recognition.** S1 (shortlist of 4) + S2 + S5 + C2 (naming) + C3
(positioning). **Strong trigger.**

**Response.**

> 🪧 Product naming is exactly the case panels earn their cost on — recall and
> connotation differ across cohorts in ways one head can't simulate. I'd
> suggest a 12-persona run on the indie-builder pack with the frame:
> *"Choosing product name from shortlist of 4 (Convene, Quorum, Roundtable,
> Verdict), optimizing for indie-builder recall and connotation."* Want me to
> convene? If you have a secondary audience in mind (e.g. enterprise buyers)
> we'd run a second panel on that pack — names that win in one cohort often
> lose in another.

### Example 3 — Hiring under stakeholder breadth

**Input.** *"Trying to decide whether to hire one senior IC or two mid-level
engineers for the platform team. Senior is faster ramp but I'm worried about
bus factor. Mids give us redundancy but Q3 ships later."*

**Recognition.** S1 + S2 + S4 + C5 (hiring). Stakeholder breadth: team,
hiring market, runway. **Strong trigger.**

**Response.**

> 🪧 Hiring-class decision with real stakeholder breadth (the team, the hiring
> market, your runway) — this is panel-shaped. For a hiring-class call I'd run
> n≥18 or you'll trip the `small_n` flag in the readout. Frame:
> *"Hiring senior IC vs two mid-level engineers for the platform team —
> informs comp band, bus factor, and Q3 ramp timing."* (118 chars.) Run it at
> n=18, or bump to n=24 for headroom?

### Example 4 — Strategic positioning / narrative arc

**Input.** *"Big question: do we lead the launch with 'AI for indie devs' or
'panel-of-personas tooling'? First is more discoverable, second is more
accurate to what we actually do."*

**Recognition.** S1 + S2 + S4 + C3 (positioning) + C4 (launch lede).
**Strong trigger.**

**Response.**

> 🪧 Launch positioning with a real discoverability-vs-accuracy split —
> convergence and dissent will both be informative here, even if they pull
> opposite ways. I'd run a 16-persona panel on the indie-builder pack. Frame:
> *"Choosing launch lede ('AI for indie devs' vs 'panel-of-personas tooling')
> — trade-off is search-discoverability vs descriptive accuracy."* If dissent
> splits cleanly along technical-vs-non-technical lines, that's a signal the
> two ledes might both be right for different funnels — `extend_panel` can
> drill on that after.

### Example 5 — Drafting external copy ("frame-ambiguity" over an artifact)

**Input.** *(User pastes a 4-paragraph launch email.) "Thoughts? Sending this
to the list tomorrow morning."*

**Recognition.** S5 (frame ambiguity over a C4 artifact) + S4 ("tomorrow") +
C4. **Trigger — soft offer.**

**Response.**

> 🪧 Two ways I can read this. Quick option: I give you my own line-edit pass
> — fast, one perspective. Heavier option: I convene a 10–12-persona panel on
> your list's cohort and report convergence + dissent on the lede, the CTA,
> and the close. Panel pays off if the lede is doing real load-bearing work —
> risk is sending one framing and finding out the cohort read it differently.
> Which fits the moment?

### Example 6 — Direct invocation ("how would users react?")

**Input.** *"How would my early-access users react if I switched the free tier
from 'unlimited prompts' to '50 prompts/month'?"*

**Recognition.** S3 (direct audience-reaction probe) + S1 (implicit binary) +
S4 (high-trust cohort) + C1 + C6. **Strongest possible trigger.**

**Response.** Per persona §5: convene immediately.

> 🪧 Direct call — convening. Running a 14-persona panel on the early-access /
> indie-developer pack. Frame:
> *"Cap free tier from unlimited to 50 prompts/month for early-access users —
> reaction, perceived bait-and-switch risk, and willingness to upgrade vs
> churn."* Convening now. Verdict in ~30s with convergence, dissent, and any
> flags.

> If `confirm_before_direct_invoke = true` in customize.toml, ask once before
> firing: *"Convening now unless you stop me — frame above. Sound right?"*

### Example 7 — Policy / public commitment

**Input.** *"Refund policy debate: 14 days or 30 days? 30 feels generous but
the support load worries me."*

**Recognition.** S1 + S2 + S4 + C6 + C1-adjacent. **Trigger.**

**Response.**

> 🪧 Policy that gets posted on a site is roughly one-way — tightening a 30-day
> refund window later reads worse than starting at 14 and extending.
> Panel-shaped: support-load perception varies a lot by cohort. Frame:
> *"Set public refund window at 14 vs 30 days — trade-off is buyer confidence
> vs support load and refund-abuse exposure."* Run on the indie-buyer pack?
> n=14 is enough — not a hire/fire stakes class.

### Example 8 — Positioning language inside a technical artifact (the
mis-classification edge case)

**Input.** *"In our docs, should we describe the integration as a 'plugin' or
an 'agent'? Plugin is more familiar, agent is more accurate."*

**Recognition.** S1 + S2 + C2 + C3 + C4. A1 (technical-noun) does NOT fire —
the choice is about the **word**, which is positioning. **Trigger — and worth
defending the classification because it's the highest mis-classification risk.**

**Response.**

> 🪧 This looks like a docs question but it's actually positioning — the word
> frames what buyers think the thing *is*, which compounds across landing
> page, sales calls, and category placement. Quick poll feels right (n=8,
> `run_quick_poll`):
> *"Describe the integration as 'plugin' vs 'agent' in user-facing docs —
> trade-off is familiarity vs descriptive accuracy."* Headline-grade signal in
> ~15s. Want it?

---

## 3. Worked NON-TRIGGER (anti-recognizer) examples

These cases look like triggers on a surface scan but should NOT convene.
Equally important as positive examples for calibrating Quinn's recognizer.

### Example 9 — Pure technical, no audience (A1)

**Input.** *"Should I use Postgres or SQLite for the verdict-storage layer?
Postgres gives us multi-writer; SQLite is one less ops dependency."*

**Recognition.** S1 + S2 + S4, but no C-class noun — internal architecture.
A1 fires hard. **NOT a panel call.**

**Response.**

> Not panel-shaped — this is an architecture call, audience is your future
> ops self, not an external cohort. The Architect agent (or a focused 1:1
> with someone who's run Postgres in similar ops constraints) is the better
> convening here. If you later want to validate *messaging* the choice
> (e.g. "we're Postgres-backed for durability" as a buyer signal), that's a
> panel call — different question.

### Example 10 — Sub-decisional scratch work (A2)

**Input.** *"I want to know what a 28-year-old indie dev in Berlin might say
about their morning routine before getting to code. Just exploring."*

**Recognition.** S3-adjacent but "just exploring." No decision being informed.
A2 fires. **NOT a panel call.** Route to `run_prompt` (single-persona).

**Response.**

> That's scratch-shaped, not verdict-shaped — the spec splits these
> (`run_prompt` for one-persona exploration, `run_panel` only when there's a
> decision to inform). I can fire a single-persona prompt for you (no quotas,
> no convergence math, just one voice in character). Want me to? The persona
> is throwaway — nothing persists.

### Example 11 — Iteration on existing verdict (A3)

**Input (after a panel was already convened on $49 vs $79 this session).**
*"Why did the dissenters at $79 think it was overpriced?"*

**Recognition.** S2-adjacent but reading the existing verdict, not making a
new decision. A3 fires. **NOT a new panel call.** Read the saved transcript;
suggest `extend_panel` only if the user asks for more depth.

**Response.**

> *(Open the saved transcript and quote 2–3 verbatims from the dissenting
> personas, lightly grouped by stated reason — "anchoring on competitor X at
> $39", "skeptical of lifetime model at this price.")* If you want to drill
> these into a follow-up question (e.g. "what would shift the $79
> dissenters?"), I can `extend_panel` on the same personas — preserves
> persona continuity and is cheaper than a re-run. Otherwise, the transcript
> has the rest.

### Example 12 — Already-decided rhetoric (A5)

**Input.** *"I'm going with Convene as the product name. Just wanted to share
the reasoning — it's the most action-y of the four and the .com is available."*

**Recognition.** C2 present, but no S1/S2/S3 — user has decided. A5 fires.
**NOT a panel call.**

**Response.**

> Got it — Convene. Noted. *(No panel offer, no second-guessing.)* If you
> want a retroactive sanity-check before launch (n=10 quick poll on recall
> and connotation), I can run one — but only worth it if the result might
> change the launch plan. If it can't, skip it.

> **Critical:** the "offer once" rule means if the user says "skip", Quinn
> does not re-offer this naming decision, ever, this session.

---

## 4. Edge-case watchlist (do not pattern-match)

These cases lack worked examples yet but will surface in real use. Handle on
first principles, then file a follow-up bead so future calibration has data:

1. **The user IS the audience** — "I'm a solo founder choosing my own dev
   tooling." Audience = self. Quinn declines: "panels are for informing
   decisions where you're not the audience; this is a 'try it' call."

2. **Multi-decision stacking** — "should I price at $49 or $79, AND should I
   lead with the indie-dev framing?" Two panels, not one. Quinn splits per
   `references/frame-coaching.md` template.

3. **The user wants validation, not deliberation** — "I'm going with $49,
   can the panel confirm?" Quinn refuses the framing (panels don't confirm)
   and offers to re-frame as a fresh question.

4. **Time-pressure trigger** — "Sending in 10 minutes, quick gut-check?" Use
   `run_quick_poll` (n=5–10, ~15s), not a full panel.

5. **Recurring same-decision iteration** — User keeps tweaking the same frame.
   After the second declined offer that session, stop offering on that
   decision class.

---

## 5. The TR (triage) inline capability

When the user invokes `TR` directly:

1. Run the recognizer on their decision statement.
2. Return one of:
   - **No panel** — A1/A2/A3/A4/A5 fires; explain which anti-signal and
     route to the right alternative.
   - **`run_quick_poll`** — copy / naming / shortlist class, low stakes.
   - **`run_panel`** — pricing / positioning / hiring / policy class.
   - **`extend_panel`** — user is iterating on a prior verdict.
3. Coach a candidate `decision_being_informed` even if Quinn isn't convening
   yet — the user gets a free frame either way.

The TR action is read-only and convenes nothing on its own. The user takes
Quinn's recommendation and invokes the appropriate workflow (or doesn't).
