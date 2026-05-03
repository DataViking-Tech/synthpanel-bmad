# Stage 6 — Persist (EP)

Save the extension verdict under the parent's run_id lineage.

## Path shape

`{workflow.verdict_output_path}` for EP defaults to:

> `{project-root}/_bmad/synthpanel/verdicts/{run_id}-ext-{n}.json`

Where:

- `{run_id}` is the **parent** `run_id` (the prior run being extended), NOT
  a fresh ID generated for the extension.
- `{n}` is incremented per extension off the same parent: first extension
  is `-ext-1`, second is `-ext-2`, etc.

The MCP server's response may include its own `run_id` for the extension
itself; persist that on the JSON as `meta.extension_run_id`, but the file
path uses the parent lineage so all extensions land near their parent on disk.

## Resolving `{n}`

Read the directory once at persist time:

```
ls verdicts/{parent_run_id}-ext-*.json
```

`n` = (count of existing matches) + 1. Atomic at file-creation time; if two
EP runs race the same parent, the second's mkdir+write may collide — retry
once with the bumped `n`.

## index.json — the lineage map

Maintain an index linking extensions to their parents:

```json
{
  "{parent_run_id}": {
    "parent_card": "cards/{parent_run_id}.md",
    "extensions": [
      {"n": 1, "verdict": "verdicts/{parent_run_id}-ext-1.json", "card": "cards/{parent_run_id}-ext-1.md", "follow_up": "<verbatim follow_up_question>"},
      {"n": 2, "verdict": "verdicts/{parent_run_id}-ext-2.json", "card": "cards/{parent_run_id}-ext-2.md", "follow_up": "<verbatim follow_up_question>"}
    ]
  }
}
```

Path: `{workflow.extension_index_path}` (default
`{project-root}/_bmad/synthpanel/verdicts/index.json`).

Read-modify-write atomically. If the index doesn't exist, create it with the
single parent entry. Don't fail the workflow on index-write issues — surface
and continue with the verdict + card written.

## Card path

Same lineage shape: `cards/{parent_run_id}-ext-{n}.md`. Stage 7 renders the
card; Stage 6 writes it atomically.

## Workflow warnings (EP-specific)

Stamp these into `meta.workflow_warnings` (workflow-level, NOT inside
`extension[]`):

- **Orphan extension** — prior_run_id resolved locally but the server
  returned `unknown run`. Persona continuity is no longer guaranteed; the
  server re-realized the panel.
- **Schema-version drift** between the parent verdict and the extension's
  `schema_version`.
- **Pack override at extension time** (only possible if customize.toml's
  inheritance flags were flipped at the team level).

## On_complete hook

Same shape as RP/QP `persist.md`. Adds environment variable
`SPB_PARENT_RUN_ID` for the hook's convenience.

## Exit criteria

Extension verdict + card written under the parent lineage; index.json
updated. Hand off to Stage 7.
