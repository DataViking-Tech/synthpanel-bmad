# BMad Conventions: What We Copy vs Invent

Source: [bmad-code-org/bmad-builder](https://github.com/bmad-code-org/bmad-builder)
(studied at commit on `main`, 2026-05-03; cloned shallow into `/tmp/bmad-study/bmad-builder`).

This doc captures the conventions the upstream BMad ecosystem uses for skills, agents,
workflows, modules, and packaging — and flags which of those SynthPanel BMad
(`synthpanel_bmad`, an expansion pack for synthetic deliberation panels) should adopt
verbatim vs invent on top.

---

## 1. Skill = the unit of work

Every skill is a directory. The directory name is the skill name (kebab-case),
and it always contains a `SKILL.md` with YAML frontmatter:

```yaml
---
name: <kebab-case, must match folder>
description: <5–8 word summary>. <"Use when user says 'X' or 'Y'.">
---
```

Reference: `samples/sample-module-setup/SKILL.md`,
`skills/bmad-workflow-builder/references/standard-fields.md`.

**Frontmatter is intentionally minimal — only `name` and `description`.** Everything
else (role, stages, tools, args) lives in the SKILL.md body. The `description` is
the *primary trigger mechanism* for AI invocation, so it is written as a 2-part
sentence: "[what it does]. [Use when user requests 'phrase' or 'phrase']."

A skill folder typically contains:

```
<skill-name>/
  SKILL.md             # entry point; describes activation, stages, body
  assets/              # templates, defaults, module.yaml/help.csv (if a module skill)
  references/          # markdown loaded on demand by SKILL.md (kept lean by design)
  scripts/             # python helpers (see scripts/tests/ alongside)
  customize.toml       # OPTIONAL: end-user/team override surface (workflow skills)
```

See: `samples/bmad-agent-sentinel/`, `samples/bmad-excalidraw/`,
`skills/bmad-agent-builder/`. None of the studied skills use a sub-skills tree —
SKILL.md is the only top-level markdown.

### Path conventions inside a skill (verbatim from `standard-fields.md`)

- Bare paths (e.g. `references/guide.md`) resolve from the skill root.
- `./` only when referencing a sibling file in the same directory.
- `{skill-root}` resolves to the installed skill directory.
- `{project-root}/...` resolves from the consumer project's working directory.
- `{skill-name}` resolves to the skill folder basename.
- Config variables (e.g. `{output_folder}`) already contain `{project-root}` — never
  double-prefix.

### "Outcome-driven" prompt style

A repeated quality dimension in the upstream builder
(`skills/bmad-agent-builder/references/quality-dimensions.md`,
`skills/bmad-workflow-builder/references/build-process.md`): SKILL.md should
describe **what each capability achieves**, not micromanage how. The persona/role
context dictates the *how*. We should match this idiom in synthpanel skills — short
SKILL.md bodies, lean references, no procedural over-specification.

---

## 2. Module manifest: `assets/module.yaml`

A module is a *bundle* of skills with shared identity. The manifest is small and
prose-friendly, not schema-heavy.

Concrete examples:

- `skills/module.yaml` (the BMad Builder module itself)
- `samples/sample-module-setup/assets/module.yaml`
- `samples/bmad-agent-dream-weaver/assets/module.yaml` (standalone variant)

Schema (verbatim from sample):

```yaml
code: dw                       # 2-4 letter module identifier
name: "Dream Weaver"           # human-friendly display name
description: "Dream journal, interpretation, and lucid dreaming coach"
module_version: 1.0.0
default_selected: false
module_greeting: >             # shown to user after setup
  Your dream space is ready. I'm Oneira...
```

Optional richer fields (per `skills/bmad-module-builder/references/create-module.md`):

- **Configuration variables** as top-level keys with `prompt`, `default`,
  `result`, plus optional `user_setting`, `single-select`, `multi-select`,
  `confirm`, `required`, `regex`, `example`, `directories`, `post-install-notes`.
- **`agents:` block** — array of agent metadata records (`code`, `name`, `title`,
  `icon`, `description`) for any agent-type skill in the module. Names may be
  the empty string if the agent learns its name at "First Breath" (see §4).

Example config variable (from `skills/module.yaml`):

```yaml
bmad_builder_output_folder:
  prompt: "Where should your custom output be saved?"
  default: "{project-root}/skills"
  result: "{project-root}/{value}"
```

**`{project-root}` is a literal token in config values** — never substituted at
write time. It is resolved by the consuming LLM at read time. This matters for
cross-machine portability and is repeated in every setup-skill SKILL.md.

---

## 3. Capability registry: `assets/module-help.csv`

Per-module CSV file listing every user-invocable capability. Columns
(`samples/bmad-agent-dream-weaver/assets/module-help.csv`,
`skills/module-help.csv`):

```
module, skill, display-name, menu-code, description, action, args,
phase, after, before, required, output-location, outputs
```

- `menu-code`: 1–3 letter shortcut, unique within module (`DL`, `DI`, `RT`, ...).
- `phase`: usually `anytime`, or workflow stage like `1-analysis`.
- `after`/`before`: dependency chain, format `skill-name:action` or `module:action`.
- `output-location`: a config variable name (e.g. `output_folder`), not a literal
  path — bmad-help resolves these at runtime.
- One row per capability, *not* per skill — a single skill can contribute many
  rows (Dream Weaver contributes 8 from one SKILL.md).

The first row of every module-help.csv is special:

```
<Module Name>,_meta,,,,,,,,,false,<docs URL>,
```

This `_meta` row points to the module's documentation. Reference:
`skills/module-help.csv` line 2.

---

## 4. Agent skill structure (sanctum pattern)

Agent skills follow a distinctive "Three Laws + Sacred Truth + Sanctum" pattern.
Concrete examples:

- `samples/bmad-agent-sentinel/SKILL.md`
- `samples/bmad-agent-creative-muse/SKILL.md`
- `samples/bmad-agent-dream-weaver/SKILL.md`

SKILL.md is a **lean bootloader** that:

1. Declares the agent's role/mission (~1 paragraph).
2. States The Three Laws (harm, obedience, self-preservation) — boilerplate copy
   across all three sample agents.
3. Declares The Sacred Truth: every session is a rebirth; agent identity lives
   in the sanctum, not the SKILL.md.
4. Routes activation by mode:
   - **No sanctum** → load `references/first-breath.md` (birth).
   - **`--headless`** → load `PULSE.md` from sanctum, execute, exit.
   - **Rebirth (default)** → batch-load 6 sanctum files: `INDEX.md`,
     `PERSONA.md`, `CREED.md`, `BOND.md`, `MEMORY.md`, `CAPABILITIES.md`.
5. Defines a Session Close discipline that writes a session log and curates
   memory.

Sanctum location is project-scoped:
`{project-root}/_bmad/memory/<skill-name>/`.

The skill ships **templates** for each sanctum file in `assets/`:
`PERSONA-template.md`, `CREED-template.md`, `BOND-template.md`, `MEMORY-template.md`,
`PULSE-template.md`, `INDEX-template.md`, `CAPABILITIES-template.md`. First Breath
copies these into the sanctum and personalizes them.

References (`references/*.md`) split out per-capability prompts and per-event
guidance: `first-breath.md`, `memory-guidance.md`, `capability-authoring.md`,
`risk-radar.md`, etc. (`samples/bmad-agent-sentinel/references/` has 10 of these.)

The agent-builder SKILL.md
(`skills/bmad-agent-builder/SKILL.md`) labels this as a spectrum:

- **Stateless agent** — everything in SKILL.md, no sanctum, no First Breath.
- **Memory agent** — bootloader SKILL.md + sanctum (6 files + First Breath).
- **Autonomous agent** — memory agent + PULSE for headless self-running.

---

## 5. Workflow skill structure

Workflow skills (e.g. `samples/bmad-excalidraw/`,
`samples/sample-module-setup/`) skip the persona machinery and instead organize
the SKILL.md by **stages**.

Pattern from `samples/bmad-excalidraw/SKILL.md`:

- Overview (3-part formula: What, How, Why/Outcome — see
  `references/standard-fields.md`).
- Activation Mode Detection (autonomous / yolo / guided).
- On Activation (load config, greet, detect intent, route).
- Stages table:

  | # | Stage | Purpose | Prompt |
  |---|-------|---------|--------|
  | 1 | Guided Design | ... | `references/guided-design.md` |
  | 2 | Generation | ... | `references/diagram-generation.md` |

Headless mode short-circuits to the generation stage. Each stage's deep prompt
lives in `references/<stage>.md` so the SKILL.md stays scannable.

### Customization surface (`customize.toml`)

Workflow skills *may* opt in to a customization surface so teams/users can
override behavior without forking. Reference:
`skills/bmad-workflow-builder/assets/customize-template.toml` and the SKILL
template at `skills/bmad-workflow-builder/assets/SKILL-template.md`.

Three layers, merged base → team → user:

- Base: `<skill>/customize.toml` (in the skill itself)
- Team: `{project-root}/_bmad/custom/{skill-name}.toml`
- Personal: `{project-root}/_bmad/custom/{skill-name}.user.toml`

Always-present keys when opted in: `activation_steps_prepend`,
`activation_steps_append`, `persistent_facts`. Workflow-specific scalars use
naming patterns `*_template`, `*_output_path`, `on_<event>`.

The merged result is read at runtime via
`{project-root}/_bmad/scripts/resolve_customization.py` and exposed in SKILL.md
prompts as `{workflow.<key>}`.

---

## 6. Setup skill: how a module installs itself

Every module ships a setup skill responsible for writing config and registering
help entries on first run. Two forms:

### Multi-skill module → dedicated `<code>-setup` skill

Example: `skills/bmad-bmb-setup/`,
`skills/bmad-module-builder/assets/setup-skill-template/`. The setup skill is a
sibling to the other skills. Its SKILL.md prescribes:

1. Read `./assets/module.yaml` — the `code` field is the module identifier.
2. Detect existing config / legacy per-module config.
3. Collect values via a single batched prompt; defaults come from
   existing > legacy > module.yaml.
4. Write a temp JSON answers file, then run two scripts in parallel:

```bash
python3 ./scripts/merge-config.py \
  --config-path "{project-root}/_bmad/config.yaml" \
  --user-config-path "{project-root}/_bmad/config.user.yaml" \
  --module-yaml ./assets/module.yaml \
  --answers <tempfile> \
  --legacy-dir "{project-root}/_bmad"
python3 ./scripts/merge-help-csv.py \
  --target "{project-root}/_bmad/module-help.csv" \
  --source ./assets/module-help.csv \
  --legacy-dir "{project-root}/_bmad" \
  --module-code <code>
```

5. Create configured output directories (`mkdir -p` resolved paths, but config
   values keep the literal `{project-root}` token).
6. Run `cleanup-legacy.py` to remove installer package directories from
   `_bmad/`.
7. Display `module_greeting` from `module.yaml`.

Both merge scripts use an **anti-zombie pattern**: existing entries for this
module are removed before fresh ones are written, so stale values never persist.

### Single-skill module → standalone self-registering

Example: `samples/bmad-agent-dream-weaver/`. Instead of a separate setup skill,
the registration logic is built into the agent skill itself via
`assets/module-setup.md` (a reference loaded when the user passes `setup` /
`configure` / `install`, or when no module config exists yet). Same scripts
(`merge-config.py`, `merge-help-csv.py`) live in the skill's own `scripts/`.

The decision rule (from
`skills/bmad-module-builder/references/create-module.md`): single skill →
standalone; multiple skills → setup skill. Either is overrideable.

---

## 7. Project runtime layout (`{project-root}/_bmad/`)

What an installed BMad project looks like at runtime:

```
{project-root}/
  _bmad/
    config.yaml             # shared, committed: per-module sections + core
    config.user.yaml        # gitignored: user_name, communication_language, user_setting flags
    module-help.csv         # aggregate of all installed modules' capability rows
    custom/                 # team/user overrides for customizable skills
      <skill-name>.toml
      <skill-name>.user.toml
    memory/<skill-name>/    # agent sanctums (per-skill)
    scripts/                # shared helpers, e.g. resolve_customization.py
  .claude/skills/           # actual installed skills (deployed by Claude plugin runtime)
```

Setup scripts cleanly delete legacy `_bmad/<module-code>/` and `_bmad/_config/`
once skills are confirmed at `.claude/skills/`. Reference:
`samples/sample-module-setup/SKILL.md` "Cleanup Legacy Directories".

---

## 8. Distribution: `.claude-plugin/marketplace.json`

Top-level repo file declaring distributable plugins. Reference:
`/tmp/bmad-study/bmad-builder/.claude-plugin/marketplace.json`.

Shape:

```json
{
  "name": "bmad-builder",
  "owner": { "name": "..." },
  "license": "MIT",
  "homepage": "...",
  "repository": "...",
  "keywords": ["bmad", "agent-builder", "..."],
  "plugins": [
    {
      "name": "bmad-builder",
      "source": "./",
      "description": "...",
      "version": "1.6.0",
      "author": { "name": "..." },
      "skills": [
        "./skills/bmad-agent-builder",
        "./skills/bmad-bmb-setup",
        "./skills/bmad-module-builder",
        "./skills/bmad-workflow-builder"
      ]
    },
    { "name": "sample-plugins", "skills": [ "./samples/..." ] },
    { "name": "bmad-dream-weaver-agent", "skills": [ "./samples/..." ] }
  ]
}
```

A single repo can ship multiple plugins, each pointing at a subset of skill
directories. The standalone-module scaffolder writes `marketplace.json` to
`<skill-parent>/.claude-plugin/marketplace.json` automatically (see
`skills/bmad-module-builder/references/create-module.md` step 7 standalone path).

---

## 9. What SynthPanel BMad copies vs invents

### Copy verbatim (don't reinvent)

- **Skill folder layout** — `SKILL.md`, `assets/`, `references/`, `scripts/`.
- **SKILL.md frontmatter** — only `name` and `description`, with the 2-part
  description format. Description triggers should name SynthPanel-specific
  phrases ("convene a panel", "run a deliberation").
- **`module.yaml` schema** — `code`, `name`, `description`, `module_version`,
  `default_selected`, `module_greeting`, plus optional config-variable blocks
  with `prompt`/`default`/`result`/`user_setting`. Use a 2–4 letter module
  `code` (proposal: `spb`).
- **`module-help.csv` columns** — exact column set, including the `_meta` first
  row pointing at SynthPanel docs.
- **Path-resolution rules** — bare paths from skill root; `./` only for
  sibling; `{project-root}/...` for project; `{skill-name}` and `{skill-root}`
  tokens.
- **Setup-skill flow** — anti-zombie merge, temp answers JSON, parallel
  `merge-config.py` + `merge-help-csv.py`, optional `cleanup-legacy.py`. We can
  copy the scripts directly from
  `skills/bmad-module-builder/assets/setup-skill-template/scripts/`.
- **`{project-root}` literal-token rule** — never substituted at write time.
- **Outcome-driven SKILL.md style** — short body, defer detail to
  `references/`, trust the executing LLM.
- **`marketplace.json`** at `.claude-plugin/marketplace.json`, listing the
  synthpanel plugin and its skills.

### Adapt to SynthPanel domain

- **Agent vs workflow choice.** A "synthetic deliberation panel" is plausibly a
  multi-agent ensemble — tempting to model each panelist as a separate agent
  skill with its own sanctum. But sanctum-per-panelist may be overkill if
  panels are ephemeral; a workflow skill that *spawns* personas inline (no
  persistent sanctum) is likely the right default. Decide per use case.
- **Stages.** If we ship a guided deliberation workflow, lay it out with the
  same stages-table idiom as `samples/bmad-excalidraw/SKILL.md`: e.g.
  `1-frame-question → 2-assemble-panel → 3-deliberate → 4-synthesize`.
- **Customization surface.** Panel composition (who's on the panel, in what
  proportion) is exactly the kind of thing teams will want to override without
  forking. Opt in to `customize.toml` from day one for any panel-running
  workflow skill, with `panel_template` / `synthesis_output_path` /
  `on_complete` scalars.
- **Help CSV menu codes.** Pick a coherent SynthPanel scheme (`CP` Convene
  Panel, `RD` Run Deliberation, `SY` Synthesize, etc.).
- **Module greeting & `_meta` docs URL.** Need our own.

### Invent (no upstream analogue)

- **Panelist persona library.** BMad's agent samples are single-persona
  bootloaders. SynthPanel needs a way to *parameterize* a workflow with a list
  of personas — likely a `panelists.yaml` (or array under `customize.toml`)
  living in `assets/` with persona records (`name`, `disposition`, `priors`,
  `voice`, ...). No upstream pattern for this; design fresh.
- **Deliberation transcript format.** Output artifacts in BMad samples are
  end-products (a diagram file, a journal entry). A deliberation produces a
  *structured transcript* — turn-by-turn with attribution, plus a synthesis.
  Define the transcript schema ourselves.
- **Cross-panel memory (optional).** If we want panels to learn across runs
  (e.g. "this panel previously concluded X"), invent a per-panel memory store.
  Could borrow the sanctum directory shape (`_bmad/memory/<skill>/`) but
  scoped per-panel-id, not per-skill.
- **Evaluation loop.** Panels need quality measurement (was the deliberation
  actually deliberative? did personas collapse to the same voice?). No upstream
  primitive — design a `quality-analysis.md`-style reference but adapted to
  panel transcripts rather than skill structure.

---

## 10. Concrete next steps for `synthpanel_bmad`

1. Pick module `code` (proposal: `spb`) and draft
   `synthpanel_bmad/skills/spb-setup/assets/module.yaml`.
2. Copy `skills/bmad-module-builder/assets/setup-skill-template/` as the
   starting point for `spb-setup` — the scripts are reusable as-is.
3. Decide on the first deliverable skill (likely a `convene-panel` workflow
   skill) and stub its SKILL.md using the
   `skills/bmad-workflow-builder/assets/SKILL-template.md` shape.
4. Define the panelist YAML schema as a new artifact (no upstream template).
5. Add `synthpanel_bmad/.claude-plugin/marketplace.json` listing the plugin.

---

## Appendix: Files referenced

All paths relative to the bmad-builder repo root unless noted.

- `samples/sample-module-setup/{SKILL.md, assets/module.yaml, assets/module-help.csv, scripts/}`
- `samples/bmad-agent-sentinel/{SKILL.md, assets/PERSONA-template.md, references/}`
- `samples/bmad-agent-creative-muse/SKILL.md`
- `samples/bmad-agent-dream-weaver/{SKILL.md, assets/module.yaml, assets/module-help.csv, assets/module-setup.md, scripts/}`
- `samples/bmad-excalidraw/SKILL.md`
- `skills/{module.yaml, module-help.csv}`
- `skills/bmad-bmb-setup/{SKILL.md, assets/, scripts/}`
- `skills/bmad-agent-builder/{SKILL.md, references/standard-fields.md (via workflow-builder)}`
- `skills/bmad-workflow-builder/{SKILL.md, references/standard-fields.md, assets/SKILL-template.md, assets/customize-template.toml}`
- `skills/bmad-module-builder/{SKILL.md, references/create-module.md, assets/setup-skill-template/, assets/standalone-module-template/}`
- `.claude-plugin/marketplace.json`
