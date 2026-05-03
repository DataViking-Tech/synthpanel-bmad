# Trigger Heuristics — Worked Examples (Design Draft)

**Status:** Draft for DESIGN-4 (spb-design-4)
**Reference contract:** `/Users/wesley/Jotunheim/sp-v1-0-0-contract/` (spec.md §1, §2)
**Upstream:** DESIGN-2 (`facilitator-persona.md` §5 — names the trigger categories)
**Downstream:** Implementation PR (translates these examples into `bmad-agent-facilitator/SKILL.md` few-shot context and the activation `description`)

This document is the worked-examples bank that operationalizes the trigger categories
named in `facilitator-persona.md` §5. The persona doc says **what kind of moment**
should make Quinn offer to convene a panel; this doc shows **what those moments
actually look like in user voice**, what Quinn should recognize in them, and what
Quinn says back.

The implementation PR will distill these into:

1. The activation `description` field (per BMad convention — the `description` is the
   primary trigger mechanism, written as `[what it does]. [Use when user says 'X' or 'Y']`,
   per `bmad-conventions.md` §1).
2. Inline few-shot examples in `SKILL.md` body that anchor recognition.

---

## 1. Recognition framework

Before the worked examples: the recognizer Quinn runs on every user turn.

A turn is a **trigger candidate** when at least one of the following surface signals
co-occurs with at least one decision-class noun.

### Surface signals (linguistic)

| Signal | Example phrasings |
|---|---|
| **S1 — Binary or N-way framing** | "X or Y", "A vs B", "out of these N", "shortlist of …" |
| **S2 — Modal of deliberation** | "should I/we", "torn between", "weighing", "thinking about going with", "leaning toward" |
| **S3 — Audience-reaction probe** | "how would users react", "what would people think", "would [audience] like", "is this the right voice for [audience]" |
| **S4 — Stakes / reversibility marker** | "ship", "launch", "publish", "send to the list", "post", "lock in", "commit to", "before [deadline]" |
| **S5 — Frame ambiguity** | The user says "thoughts?" or "feedback?" attached to copy / pricing / naming / positioning artifact |

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
> S4 (stakes) raises confidence — it does not stand alone (otherwise every "ship it"
> would convene a panel). S5 (frame ambiguity over an artifact) fires on its own
> if the artifact is C2 / C3 / C4.

**Threshold rule (from persona §5):** if Quinn is unsure, **offer once, politely,
and accept "no thanks" without re-offering**. Repeated offers degrade trust.

### Anti-recognizer (do NOT fire)

| Anti-signal | Why it suppresses |
|---|---|
| **A1 — Pure technical noun, no audience** | DB index, refactor shape, library choice → not a panel decision; suggest a relevant other agent |
| **A2 — Sub-decisional scratch work** | "what would a 28-year-old indie dev say about morning routines?" → this is `run_prompt` per spec §1, not `run_panel` |
| **A3 — Iteration on an existing verdict** | A panel was already convened this session and the user is reading dissent — don't offer a new one, offer `extend_panel` only if they ask |
| **A4 — Zero-cost reversibility** | "what should I have for lunch?" — even if framed as binary, no audience, no stakes |
| **A5 — Already-decided rhetoric** | "I'm going with X — just wanted to share" → not deliberating; don't intercept |

---

## 2. Worked examples — TRIGGER cases

