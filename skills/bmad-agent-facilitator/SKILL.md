---
name: bmad-agent-facilitator
description: Quinn — the SynthPanel Panel Facilitator. Coaches decision frame, picks panel size and persona pack, reads the verdict back honestly. Use when the user says 'convene a panel', 'run a deliberation', 'I'm stuck on a decision', 'should I use synthpanel', or asks for a 'panel readout'.
---

# Panel Facilitator (Quinn)

> **Status:** stub. Filled in by IMPL-2 (`spb-impl-2`).
>
> Canonical persona content lives in `design-notes/facilitator-persona.md` (DESIGN-2).

This skill bootloads Quinn, the Panel Facilitator agent for SynthPanel. The full
persona — identity, voice, trigger heuristics, frame coaching, panel-size and
pack selection rules, verdict readback style, principles — is to be ported from
`design-notes/facilitator-persona.md` per IMPL-2.

The agent skill follows the BMad agent pattern (DESIGN-1 §4): lean SKILL.md
bootloader plus `references/*.md` for per-capability prompts plus an optional
sanctum at `{project-root}/_bmad/memory/bmad-agent-facilitator/`. Whether Quinn
ships as a stateless agent, memory agent, or autonomous agent is an IMPL-2
decision.

`customize.toml` exposes Quinn's overrideable scalars (principles, persistent
facts, panel-pack defaults) — also IMPL-2.
