# Changelog

All notable changes to **synthpanel-bmad** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Pre-1.0 — breaking changes are possible on minor bumps. See
[`design-notes/distribution-versioning.md`](design-notes/distribution-versioning.md)
for the full versioning policy and the pack ↔ SynthPanel `schema_version`
mapping rule.

## [Unreleased]

**Compatible with SynthPanel `schema_version`:** `>=1.0.0,<2.0.0`

### Added
- (none)

### Changed
- (none)

### Deprecated
- (none)

### Removed
- (none)

### Fixed
- (none)

---

## [0.1.0] — 2026-05-03

**Compatible with SynthPanel `schema_version`:** `>=1.0.0,<2.0.0`

Initial release. The pack ships the BMad-side surface area for SynthPanel: a
facilitator agent (Quinn), four user-facing workflows wrapping the SynthPanel
MCP server's deliberation tools, and a setup skill for project-level config.

### Added
- **Agent: `bmad-agent-facilitator` (Quinn).** Panel Facilitator persona —
  triggers on hard-to-reverse decisions, coaches `decision_being_informed`,
  routes to the right workflow, and reads back verdicts verbatim. See
  [`design-notes/facilitator-persona.md`](design-notes/facilitator-persona.md).
- **Workflow: `spb-convene-quick-poll` (`QP`).** Wraps `run_quick_poll` for
  fast 5–10-persona signal on copy / naming / shortlist decisions. Readback
  explicitly names the limit ("not ship-blocking").
- **Workflow: `spb-convene-panel` (`RP`).** Wraps `run_panel` for the standard
  12–20-persona structured deliberation on pricing, positioning, hiring, and
  external-audience copy. Default size 16.
- **Workflow: `spb-extend-panel` (`EP`).** Wraps `extend_panel` for follow-up
  questions on a prior run. Persona continuity preserved; readback diffs
  convergence vs the parent run.
- **Workflow: `spb-render-readback` (`RB`).** Local-only re-render of a saved
  `panel_verdict.json` as the markdown verdict card. Useful for sharing prior
  verdicts or regenerating after a `customize.toml` template change.
- **Skill: `spb-setup`.** Project-level configurator. Probes the SynthPanel
  MCP server, prompts for module values (`verdict_output_path`,
  `readback_card_path`, `mcp_server_url`), writes `_bmad/config.yaml` +
  `_bmad/config.user.yaml`, registers the four workflow capabilities in
  `_bmad/module-help.csv`.
- **Marketplace listing.** `.claude-plugin/marketplace.json` registers the
  pack for installation via `/plugin marketplace add` + `/plugin install`.
- **Docs.** `README.md`, `INSTALL.md`, plus design notes covering BMad
  conventions, facilitator persona, workflow scoping, MCP host setup,
  trigger heuristics, and distribution + versioning policy.

### Schema-compat
- Targets the frozen SynthPanel `schema_version` `1.0.x`. Schema range
  declared as `>=1.0.0,<2.0.0` per the v0.1.x compatibility row in
  [`design-notes/distribution-versioning.md`](design-notes/distribution-versioning.md).
  Newer minors are accepted optimistically (unknown fields ignored); a
  different MAJOR is a hard stop.

[Unreleased]: https://github.com/synthpanel/synthpanel-bmad/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/synthpanel/synthpanel-bmad/releases/tag/v0.1.0
