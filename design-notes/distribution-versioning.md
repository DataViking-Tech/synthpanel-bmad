# Distribution & Versioning

**Status:** Proposed (DESIGN-6)
**Owner:** synthpanel-bmad maintainers
**Audience:** pack maintainers, downstream BMad users, SynthPanel core team

This note records the policy for how the `synthpanel-bmad` expansion pack is
distributed, versioned, and released. It pairs with the SynthPanel v1.0.0
contract (`/sp-v1-0-0-contract/spec.md`), which defines the wire-level
`schema_version` this pack consumes via MCP.

---

## 1. Distribution

### 1.1 Channel: BMad public marketplace

The pack is published as a public BMad expansion pack. It is discoverable and
installable via the standard BMad expansion-pack mechanism — no out-of-band
download, no private registry.

| Field | Value |
|---|---|
| Pack ID | `synthpanel-bmad` |
| Listing name | `SynthPanel — Synthetic Deliberation Panels` |
| Tags | `synth-panel`, `deliberation`, `decision-support`, `mcp` |
| Repo | `bmad-code-org`-style upstream (TBD on publish) |
| License | Same as the SynthPanel project (TBD; mirror upstream choice) |
| Prereq | SynthPanel MCP server reachable (see DESIGN-5) |

Listing copy MUST state, in plain language:

1. What the pack does (facilitates synthetic deliberation panels for decisions).
2. The required external dependency (SynthPanel MCP server, a specific
   `schema_version` range — see §3).
3. That installing the pack alone does not provision the MCP server.

### 1.2 What ships in the pack

The published artifact contains only the BMad-side surface area: agent persona,
workflows, skills, and supporting docs. It does NOT vendor the SynthPanel MCP
server or its schema files; those live in the SynthPanel project and are
referenced by version. This keeps the pack small and avoids duplicating a
contract that has its own release cadence.

### 1.3 Out of scope for v1

- Private/enterprise channels.
- Mirroring to non-BMad registries.
- Auto-installing the MCP server.

---

## 2. Versioning — semver, applied to the pack

The pack uses **semantic versioning** (`MAJOR.MINOR.PATCH`) with the following
contract for what each component means **for pack consumers** (not for the
SynthPanel wire schema, which has its own semver — see §3):

| Bump | Triggers |
|---|---|
| **MAJOR** | Backwards-incompatible change to the agent's externally observable behavior, workflow names/IDs, persona triggers, or the supported `schema_version` range (e.g. dropping support for an older schema major). Removing a workflow. Renaming the pack ID. |
| **MINOR** | New workflows, new skills, new persona capabilities, expanded `schema_version` support, or any change that adds capability without breaking existing wiring. |
| **PATCH** | Doc fixes, prompt tuning that does not change observable workflow shape, dependency updates that don't change behavior, internal refactors. |

### 2.1 Pre-1.0 caveat

While the pack is on a `0.MINOR.PATCH` line, MINOR bumps MAY include breaking
changes — this is conventional pre-1.0 semver and lets us iterate on persona
voice and workflow shape before public commitment. The pack reaches `1.0.0`
when:

1. SynthPanel MCP itself is on a stable `1.x.x`.
2. At least one full deliberation workflow has been used end-to-end against
   that stable MCP.
3. Persona voice / verdict-readback shape (DESIGN-2) has settled.

Until then the listing notes the pack as **pre-1.0 / breaking changes possible
on minor bumps**.

### 2.2 Version source of truth

The pack version lives in the BMad pack manifest (one place), and is mirrored
into:

- `CHANGELOG.md` (newest entry header)
- The release tag on git (`v0.1.0`, `v0.2.0`, …)
- The marketplace listing (auto-derived from the manifest at publish time)

CI MUST fail a release if any of these three disagree.

---

## 3. Tracking SynthPanel `schema_version`

The SynthPanel MCP server publishes a wire contract identified by
`schema_version` (per `/sp-v1-0-0-contract/spec.md`: v1.0.0 frozen at
handoff, schema location `synthpanel/schemas/v1.0.0.json`, `schema_version`
echoed in every response and every error, breaking changes = major bump +
parallel schema file).

The pack's job is to **call that contract correctly** — so each pack release
declares which `schema_version` range it targets.

### 3.1 Mapping rule

The pack version and schema version are **independent semver lines** that are
**linked by an explicit support range**, not by sharing a number.

> **Rule:** the pack's MAJOR version is bumped when its supported
> `schema_version` MAJOR changes (drop or replace). MINOR and PATCH bumps do
> NOT change the supported schema MAJOR, but MAY widen support (add a newer
> schema MINOR) or narrow it (deprecate, then drop in the next pack MAJOR).