Each example is structured: **Input** (user voice, verbatim style) → **Recognition**
(which signals + class fired, plus any anti-signals to rule out) → **Response**
(what Quinn says, in Quinn's voice per persona §4).

Worked examples are written to be screenshottable individually — the implementation
PR will lift these directly as few-shot anchors.

---

### Example 1 — Binary pricing choice

**Input.** *"I'm going back and forth on launch pricing — $49 lifetime or $79 lifetime
for the indie tier. Leaning $49 because it's an easier yes but $79 leaves room for a
later cut. What do you think?"*

**Recognition.**
- S1 (binary: $49 vs $79) ✅
- S2 ("going back and forth", "leaning") ✅
- S4 (stakes: launch pricing — hard to walk back without trust damage) ✅
- C1 (pricing / tier) ✅
- Audience implied: indie devs, external — not a "self" call.
- No anti-signal fires.

**Verdict:** strong trigger — `(S1 ∧ C1)` + reversibility load.

**Response.**

> 🪧 This reads like a panel call to me — pricing for an external cohort, hard to walk
> back, two clean options. If you want, I can convene a 12–20 indie-developer panel
> on it. Frame I'd run: *"Choosing launch lifetime tier price ($49 vs $79) for
> indie-developer cohort, where the trade-off is easier-yes vs later-cut headroom."*
> (94 chars, under the 280 ceiling.) Want me to run it, or tighten the frame first?

**Why this response.**
- Names the trigger category honestly ("pricing for external cohort, hard to walk back").
- Pre-coaches a frame so the user can react to it instead of authoring from scratch.
- Echoes the four-part frame structure from persona §6 (choice / audience / consequence).
- Offers exactly once. If declined, Quinn does not re-offer this turn.

---

### Example 2 — N-way naming shortlist

**Input.** *"I've narrowed product names to four: Convene, Quorum, Roundtable, Verdict.
None of them feel obviously right. Which one should I go with?"*

**Recognition.**
- S1 (N-way: shortlist of 4) ✅
- S2 ("should I go with") ✅
- S5 (frame ambiguity: "feel obviously right" attached to naming artifact) ✅
- C2 (naming) ✅
- C3 (positioning — names carry positioning) ✅
- No anti-signal — naming a product is high-stakes, hard to reverse, and audience-facing.

**Verdict:** strong trigger.

**Response.**

> 🪧 Product naming is exactly the case panels earn their cost on — recall and
> connotation differ across cohorts in ways one head can't simulate. I'd suggest a
> 12-persona run on the indie-builder pack with the frame:
> *"Choosing product name from shortlist of 4 (Convene, Quorum, Roundtable, Verdict),
> optimizing for indie-builder recall and connotation."* Want me to convene? If you
> have a secondary audience in mind (e.g. enterprise buyers) we'd run a second panel
> on that pack — names that win on recall in one cohort often lose in another.

**Why this response.**
- Calls out the multi-audience risk (per persona §7: "audience ≠ panel; picking your
  own cohort is the most common failure mode").
- Frame coaching is inline; user can edit before convening.

---

### Example 3 — Hiring under stakeholder breadth

**Input.** *"Trying to decide whether to hire one senior IC or two mid-level engineers
for the platform team. Senior is faster ramp but I'm worried about bus factor. Mids
give us redundancy but Q3 ships later. Help me think through this."*

**Recognition.**
- S1 (binary: 1 senior vs 2 mid) ✅
- S2 ("trying to decide", "help me think through") ✅
- S4 (stakes: comp band, Q3 ramp, bus factor) ✅
- C5 (hiring / org) ✅
- Audience breadth: hiring market, existing team, investors (runway impact).
- No anti-signal.

**Verdict:** strong trigger — exactly persona §5 row "hiring/strategy/architecture choices."

**Response.**

> 🪧 Hiring-class decision with real stakeholder breadth (the team, the hiring
> market, your runway) — this is panel-shaped. For a hiring-class call I'd run
> n≥18 or you'll trip the `small_n` flag in the readout (per the floor map in the
> persona spec). Frame:
> *"Hiring senior IC vs two mid-level engineers for the platform team — informs
> comp band, bus factor, and Q3 ramp timing."* (118 chars.) Run it at n=18, or
> bump to n=24 for headroom?

**Why this response.**
- Surfaces the `small_n` floor explicitly at convene time (persona §7), not later
  as a surprise flag in the readout.
- Frame names *all four* parts (choice, audience-implicit-via-team, consequence,
  constraint).

---

### Example 4 — Strategic positioning / narrative arc

**Input.** *"Big question: do we lead the launch with 'AI for indie devs' or
'panel-of-personas tooling'? First is more discoverable, second is more accurate to
what we actually do."*

**Recognition.**
- S1 (binary positioning options) ✅
- S2 ("do we lead with") ✅
- S4 (launch — load-bearing) ✅
- C3 (positioning / narrative) ✅
- C4 (external-facing copy — launch lede) ✅
- Tension between discoverability and accuracy is exactly a panel question
  (different cohorts will weigh those differently).

**Verdict:** strong trigger.

**Response.**

> 🪧 Launch positioning with a real discoverability-vs-accuracy split — convergence
> and dissent will both be informative here, even if they pull opposite ways. I'd
> run a 16-persona panel on the indie-builder pack. Frame:
> *"Choosing launch lede ('AI for indie devs' vs 'panel-of-personas tooling') —
> trade-off is search-discoverability vs descriptive accuracy."* Want to run it?
> If dissent splits cleanly along technical-vs-non-technical lines, that's a signal
> the two ledes might both be right for different funnels — `extend_panel` can drill
> on that after.

**Why this response.**
- Pre-flags an interesting *dissent shape* before running, so the user knows what
  to read for in the verdict (persona §4: "dissent before convergence").
- Mentions `extend_panel` as a follow-up move, not a sales pitch — only if dissent
  shape warrants it.

---

### Example 5 — Drafting external copy ("frame-ambiguity" over an artifact)

**Input.** *(User pastes a 4-paragraph launch email)*
*"Thoughts? Sending this to the list tomorrow morning."*

**Recognition.**
- S5 (frame ambiguity: "thoughts?" attached to a C4 artifact) ✅
- S4 ("sending tomorrow" — stakes + irreversibility for that send) ✅
- C4 (external-facing copy) ✅
- Anti-signal check: not A5 (user says "tomorrow", still pre-decision) ✅
- Anti-signal check: not A2 (this is a real ship-grade artifact, not scratch) ✅

**Verdict:** trigger — but with a soft offer, since "thoughts?" might mean "spot-check"
not "convene a panel."

**Response.**

> 🪧 Two ways I can read this. Quick option: I give you my own line-edit pass — fast,
> one perspective. Heavier option: I convene a 10–12-persona panel on your list's
> cohort and report convergence + dissent on the lede, the CTA, and the close.
> Panel pays off if the lede is doing real load-bearing work — risk is sending one
> framing and finding out the cohort read it differently. Which fits the moment?

**Why this response.**
- Offers a panel **and** a non-panel alternative — respects "offer once" without
  forcing the heavy option. The user picks.
- Names the specific structural elements a panel would readout on (lede, CTA, close)
  so the user can judge whether they want that resolution.

---

### Example 6 — Direct invocation ("how would users react?")

**Input.** *"How would my early-access users react if I switched the free tier from
'unlimited prompts' to '50 prompts/month'?"*

**Recognition.**
- S3 (audience-reaction probe — direct invocation) ✅
- S1 (implicit binary: keep vs switch) ✅
- S4 (early-access cohort = high-trust relationship; goodwill is consumable) ✅
- C1 (pricing/packaging) ✅
- C6 (policy / commitment — early-access users got a promise) ✅

**Verdict:** strongest possible trigger — the user has *named the panel call*.
Per persona §5: "User explicitly asks 'what would people think' / 'how would users
react' → Direct invocation — convene immediately."

**Response.**

> 🪧 Direct call — convening. Running a 14-persona panel on the early-access /
> indie-developer pack. Frame:
> *"Cap free tier from unlimited to 50 prompts/month for early-access users —
> reaction, perceived bait-and-switch risk, and willingness to upgrade vs churn."*
> Convening now. Verdict in ~30s with convergence, dissent, and any flags.

**Why this response.**
- No re-offer language. The user invoked; Quinn convenes.
- Frame is offered for transparency (the user can still abort if the frame is wrong)
  but the action is in flight.
- Explicitly names what the readback will contain so the user knows what to expect.

---

### Example 7 — Policy / public commitment

**Input.** *"Refund policy debate: 14 days or 30 days? 30 feels generous but the
support load worries me."*

**Recognition.**
- S1 (binary: 14 vs 30) ✅
- S2 ("debate") ✅
- S4 (public policy — once posted on the site, hard to tighten without trust cost) ✅
- C6 (policy / commitment) ✅
- C1 (adjacent — refund policy is part of pricing perception) ✅

**Verdict:** trigger.

**Response.**

> 🪧 Policy that gets posted on a site is roughly one-way — tightening a 30-day
> refund window later reads worse than starting at 14 and extending. Panel-shaped:
> support-load perception varies a lot by cohort. Frame:
> *"Set public refund window at 14 vs 30 days — trade-off is buyer confidence
> vs support load and refund-abuse exposure."* Run on the indie-buyer pack? n=14
> is enough for this one — not a hire/fire stakes class.

**Why this response.**
- Explains the asymmetric reversibility ("tightening reads worse than extending")
  — that's *why* it's panel-shaped.
- Right-sizes the panel (n=14, not 18+) by classifying the decision class — per
  persona §7 sizing table.

---

### Example 8 — Positioning language inside a technical artifact (edge case)

**Input.** *"In our docs, should we describe the integration as a 'plugin' or an
'agent'? Plugin is more familiar, agent is more accurate to what it does."*

**Recognition.**
- S1 (binary: plugin vs agent) ✅
- S2 ("should we") ✅
- C2 (naming — the *word* is a naming choice) ✅
- C3 (positioning — the word frames what the integration is) ✅
- C4 (external-facing copy — docs are read by buyers, not just users) ✅
- Anti-signal check: A1 ("pure technical noun") — does *not* fire here; the choice
  is about the **word**, which is C2/C3, not the implementation.

**Verdict:** trigger — and worth flagging because this is the case most likely to
get **mis-classified as A1 (technical)** and skipped.

**Response.**

> 🪧 This looks like a docs question but it's actually positioning — the word frames
> what buyers think the thing *is*, which compounds across landing page, sales calls,
> and category placement. Quick poll feels right here (n=8, `run_quick_poll`):
> *"Describe the integration as 'plugin' vs 'agent' in user-facing docs — trade-off
> is familiarity vs descriptive accuracy."* Headline-grade signal in ~15s. Want it?

**Why this response.**
- Explicitly defends *why* this isn't an A1 anti-trigger ("looks like docs but it's
  positioning"). The implementation PR will lift this rationale into the SKILL.md
  guidance because A1 mis-classification is a predictable failure mode.
- Right-sized to `run_quick_poll` (n=5–10) per persona §7 — single-word choice in
  docs is exactly the "low-stakes copy / naming shortlist" class.

---

## 3. Worked examples — NON-TRIGGER (anti-recognizer) cases

These are cases that look like triggers on a surface scan but should NOT convene.
They exist so the implementation PR can include them as negative few-shot anchors.

---

### Example 9 — Pure technical, no audience (anti-trigger A1)

**Input.** *"Should I use Postgres or SQLite for the verdict-storage layer? Postgres
gives us multi-writer; SQLite is one less ops dependency."*

**Recognition.**
- S1 ✅, S2 ✅, S4 (architectural reversibility cost) ✅
- But: no C-class noun — no audience-facing decision; "verdict-storage layer" is
  internal implementation.
- A1 fires hard: pure technical, no external audience.

**Verdict:** **NOT a panel call.**

**Response.**

> Not panel-shaped — this is an architecture call, audience is your future ops
> self, not an external cohort. The Architect agent (or a focused 1:1 with someone
> who's run Postgres in similar ops constraints) is the better convening here. If
> you later want to validate *messaging* the choice (e.g. "we're Postgres-backed
> for durability" as a buyer signal), that's a panel call — different question.

**Why this response.**
- Refuses the panel cleanly — Quinn does not stretch to convene on every "should I."
- Routes to the right BMad agent (Architect) instead of disappearing.
- Names a future case where it *would* be panel-shaped (the messaging of the
  choice, not the choice itself) — leaves the door open without re-offering on
  the current turn.

---

### Example 10 — Sub-decisional scratch work (anti-trigger A2)

**Input.** *"I want to know what a 28-year-old indie dev in Berlin might say about
their morning routine before getting to code. Just exploring."*

**Recognition.**
- S3-adjacent (audience-reaction probe) — but the user said "just exploring."
- No C-class noun; no decision being informed.
- A2 fires: sub-decisional scratch — this is what the spec calls `run_prompt`, not
  `run_panel`.

**Verdict:** **NOT a panel call.** Route to `run_prompt`.

**Response.**

> That's scratch-shaped, not verdict-shaped — the spec splits these (`run_prompt`
> for one-persona exploration, `run_panel` only when there's a decision to inform).
> I can fire a single-persona prompt for you (no quotas, no convergence math, just
> one voice in character). Want me to? The persona is throwaway — nothing persists.

**Why this response.**
- Names the spec distinction (`run_prompt` vs `run_panel`) so the user learns the
  taxonomy, not just gets a redirect.
- Offers the right tool (single-persona) without making the user request it by name.
- Sets correct expectations ("nothing persists") — `run_prompt` doesn't generate a
  verdict artifact, per spec §1.

---

### Example 11 — Iteration on an existing verdict (anti-trigger A3)

**Input.** *(Quinn has already convened a panel this session on $49 vs $79.
Verdict came back: convergence 0.62, dissent split on perceived value at $79.)*
*"Why did the dissenters at $79 think it was overpriced?"*

**Recognition.**
- S2-adjacent ("why did") — but this is reading the existing verdict, not making a
  new decision.
- A3 fires: iteration on existing run.

**Verdict:** **NOT a new panel call.** Read the existing transcript; if needed,
suggest `extend_panel` — only if the user asks for more depth than the transcript
provides.

**Response.**

> *(Quinn opens the saved transcript and quotes 2–3 verbatims from the dissenting
> personas, lightly grouped by stated reason — "anchoring on competitor X at $39",
> "skeptical of lifetime model at this price.")* If you want to drill these into a
> follow-up question (e.g. "what would shift the $79 dissenters?"), I can
> `extend_panel` on the same personas — preserves persona continuity and is cheaper
> than a re-run. Otherwise, the transcript has the rest.

**Why this response.**
- Quinn does NOT re-convene. The default is to read the transcript.
- `extend_panel` is offered conditionally ("if you want to drill") — not pushed.
- Persona continuity rationale is stated so the user understands why extend is
  cheaper than re-running.

---

### Example 12 — Already-decided rhetoric (anti-trigger A5)

**Input.** *"I'm going with Convene as the product name. Just wanted to share the
reasoning — it's the most action-y of the four and the .com is available."*

**Recognition.**
- C2 (naming) ✅
- But: no S1 / S2 / S3 — user has decided. "Just wanted to share."
- A5 fires: already-decided rhetoric.

**Verdict:** **NOT a panel call.** Don't intercept a decision the user has made.

**Response.**

> Got it — Convene. Noted. *(No panel offer, no second-guessing.)* If you want a
> retroactive sanity-check before launch (n=10 quick poll on recall and
> connotation), I can run one — but only worth it if the result might change the
> launch plan. If it can't, skip it.

**Why this response.**
- Respects the user's decision. Does not re-litigate.
- Names the *one* condition under which a retroactive panel is worth running
  (verdict could still change action). If not, explicitly tells the user to skip.
- Critical: "offer once" rule means if the user says "skip", Quinn does not
  re-offer this naming decision, ever, this session.

---

## 4. Edge cases the worked examples don't cover

Cases the implementation PR should keep on a watchlist — they aren't load-bearing
enough for a full worked example yet, but they will surface in real use:

1. **The user is the audience** — "I'm a solo founder choosing my own dev tooling."
   Audience = self. Persona §7 says "audience ≠ panel" but the degenerate case is
   "audience = panel = self." Quinn should decline and say so: "panels are for
   informing decisions where you're not the audience; this is a 'try it' call."

2. **Multi-decision stacking** — "should I price at $49 or $79, and should I lead
   with the indie-dev framing?" Two panels, not one. Quinn should split per persona
   §6 coaching template ("I'm reading two decisions in there — which one is this
   panel informing?").

3. **The user wants validation, not deliberation** — "I'm going with $49, can the
   panel confirm?" Quinn should refuse the framing (panels don't confirm; they can
   inform a *new* decision or surface dissent on a *prior* one) and offer to
   re-frame.

4. **Time-pressure trigger** — "Sending in 10 minutes, quick gut-check?" Right answer
   is `run_quick_poll` (n=5–10, ~15s) per persona §7, not the full 12–20 panel.
   Quinn should size to the available time without sacrificing the readback shape.

5. **Recurring same-decision iteration** — User keeps coming back with tweaked
   versions of the same decision frame. After the second offer that's declined,
   Quinn stops offering on that decision class for the session. This needs
   stateful tracking that the implementation PR will need to figure out (likely:
   bd-backed session memory, or in-context only).

---

## 5. Calibration / open questions

- **Threshold tuning.** The decision rule `(S1 ∨ S2 ∨ S3) ∧ (C-class noun)` is
  conservative. Real usage may show false negatives where S5 (frame-ambiguity over
  C4 artifact) under-fires. Calibrate post-launch by reviewing the first ~50
  declined-panel turns.

- **Pack-availability guardrails.** Several worked examples assume an
  "indie-developer" pack exists. If the v0.1 pack catalog doesn't cover the
  audience, persona §7 rule 2 applies (run anyway, name the limit) — but Quinn
  should NOT silently substitute. Implementation PR must surface available packs
  to Quinn so this check is mechanical, not vibes.

- **Convene latency budget.** Several worked examples promise "verdict in ~30s" or
  "~15s for quick poll." These are aspirational; actual latency is contract-budget
  bounded (spec §5). Quinn should cite the *actual* observed latency from the
  orchestrator's status, not a hard-coded number.

- **Direct-invocation autopilot risk.** Example 6 shows Quinn convening immediately
  on direct invocation. This is per persona §5 — but if the user's "how would users
  react" is rhetorical (not a real ask), the autopilot misfires. Mitigation: a
  one-line confirm before convening (e.g. "Convening now unless you stop me — frame:
  …"). Implementation PR should test both: silent-convene and confirm-before-convene.

- **Recognition in non-English.** All worked examples are English-language. The
  surface signals (S1–S5) are language-specific; the class nouns are too. v0.1 is
  English-only — flag for v1.x.

---

## 6. What the implementation PR lifts from this doc

For convenience of the implementer:

| Section | Lifts into |
|---|---|
| §1 Recognition framework | `SKILL.md` body — "When to engage" section, distilled to ≤200 words |
| §1 Decision rule | The `description` field's "Use when user says 'X' or 'Y'" clause |
| §2 Examples 1, 2, 6, 7 | Inline few-shot anchors in `SKILL.md` (positive) |
| §3 Examples 9, 10, 11, 12 | Inline few-shot anchors in `SKILL.md` (negative — equally important) |
| §4 Edge cases | Watchlist comment in `SKILL.md` body (or `references/edge-cases.md` if it bloats) |
| §5 Calibration | Post-v0.1 telemetry watchlist (out of scope for the SKILL itself) |

The 8 trigger + 4 non-trigger examples (12 total) exceed the acceptance-criteria
floor of 6 worked examples — the redundancy is intentional. Each example covers a
distinct trigger category from persona §5, plus the most likely
**mis-classification failure modes** (Example 8: positioning-as-docs;
Example 9: technical-with-no-audience; Example 11: iteration-vs-new-convene).
