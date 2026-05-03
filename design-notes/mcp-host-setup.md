# DESIGN-5 — MCP host prereq + setup

**Issue:** spb-design-5
**Status:** decided
**Date:** 2026-05-03

## Decision

**Treat the SynthPanel MCP server as a documented prerequisite, not a bundled
setup workflow.** The expansion pack will not install or auto-configure
SynthPanel. The pack ships:

1. A **prereq section** in the pack README with copy-paste install commands and
   MCP-host config snippets for the supported hosts (Claude Desktop, Cursor,
   Continue, Windsurf).
2. A **preflight step** in every panel-running workflow that probes for the
   `synthpanel` MCP toolset and stops with a pointed error (linking the README)
   if the tools are not reachable.
3. A **schema-version compatibility note**: pack `0.1.x` requires SynthPanel
   schema `1.0.x` (see DESIGN-6 for the full versioning policy).

## Rationale

### Why not bundle setup

- **BMad expansion packs are not package managers.** They are workflow + persona
  + skill templates. Reaching outside the project to `pip install` a Python
  package, write to `~/.config/`, or edit a host config file is a category
  error — and on most hosts it requires a process restart the pack cannot
  trigger.
- **MCP host config files diverge.** Claude Desktop uses
  `claude_desktop_config.json`, Cursor has its own settings UI + JSON, Continue
  reads `~/.continue/config.json`, Windsurf has its own format. A bundled
  installer would need per-host code paths and would silently bit-rot as host
  configs evolve. The release-readiness doc only commits to verifying two
  hosts at v1.0; we should not promise installer support for hosts SynthPanel
  itself does not yet certify.
- **Credentials must stay user-owned.** SynthPanel is BYOK
  (`ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `GEMINI_API_KEY` / local). A
  bundled setup workflow that prompts for and persists keys would either be
  insecure (writing keys into project files) or reproduce a credential manager
  the host already has. Letting the user place keys in the host's standard env
  block is the right division of labor.
- **Independent release cadence.** SynthPanel ships from
  `sp-v1-0-0-contract` on PyPI; this pack ships through the BMad marketplace.
  Coupling install logic into the pack would force a pack release every time
  SynthPanel changes its install story.

### Why a prereq doc + preflight, not just a prereq doc

- The dominant failure mode for new users is "I installed the pack, ran the
  facilitator, got a blank stare." A preflight tool-probe converts that into a
  specific, actionable error pointing at the install instructions. Cost: one
  list-tools call per workflow run.

## Prereq doc — outline

This is the structure the pack README's "Setup" section must contain. (Actual
copy lives in the README, written when the pack is scaffolded.)

### 1. Install SynthPanel

```bash
pip install 'synthpanel>=1.0,<2.0'
```

Verify:

```bash
synthpanel --version   # expect 1.0.x
```

### 2. Set your LLM key

SynthPanel is BYOK. Set **one** of:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export GEMINI_API_KEY=...
# or point at a local inference endpoint per SynthPanel docs
```

Place this in your shell profile or in the host's MCP `env` block (below).

### 3. Register the MCP server with your host

**Claude Desktop** — edit `claude_desktop_config.json`
(macOS: `~/Library/Application Support/Claude/`):

```jsonc
{
  "mcpServers": {
    "synthpanel": {
      "command": "synthpanel",
      "args": ["mcp"],
      "env": { "ANTHROPIC_API_KEY": "sk-ant-..." }
    }
  }
}
```

Restart Claude Desktop.

**Cursor / Continue / Windsurf** — link out to each host's MCP config doc and
the matching SynthPanel docs page; do not reproduce host configs that drift
out of date.

### 4. Verify the connection

In your host, ask the agent to call `run_quick_poll` with a trivial decision.
A successful 200 with a `panel_verdict` envelope confirms setup. The pack's
own preflight step does this automatically before the first panel.

## Preflight workflow step — sketch

Every workflow that calls SynthPanel MCP tools begins with:

```yaml
- id: preflight
  name: Verify SynthPanel MCP availability
  action: probe-mcp-tools
  expects:
    - run_panel
    - run_quick_poll
    - extend_panel
  on_missing:
    fail_with: |
      SynthPanel MCP server is not reachable.

      The Panel Facilitator workflows require the synthpanel MCP server to be
      installed and registered with this host. Setup instructions:
      <pack-readme-url>#setup

      Once SynthPanel is installed and the host restarted, re-run this
      workflow.
```

The exact action name (`probe-mcp-tools`) is a placeholder; the real wiring
depends on what BMad's runtime exposes for tool introspection (DESIGN-1
output). If BMad does not expose a tool-listing primitive, fall back to a
guarded first call to `run_quick_poll` with a known-cheap decision and
treat a "tool not found" error as the preflight failure.

## Schema-version compatibility

The pack README must state the required SynthPanel schema range and how it
maps to pack versions. Concrete mapping is owned by DESIGN-6; the prereq doc
just surfaces it:

> This pack version `0.1.x` requires SynthPanel schema `1.0.x`. If
> SynthPanel returns a `schema_version` outside this range, the facilitator
> will refuse to run and link to the matching pack release.

## Acceptance check

- [x] Decision recorded with rationale (this file).
- [x] If bundled: install steps documented. **Decision is "not bundled"; install
      steps are nonetheless documented as the prereq outline above so the pack
      author has a copy-paste source when scaffolding the README.**

## Open follow-ups (file as separate beads if pursued)

- Confirm BMad runtime has a tool-introspection primitive for the preflight
  step; if not, the fallback path is a guarded `run_quick_poll` call.
- Once SynthPanel publishes its own host-compat matrix, mirror the supported
  hosts list in the pack README rather than maintaining a parallel list.
- Decide whether the preflight step is opt-out (some users may want offline
  dry-runs of the facilitator persona without a live MCP connection).
