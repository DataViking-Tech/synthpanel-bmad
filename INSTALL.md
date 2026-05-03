# Install & Setup

This pack is the BMad expansion for **SynthPanel** — Quinn (Panel Facilitator)
plus the `convene-panel` / `convene-quick-poll` / `extend-panel` /
`render-readback` workflows. It wraps the SynthPanel MCP server's deliberation
tools; the pack itself does **not** bundle that server.

Setup is three steps:

1. Install **SynthPanel** (the MCP server) and register it with your host.
2. Install **synthpanel-bmad** (this pack) into your project.
3. Run the **`spb-setup`** skill to write project config.

## Compatibility

| This pack | Required SynthPanel `schema_version` |
|---|---|
| `0.1.x` | `>=1.0.0,<2.0.0` |

Pre-1.0 — breaking changes possible on minor bumps. See
[`design-notes/distribution-versioning.md`](design-notes/distribution-versioning.md)
for the full versioning policy.

---

## 1. Install SynthPanel (prereq)

The pack's workflows call SynthPanel via MCP. You need it installed and
reachable from your host **before** running any `convene-*` workflow.

### 1a. Install the server

```bash
pip install 'synthpanel>=1.0,<2.0'
```

Verify:

```bash
synthpanel --version   # expect 1.0.x
```

### 1b. Set your LLM key

SynthPanel is BYOK. Set **one** of these in your shell profile *or* in the host
MCP `env` block (below):

```bash
export ANTHROPIC_API_KEY=sk-ant-...
# or
export OPENAI_API_KEY=sk-...
# or
export GEMINI_API_KEY=...
# or point at a local inference endpoint per SynthPanel docs
```

### 1c. Register the MCP server with your host

#### Claude Desktop

Edit `claude_desktop_config.json` (macOS:
`~/Library/Application Support/Claude/claude_desktop_config.json`; Windows:
`%APPDATA%\Claude\claude_desktop_config.json`):

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

Restart Claude Desktop after saving.

#### Claude Code (CLI)

```bash
claude mcp add synthpanel synthpanel mcp \
  --env ANTHROPIC_API_KEY=sk-ant-...
```

#### Cursor / Continue / Windsurf

Each host has its own MCP config UI. Use the same `command: synthpanel`,
`args: ["mcp"]`, and `env` shape, and consult the host's MCP docs for where
to put it. SynthPanel's own docs site mirrors per-host instructions when they
diverge.

### 1d. Verify the connection

In your host, ask the agent:

> Call `run_quick_poll` with the decision "ship friday vs monday" and 3 voices.

A successful response with a `panel_verdict` envelope confirms the MCP wiring
works. (The pack's `spb-setup` skill and every `convene-*` workflow run a
preflight probe automatically — this manual check is just for sanity.)

---

## 2. Install the pack

This pack ships through the BMad / Claude Code plugin marketplace. The
`.claude-plugin/marketplace.json` at the repo root is the listing manifest.

### Claude Code

From within a Claude Code session in your project root:

```text
/plugin marketplace add synthpanel/synthpanel-bmad
/plugin install synthpanel-bmad@synthpanel-bmad
```

For local development against a checkout:

```text
/plugin marketplace add ./path/to/synthpanel-bmad
/plugin install synthpanel-bmad@synthpanel-bmad
```

This installs all six skills (`spb-setup`, `bmad-agent-facilitator`,
`spb-convene-panel`, `spb-convene-quick-poll`, `spb-extend-panel`,
`spb-render-readback`) into the project's plugin scope.

### Other BMad hosts

Use your host's standard expansion-pack install flow against this repo or its
marketplace listing (`synthpanel-bmad`).

---

## 3. Configure the project

In your host, invoke the setup skill:

> `/spb-setup`

Or describe it in prose: *"install synthpanel"*, *"configure SynthPanel"*,
*"setup spb module"*.

`spb-setup` will:

1. Probe the SynthPanel MCP server (warns, but does not block, if it can't
   reach the server — workflows will preflight-fail later if the server stays
   unreachable).
2. Prompt for three module values (defaults shown):
   - **`verdict_output_path`** — where raw `panel_verdict.json` artifacts are
     saved (default: `{project-root}/_bmad/synthpanel/verdicts`).
   - **`readback_card_path`** — where rendered markdown verdict cards land
     (default: `{project-root}/_bmad/synthpanel/cards`).
   - **`mcp_server_url`** — `stdio` for a local server (default), or a URL
     for a remote one.
3. Write `_bmad/config.yaml` (shared) and `_bmad/config.user.yaml`
   (gitignored, personal).
4. Register the four user-facing capabilities (`QP`, `RP`, `EP`, `RB`) in
   `_bmad/module-help.csv`.

For a non-interactive install:

> `/spb-setup accept all defaults`

### Recommended `.gitignore` entries

```gitignore
_bmad/config.user.yaml
_bmad/synthpanel/verdicts/
_bmad/synthpanel/cards/
```

`config.user.yaml` holds personal overrides (your name, language) and any
variables marked `user_setting: true`. Verdict JSON and rendered cards are
local artifacts — commit them only if your team explicitly wants a shared
record.

---

## Verify the install

From your host:

> *"Convene a quick poll on whether to ship a `--dry-run` flag for our
> migrate command. 5 voices."*

Expected: Quinn (the Panel Facilitator) acknowledges the decision, runs
`run_quick_poll` via the SynthPanel MCP server, and returns a
`panel_verdict` rendered as a verdict card. If the preflight probe fails,
Quinn stops with a pointed error linking back to **§1** of this doc.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `convene-*` halts with "SynthPanel MCP server is not reachable" | Host hasn't been restarted after registering the server, or the `synthpanel` command is not on `PATH` | Restart host; run `which synthpanel` to confirm install |
| `run_quick_poll` returns an auth error | LLM key missing or wrong env var | Set the matching `*_API_KEY` in either your shell or the MCP `env` block; restart host |
| Pack greets you but no `convene-*` skills appear | `spb-setup` not yet run, or run in the wrong project root | Re-run `/spb-setup` from the project root |
| Verdict JSON not written | `verdict_output_path` directory missing | `mkdir -p _bmad/synthpanel/verdicts` (setup creates this; recreate if you've cleaned `_bmad/`) |
| `schema_version` mismatch error | Server schema major differs from the pack's supported range | Upgrade the pack to a version whose compatibility table covers the server's major (or pin SynthPanel to a compatible major) |

For schema-drift behavior in detail, see
[`design-notes/distribution-versioning.md` §3.3](design-notes/distribution-versioning.md).

---

## What this pack does **not** do

- **Install SynthPanel for you.** It is a documented prereq (§1).
- **Manage your LLM keys.** Place them in your shell or the host's MCP `env`
  block — never in pack files.
- **Auto-restart your host** after you edit MCP config. You must restart.

The rationale for this division of labor lives in
[`design-notes/mcp-host-setup.md`](design-notes/mcp-host-setup.md) (DESIGN-5).
