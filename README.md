# synthpanel-bmad

BMad expansion pack for **SynthPanel** — agent + workflows for synthetic
deliberation panels on hard-to-reverse product decisions.

The pack ships **Quinn (the Panel Facilitator)** plus four workflows
(`convene-quick-poll`, `convene-panel`, `extend-panel`, `render-readback`)
that wrap the SynthPanel MCP server's deliberation tools. Use it to get
structured multi-persona signal on the kind of choices where "ask three
people in Slack" is too thin and "run a full user study" is too slow.

The pack itself does **not** bundle the MCP server. SynthPanel is a
prerequisite — see [§ Setup](#setup).

## What you get

| Capability | Code | Wraps | Use it for |
|---|---|---|---|
| **Convene a Quick Poll** | `QP` | `run_quick_poll` | 5–10-persona signal on copy variants, naming shortlists, sub-headline phrasing. **Not** ship-blocking. |
| **Convene a Panel** | `RP` | `run_panel` | 12–20-persona structured deliberation on pricing, positioning, hiring, launch lede, external-audience copy. Default n=16. |
| **Extend a Panel** | `EP` | `extend_panel` | Follow-up question on a prior panel — persona continuity preserved, convergence diffed against parent. |
| **Render a Readback** | `RB` | (local) | Re-render a saved `panel_verdict.json` as the markdown verdict card. |

Each workflow runs an eight-stage spine — preflight → trigger-check → frame →
assemble → call → validate → persist → readback — and saves both the raw
verdict envelope (`_bmad/synthpanel/verdicts/{run_id}.json`) and a rendered
markdown card (`_bmad/synthpanel/cards/{run_id}.md`). The card is the
shareable artifact: paste it into a PR, a doc, or a Slack thread.

Quinn (the agent) decides which workflow fits and runs it. You usually don't
invoke skills by name — you describe the decision in prose.

## Setup

See [**INSTALL.md**](INSTALL.md) for copy-paste install + setup steps.

Three steps:

1. Install **SynthPanel** (the MCP server) and register it with your host.
2. Install **synthpanel-bmad** via your host's plugin / expansion-pack flow.
3. Run **`/spb-setup`** to write project config.

## Workflow examples

The natural way to invoke Quinn is to describe the decision. Quinn picks the
workflow; the menu codes (`QP` / `RP` / `EP` / `RB`) are available as fallback
shortcuts when you want to be explicit.

### Quick Poll (`QP`) — low-stakes signal

> *"Quinn, quick poll: should we name the new flag `--dry-run` or
> `--preview`? 8 voices."*

Quinn runs `run_quick_poll` against a default 8-persona cohort, validates the
response envelope, persists the verdict, and reads it back with the limit
named up front:

```
Quick-poll (n=8, pack=indie-builders) — not ship-blocking. --dry-run wins
6/8, with two voices flagging --preview reads less destructive on first
encounter.

Top dissent (verbatim):
- "preview matches the screenshot tooling I'd already have open."
- "dry-run is what every CLI I use says, that's the whole point."
```

### Convene Panel (`RP`) — flagship deliberation

> *"I'm setting our launch tier at $19/mo or $29/mo. Convene a panel."*

Quinn coaches the frame (four-part: choice, audience, success, constraint),
defaults to n=16 with an audience-matched pack, and runs `run_panel`. The
readback ledes with the frame, then convergence band, dissent count, top-3
verbatim quotes, surfaced flags, and what this panel *isn't* (a substitute
for pricing-page A/B data).

If `convergence < 0.45`, the readback ends with a soft offer to extend the
panel on the cleavage — your opt-in to `extend-panel`.

### Extend Panel (`EP`) — follow-up on a prior run

> *"Extend the pricing panel — what does the cohort think about a free
> tier as a wedge?"*

Quinn loads the prior run's persona realization, calls `extend_panel` with
your follow-up, and reads the verdict back as **`Extension of: …`** —
diffing convergence and dissent against the parent run so you can see whether
the cohort moved.

### Render Readback (`RB`) — re-render a saved verdict

> *"Render the readback for run 01HZ… — terminal scrolled."*

Local-only. Loads `_bmad/synthpanel/verdicts/{run_id}.json`, applies the
current card template, and prints (or with `--write`, overwrites) the
markdown card. Useful for sharing a verdict from a prior session or
regenerating a card after a `customize.toml` template change.

## Compatibility

| This pack | Required SynthPanel `schema_version` |
|---|---|
| `0.1.x` | `>=1.0.0,<2.0.0` |

Pre-1.0 — breaking changes possible on minor bumps. See
[`CHANGELOG.md`](CHANGELOG.md) for per-release schema-compat notes and
[`design-notes/distribution-versioning.md`](design-notes/distribution-versioning.md)
for the full versioning policy.

## License

[MIT](LICENSE).
