# Workflow Scoping — v0.1

**Issue:** spb-design-3
**Status:** Decided (design)
**Date:** 2026-05-03
**Upstream:** DESIGN-1 (`bmad-conventions.md`), DESIGN-2 (`facilitator-persona.md`)
**Downstream:** Implementation PR (skill scaffolds), DESIGN-4 (trigger examples)
**Reference contract:** `/sp-v1-0-0-contract/spec.md`

---

## 1. Decision summary

The v0.1 pack ships **three panel-running workflows + one supporting workflow**,
each implemented as a BMad workflow skill (per the `samples/bmad-excalidraw/`
stages-table idiom — DESIGN-1 §5):

| Workflow skill | Menu code | Wraps MCP tool | Purpose |
|---|---|---|---|
| `convene-quick-poll` | `QP` | `run_quick_poll` | Fast 5–10-persona signal on a low-stakes choice |
| `convene-panel` | `RP` | `run_panel` | Standard 12–20-persona structured deliberation |
| `extend-panel` | `EP` | `extend_panel` | Follow-up on an existing run; persona continuity preserved |
| `render-readback` | `RB` | (none — local) | Re-render a saved `panel_verdict.json` as the markdown card |

**Rejected for v0.1** (recorded so we don't relitigate):

- `frame-coach` standalone workflow (`FR` in DESIGN-2 §10) — folded into
  `convene-*` Stage 2 instead of a separate skill. Rationale: 90% of
  frame-coaching happens in the lead-up to a panel call; making it a separate
  workflow invites users to over-coach without ever convening.
- A `run_prompt` wrapper — sub-decisional scratch work (spec §1) is not what
  Quinn convenes for; it's a different agent surface and out of scope for the
  Facilitator pack.
- `compare-verdicts` — file as a v0.2 follow-up bead. Useful but not on the
  critical path; needs two saved verdicts to operate, which is rare in v0.1
  usage.

### Why three panel-running workflows, not one parameterized workflow

It is tempting to ship a single `convene` workflow that takes panel size as a
parameter. We reject this for v0.1 because:

1. **The decision class drives the readout.** A quick-poll readout names its
   limits ("not for ship-blocking decisions"); a full panel readout doesn't.
   Branching on size inside one workflow means branching on size inside the
   readback template — fragile.
2. **The trigger heuristics differ.** Quinn offers a quick-poll on copy /
   naming shortlists; offers a full panel on pricing / hiring / positioning
   (DESIGN-2 §5, §7). Distinct workflows give distinct trigger surfaces in
   `module-help.csv`.
3. **`extend_panel` is structurally different** — it takes a prior
   `full_transcript_uri`, not a fresh `decision_being_informed` from scratch.
   Forcing it into the same shape buys nothing.

`module-help.csv` rows are cheap; one row per decision class keeps the menu
self-documenting.

---

## 2. Cross-cutting shape

All three panel-running workflows share the same eight-stage spine. Differences
between them are confined to **Stage 3 (size/pack defaults)**, **Stage 4 (the
MCP tool name)**, and **Stage 7 (readback voice)**.

Numbering matches the BMad excalidraw stages-table convention (DESIGN-1 §5):
each stage has a one-line purpose and a `references/<stage>.md` deep-prompt
target. SKILL.md stays scannable; depth lives in `references/`.

```
0 → preflight        # MCP tools reachable?           (DESIGN-5)
1 → trigger-check    # is this actually a panel call? (DESIGN-2 §5)
2 → frame            # coach decision_being_informed  (DESIGN-2 §6)
3 → assemble         # pick size + pack + quotas      (DESIGN-2 §7)
4 → call             # invoke the MCP tool            (spec §1, §6)
5 → validate         # error-envelope handling        (spec §4, §5)
6 → persist          # save verdict + transcript URI
7 → readback         # render the markdown card       (DESIGN-2 §8)
```

The activation block at the top of each SKILL.md routes by mode (autonomous /
yolo / guided), exactly as `samples/bmad-excalidraw/SKILL.md` does. Headless
mode short-circuits stages 1–3 (caller already framed the decision) and runs
4–7. This matches the BMad convention that headless mode jumps to the
generation step.

### What `customize.toml` exposes (cross-cutting)

Every panel-running workflow ships a `customize.toml` (DESIGN-1 §5,
"Customization surface") with at least:

```toml
# Stage 3 defaults — teams can pre-pick a pack + quota template
panel_template = ""              # path to a panelists.yaml (optional)
default_pack = ""                # pack id, e.g. "indie-builders"

# Stage 6 — where to write the local pointer record
verdict_output_path = "{project-root}/_bmad/synthpanel/verdicts/{run_id}.json"

# Stage 7 — readback shape
readback_card_path = "{project-root}/_bmad/synthpanel/cards/{run_id}.md"
on_complete = ""                 # optional shell hook

# Cross-cutting persistent context (Quinn always loads these)
activation_steps_prepend = []
activation_steps_append = []
persistent_facts = [
  "file:{project-root}/sp-v1-0-0-contract/spec.md",
  "file:{project-root}/sp-v1-0-0-contract/launch-plan.md",
]
```

Per-workflow customizations layer on top (see §3, §4, §5).

### Where the verdict lives on disk

`Stage 6 (persist)` writes two artifacts under
`{project-root}/_bmad/synthpanel/`:

- `verdicts/{run_id}.json` — the raw `panel_verdict.json` envelope returned by
  the MCP server. Source of truth for re-rendering and for `extend-panel`
  follow-ups.
- `cards/{run_id}.md` — the rendered markdown card (the "hero artifact" per
  launch-plan). Screenshottable, pasteable into a PR.

`run_id` comes from the MCP server response; if absent (older schema), Quinn
generates a UUIDv7 client-side and stamps it into `meta.run_id` before save.
This guarantees the verdict survives multiple workflow runs in one session and
is addressable by `extend-panel` and `render-readback`.

### Mapping from skill action to spec field

Each call-stage (Stage 4) constructs the request body strictly per spec §1:

| Workflow | Required request fields | Optional request fields |
|---|---|---|
| `convene-quick-poll` | `decision_being_informed` | `pack_id`, `size` (5–10), `quotas` |
| `convene-panel` | `decision_being_informed` | `pack_id`, `size` (12–20+), `quotas`, `seed` |
| `extend-panel` | `prior_run_id` (resolves to `full_transcript_uri`), `follow_up_question` | `additional_size` |

`decision_being_informed` is **never** sent to `extend-panel` (per spec §1: it
lives on `run_panel` / `run_quick_poll` / `extend_panel` — but `extend_panel`
inherits it from the prior run via the transcript URI; we do not re-coach a
frame mid-extension because the audit chain depends on the original frame).

> **Open clarification for implementation:** spec §1 lists `extend_panel` in
> the "Lives on" row for `decision_being_informed`. Workflow design here treats
> the field as **inherited from the prior run, not re-supplied**. If the
> server actually requires a fresh value on `extend_panel` calls, Stage 4 of
> `extend-panel` echoes the prior verdict's `meta.decision_being_informed`
> verbatim (zero paraphrase — DESIGN-2 §8). Either way the audit chain stays
> intact. File a follow-up bead if the spec disambiguates this differently.

---

## 3. `convene-quick-poll` (`QP`) — step-by-step

Fast signal for low-stakes decisions: copy variants, naming shortlists,
sub-headline phrasing. Default 5–10 personas. **Quinn explicitly names this is
not ship-blocking signal at readback time** (DESIGN-2 §7).

| # | Stage | Purpose | Prompt |
|---|---|---|---|
| 0 | Preflight | Probe `synthpanel` MCP tools (`run_quick_poll` required) | `references/preflight.md` |
| 1 | Trigger check | Confirm caller intent matches QP class (DESIGN-2 §5 categories: copy/naming/shortlist) | `references/trigger-check-qp.md` |
| 2 | Frame | Coach `decision_being_informed` to 12–280 chars, single-line; reject silently-truncated input | `references/frame.md` |
| 3 | Assemble | Default `size=8`, allow user override 5–10. Pack from audience, not asker (DESIGN-2 §7 rule 1). Warn if asker == audience cohort. | `references/assemble-qp.md` |
| 4 | Call | `run_quick_poll(decision_being_informed, pack_id, size, quotas?)` | `references/call.md` |
| 5 | Validate | Inspect typed-error envelope; on `MISSING_DECISION` / `DECISION_TOO_LONG` loop back to Stage 2; on `MODEL_TIMEOUT` retry once, then surface; on `INTERNAL_ERROR` surface and stop. (Spec §4) | `references/validate.md` |
| 6 | Persist | Save `verdicts/{run_id}.json` + `cards/{run_id}.md` per `customize.toml` paths | `references/persist.md` |
| 7 | Readback | Render the QP-flavored markdown card: lede the limit ("quick-poll, n=8 — not ship-blocking"), then standard convergence/dissent/flags shape (DESIGN-2 §8) | `references/readback-qp.md` |

### QP-specific defaults (`customize.toml` overlay)

```toml
default_size = 8
allowed_size_range = [5, 10]
readback_lede_template = "Quick-poll (n={size}, pack={pack_id}) — not ship-blocking. {headline}"
small_n_floor_class = "qp"        # tells assemble.md which floor to map against
```

### Failure modes worth naming up front

- **User asks for n>10 in QP**: Stage 3 stops and offers to switch to
  `convene-panel`. We do not silently up-route — the menu code is part of the
  audit story.
- **`small_n` flag fires anyway**: surface it; QP at n=5 on a hire-class
  decision will trip `small_n` and that's a feature, not a bug
  (DESIGN-2 §7 floor mapping).

---

## 4. `convene-panel` (`RP`) — step-by-step

The flagship workflow. Pricing, positioning, naming for product-level scope,
launch lede, hiring class. Default 16 personas.

| # | Stage | Purpose | Prompt |
|---|---|---|---|
| 0 | Preflight | Probe `synthpanel` MCP tools (`run_panel` required; `extend_panel` warn-if-missing) | `references/preflight.md` |
| 1 | Trigger check | Confirm caller intent matches RP class (DESIGN-2 §5: ≥1 of binary/N-way choice / hard-to-reverse / stakeholder breadth / external-audience copy) | `references/trigger-check-rp.md` |
| 2 | Frame | Coach `decision_being_informed` using the four-part frame (DESIGN-2 §6); reject stacked decisions and offer to split | `references/frame.md` (shared) |
| 3 | Assemble | Default `size=16`, allow override 12–20+. Pack selection per DESIGN-2 §7 rules (audience ≠ asker; mismatch → run anyway with `out_of_distribution`). Quota template from `customize.toml` if present. | `references/assemble-rp.md` |
| 4 | Call | `run_panel(decision_being_informed, pack_id, size, quotas?, seed?)` | `references/call.md` (shared) |
| 5 | Validate | Same envelope handling as QP. Additionally: on `schema_drift` flag (not error — spec §5), continue to readback but stamp the degradation in the card. | `references/validate.md` (shared) |
| 6 | Persist | Same as QP. Persist `seed` if supplied so `extend-panel` can replay persona realization. | `references/persist.md` (shared) |
| 7 | Readback | Full readback per DESIGN-2 §8 template. Lede the frame, then convergence/band, then dissent count, then top-3 verbatims (verbatim, no paraphrase), then flags, then "what this panel is and isn't." | `references/readback-rp.md` |

### RP-specific defaults (`customize.toml` overlay)

```toml
default_size = 16
allowed_size_range = [12, 40]
readback_lede_template = "{decision_being_informed}"   # the frame IS the lede
small_n_floor_class = "rp"
suggest_extend_on_low_convergence = true   # Stage 7 offers extend-panel as next step if convergence < 0.45
```

### Stage 7 → next-step plumbing

When `convergence < 0.45` (the "moderate" / "low" bands per DESIGN-2 §8) AND
`suggest_extend_on_low_convergence = true`, the readback ends with a single
nudge offering `extend-panel` on the cleavage. We do not auto-invoke; the user
opts in. This is how the EP workflow gets its real-world entry point — most
EP runs originate from a soft suggestion at the end of an RP run.

---

## 5. `extend-panel` (`EP`) — step-by-step

Follow-up on a prior run when the verdict raised a flag, surfaced a cleavage,
or the user has a downstream sub-question. Persona continuity preserved (uses
the same persona realization as the prior run).

| # | Stage | Purpose | Prompt |
|---|---|---|---|
| 0 | Preflight | Probe `extend_panel` available; resolve `prior_run_id` to a real `verdicts/{run_id}.json` on disk | `references/preflight-ep.md` |
| 1 | Trigger check | Confirm this is a follow-up, not a fresh decision. If user is reframing the original question, route to `convene-panel` instead. | `references/trigger-check-ep.md` |
| 2 | Frame | Coach `follow_up_question` (NOT a new `decision_being_informed`); echo prior frame so user sees what's being extended. | `references/frame-ep.md` |
| 3 | Assemble | Default `additional_size=0` (re-poll same panel with new question). Allow `additional_size > 0` to broaden. Same pack, same quotas — deviating defeats the purpose. | `references/assemble-ep.md` |
| 4 | Call | `extend_panel(prior_run_id_or_uri, follow_up_question, additional_size?)` | `references/call.md` (shared) |
| 5 | Validate | Same envelope handling. Additional case: `prior_run_id` not resolvable server-side → surface and offer to fall back to a fresh `convene-panel` run. | `references/validate.md` (shared) |
| 6 | Persist | Save under same `run_id` lineage: `verdicts/{run_id}-ext-{n}.json`. Update an `index.json` linking extensions to their parent. | `references/persist-ep.md` |
| 7 | Readback | Lede with "Extension of: {prior decision_being_informed} | Follow-up: {follow_up_question}". Then standard verdict shape. Diff convergence/dissent vs prior run if both available. | `references/readback-ep.md` |

### EP-specific defaults (`customize.toml` overlay)

```toml
default_additional_size = 0
allowed_additional_size_range = [0, 12]
readback_show_diff_vs_parent = true
```

### Failure mode: orphan extension

If `prior_run_id` resolves locally (we have the JSON) but the server returns
"unknown run," Stage 5 surfaces this explicitly: the persona realization is
gone server-side, persona continuity is no longer guaranteed. User can either
re-`convene-panel` from scratch or accept a re-realized panel (stamped with a
warning in the readback).

---

## 6. `render-readback` (`RB`) — step-by-step

Local-only. Re-render a saved `panel_verdict.json` as the markdown card.
Useful for: stale terminals where the original card scrolled off, sharing a
verdict from a prior session, regenerating after a `customize.toml` template
change.

| # | Stage | Purpose | Prompt |
|---|---|---|---|
| 0 | Preflight | (skip — no MCP needed) |  |
| 1 | Resolve | Take `run_id` or path argument; load `verdicts/{run_id}.json`; on miss, list nearest matches | `references/resolve-rb.md` |
| 2 | Render | Apply current `readback_card_path` template to the loaded verdict; surface schema-version mismatch if the card template expects newer fields than the verdict has | `references/render-rb.md` |
| 3 | Persist | Optionally overwrite `cards/{run_id}.md` (default: print to terminal only) | `references/persist-rb.md` |

No `customize.toml` of its own; reads the panel-running workflow's
`readback_card_path` template.

---

## 7. MCP tool mapping (consolidated)

| Workflow skill | MCP tool | Required args | Optional args | Notes |
|---|---|---|---|---|
| `convene-quick-poll` | `run_quick_poll` | `decision_being_informed` | `pack_id`, `size`, `quotas` | Size floor 5, ceiling 10 |
| `convene-panel` | `run_panel` | `decision_being_informed` | `pack_id`, `size`, `quotas`, `seed` | Size floor 12, suggested ceiling 40 |
| `extend-panel` | `extend_panel` | `prior_run_id` (or `full_transcript_uri`), `follow_up_question` | `additional_size` | See §2 open clarification on `decision_being_informed` |
| `render-readback` | (none) | `run_id` or path | `--write` flag | Local-only |

**Tools the pack does NOT wrap:**

- `run_prompt` — sub-decisional, not a panel call. Out of scope for the
  Facilitator persona (DESIGN-2 §5 anti-trigger: "user is doing sub-decisional
  scratch work").

---

## 8. What the implementation PR builds

The implementation PR (post-design) creates these skill folders under
`synthpanel_bmad/skills/` (per DESIGN-1 §1 layout):

```
skills/
  spb-convene-quick-poll/
    SKILL.md
    customize.toml
    references/
      preflight.md
      trigger-check-qp.md
      frame.md                  # symlink-or-shared; canonical lives in convene-panel
      assemble-qp.md
      call.md                   # shared
      validate.md               # shared
      persist.md                # shared
      readback-qp.md
  spb-convene-panel/
    SKILL.md
    customize.toml
    references/
      preflight.md
      trigger-check-rp.md
      frame.md                  # canonical
      assemble-rp.md
      call.md                   # canonical
      validate.md               # canonical
      persist.md                # canonical
      readback-rp.md
  spb-extend-panel/
    SKILL.md
    customize.toml
    references/
      preflight-ep.md
      trigger-check-ep.md
      frame-ep.md
      assemble-ep.md
      persist-ep.md
      readback-ep.md
  spb-render-readback/
    SKILL.md
    references/
      resolve-rb.md
      render-rb.md
      persist-rb.md
```

**Sharing decision (resolve in implementation):** BMad skills don't have a
first-class "shared references" mechanism — each skill is meant to be
self-contained. Implementation should either (a) duplicate `frame.md` /
`call.md` / `validate.md` / `persist.md` into each skill (simplest, fits the
"skill = unit of work" rule), or (b) extract them into a sibling
`spb-shared-references/` skill that the others load via `{project-root}` paths
(DRY but introduces cross-skill coupling). Recommendation: **(a) duplicate**
for v0.1; revisit if the shared text grows past ~200 lines per file.

### `module-help.csv` rows the pack registers

```csv
SynthPanel,_meta,,,,,,,,,false,<docs URL>,
synthpanel,convene-quick-poll,Quick Poll,QP,Fast 5–10-persona signal on a low-stakes choice,run_quick_poll,decision_being_informed,anytime,,,true,verdict_output_path,verdict-card
synthpanel,convene-panel,Convene Panel,RP,Standard 12–20-persona structured deliberation,run_panel,decision_being_informed,anytime,,,true,verdict_output_path,verdict-card
synthpanel,extend-panel,Extend Panel,EP,Follow-up question on an existing panel run,extend_panel,prior_run_id|follow_up_question,anytime,convene-panel:run_panel,,true,verdict_output_path,verdict-card
synthpanel,render-readback,Render Readback,RB,Re-render a saved verdict as the markdown card,render-card,run_id,anytime,,,false,readback_card_path,verdict-card
```

(Column order per DESIGN-1 §3.)

---

## 9. Open follow-ups

File as separate beads when the implementation PR lands or telemetry arrives:

- **`compare-verdicts` workflow (`CV`)** — compare two saved
  `panel_verdict.json` artifacts. Defer to v0.2; needs at least two stable
  verdicts in the wild to design against.
- **Headless / CI mode for `convene-quick-poll`** — useful for "before merging
  this copy change, check it against the cohort." Out of scope for v0.1 because
  it requires a deterministic seed contract with the MCP server.
- **Spec §1 clarification on `extend_panel.decision_being_informed`** — see §2
  open clarification. File once spec maintainers are reachable.
- **Calibrate `small_n_floor_class` mappings** — DESIGN-2 §7 punts to
  post-v1.0 telemetry; revisit after the first ~100 real panel runs land in
  evaluator hands.
- **Persona-pack discovery** — Stage 3 currently assumes the user (or
  `customize.toml`) names the pack. A `list-packs` capability against the MCP
  server would help; depends on whether the server exposes one.