Example trajectory (illustrative, matches the bead's example):

| Pack | Supported `schema_version` | Notes |
|---|---|---|
| `0.1.x` | `1.0.x` | Initial release. Targets the frozen v1.0.0 schema. |
| `0.2.0` | `1.0.x \|\| 1.1.x` | Adds support for a new minor (e.g. `panel_verdict.flags` additions). No pack-side break. |
| `0.3.0` | `1.0.x \|\| 1.1.x` | Adds new workflow. Schema range unchanged. |
| `1.0.0` | `1.x` | Pack goes stable. Supports schema majors `1.x`. |
| `2.0.0` | `2.x` | SynthPanel ships schema v2; pack MAJOR-bumps to drop v1 support. |

### 3.2 Manifest field

Each pack release MUST declare the supported range in its manifest:

```yaml
synthpanel:
  schema_version: ">=1.0.0,<2.0.0"
```

This is the contract the pack publishes to its users. It is consumed by:

- The agent persona at runtime (for the warning in §3.3).
- Doc generation (the README "Compatible with SynthPanel" badge).
- CI (validates that the pack's example calls validate against every schema
  version inside the declared range).

### 3.3 Runtime drift behavior

When the pack's facilitator agent receives a `schema_version` from the MCP
server that is OUTSIDE its declared range, behavior is:

| Drift kind | Behavior |
|---|---|
| Server schema is **newer MINOR** than declared | Continue. Log a one-line notice ("server is on 1.2, pack supports up to 1.1 — proceeding optimistically"). New optional fields in the response are ignored, not errored. |
| Server schema is **older MINOR** than declared | Continue. Persona avoids fields that didn't exist in the older minor. |
| Server schema is **different MAJOR** | Hard stop. Surface a typed error with remediation: "Install pack version that supports schema X.y" with a link to the pack's compatibility table. |

This is consistent with SynthPanel's own append-only-schema discipline (per
`spec.md`: breaking changes = major bump + parallel schema file, no in-place
mutation).

### 3.4 Why not unify the version numbers

It is tempting to make pack `1.0.0` mean "supports schema `1.0.0`". We
explicitly reject this because:

- The pack ships independently of SynthPanel and will frequently release
  PATCH/MINOR versions that have nothing to do with the schema.
- Conflating the numbers makes "the schema didn't change but we shipped a doc
  fix" require a contortion (e.g. `1.0.0+1`), which BMad's listing UI may not
  render cleanly.
- Independent numbering with an explicit support range is the same pattern
  used by language SDKs vs. their wire APIs — well-understood.

---

## 4. Changelog conventions

### 4.1 Format

`CHANGELOG.md` at the repo root, **Keep a Changelog** style, newest entry on
top. Each released version gets a section with the date and the supported
`schema_version` range:

```markdown
## [0.2.0] — 2026-06-15

**Compatible with SynthPanel `schema_version`:** `>=1.0.0,<1.2.0`

### Added
- Workflow: `quick-poll` for sub-decisional scratch work.
- Persona trigger heuristic: panel size auto-suggested from
  `decision_being_informed` length.

### Changed
- Verdict-readback template now surfaces `dissent_count` before
  `top_3_verbatims`.

### Deprecated
- (none)

### Removed
- (none)

### Fixed
- Facilitator no longer drops `meta.decision_being_informed` when echoing
  verdicts back.

### Schema-compat
- Added support for SynthPanel `schema_version` `1.1.x` (new optional
  `flags` enum members are ignored gracefully).
```

### 4.2 Required sections

Every released entry MUST include:

1. The version + date header.
2. The **Compatible with SynthPanel `schema_version`** line, even when
   unchanged from the prior release. This makes drift visible at a glance.
3. At least one of `Added` / `Changed` / `Deprecated` / `Removed` / `Fixed`.
4. A `Schema-compat` subsection whenever the supported `schema_version` range
   changes.

### 4.3 Unreleased section

An `## [Unreleased]` section sits at the top and accumulates entries during
development. The release process renames `[Unreleased]` to the new version
header, stamps the date, and creates a fresh empty `[Unreleased]` block.

### 4.4 Breaking-change call-out

Any MAJOR bump MUST include a **`### Breaking`** subsection that lists, with
migration guidance, every observable change. If a workflow ID, agent name, or
manifest field renamed, the old and new names appear side by side.

### 4.5 Commit message hints (optional)

Conventional-commit-style prefixes (`feat:`, `fix:`, `chore:`, `docs:`,
`refactor:`, `BREAKING CHANGE:`) are encouraged but not enforced — the
changelog is curated by hand at release time, not auto-generated, so prefixes
are a hint, not a contract.

---

## 5. Release process (summary)

1. Open a release PR that:
   - Renames `[Unreleased]` to the new version, stamps the date.
   - Bumps the manifest version.
   - Updates the supported `schema_version` range if it changed.
2. CI runs:
   - Manifest version === changelog top entry === proposed git tag.
   - Pack examples validate against every `schema_version` in the declared
     range.
3. Merge → tag (`vX.Y.Z`) → BMad marketplace publish job picks up the tag.
4. Post-publish: the listing's "compatible with" badge auto-updates from the
   manifest range.

---

## 6. Open questions (defer to follow-up beads)

- License selection — match SynthPanel upstream once chosen.
- Whether to publish a `next` / `beta` channel on the marketplace pre-1.0,
  or keep all `0.x` releases on the default channel with a clear pre-1.0
  warning in the listing.
- Whether the pack should ship a one-shot "check my SynthPanel version"
  diagnostic skill, or rely entirely on runtime drift handling (§3.3).

---

## References

- SynthPanel v1.0.0 contract: `/sp-v1-0-0-contract/spec.md`
- SynthPanel build plan (changelog precedent): `/sp-v1-0-0-contract/build-plan.md`
- BMad expansion pack conventions: see DESIGN-1 output (`design-notes/bmad-conventions.md`).
- MCP host prereq + setup: see DESIGN-5.
