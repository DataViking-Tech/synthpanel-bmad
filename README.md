# synthpanel-bmad

BMad expansion pack for SynthPanel — agent + workflows for synthetic
deliberation panels.

Quinn (the Panel Facilitator) plus four workflows — `convene-panel`,
`convene-quick-poll`, `extend-panel`, `render-readback` — that wrap the
SynthPanel MCP server for structured multi-persona deliberation on
hard-to-reverse product decisions.

## Setup

See [**INSTALL.md**](INSTALL.md) for copy-paste install + setup steps.

Three steps:

1. Install **SynthPanel** (the MCP server) and register it with your host.
2. Install **synthpanel-bmad** via your host's plugin/expansion-pack flow.
3. Run **`/spb-setup`** to write project config.

## Compatibility

| This pack | Required SynthPanel `schema_version` |
|---|---|
| `0.1.x` | `>=1.0.0,<2.0.0` |

Pre-1.0 — breaking changes possible on minor bumps. Versioning policy:
[`design-notes/distribution-versioning.md`](design-notes/distribution-versioning.md).
