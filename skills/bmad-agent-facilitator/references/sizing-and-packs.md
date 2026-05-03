# Sizing & Persona Pack Selection

Loaded by Quinn (`bmad-agent-facilitator`) at Stage 3 (Assemble) of any
convene-* workflow, and when sizing or pack selection is in question.

Quinn picks panel size and pack from the *shape* of the decision, not user
request alone. Defaults must be defendable at readout time.

---

## 1. Decision-class → panel-size mapping

| Decision class | Default size | Workflow |
|---|---|---|
| Quick gut-check, low-stakes copy / naming shortlist | 5–10 personas | `spb-convene-quick-poll` (QP) |
| Pricing, positioning, narrative arc, launch lede | 12–20 personas | `spb-convene-panel` (RP) |
| High-consequence strategic / compliance / hiring | 20+ personas | `spb-convene-panel` (RP) — bump size for headroom |
| Follow-up after a verdict raised a flag or new question | varies | `spb-extend-panel` (EP) — preserves persona continuity |

`customize.toml` overrides the per-class defaults (`default_quick_poll_size`,
`default_panel_size`).

---

## 2. The `small_n` floor map

The `small_n` flag (spec §3) fires when the realized panel size is below the
floor mapped from `decision_being_informed`. A shortlist-narrowing call at
n=8 may be fine; a hire/fire decision at n=8 is not.

Floors come from `customize.toml [small_n_floors]`:

| Decision class | Default floor |
|---|---|
| copy_naming | 5 |
| pricing | 12 |
| positioning | 12 |
| policy | 14 |
| hiring | 18 |
| strategic | 18 |

**Quinn surfaces the floor at convene time, not as a surprise flag at
readout.** Example:

> "For a hiring-class decision I'd run n≥18 or you'll trip the `small_n`
> flag — want to bump the size or accept the flag in the readback?"

If the user accepts the flag, Quinn convenes anyway and stamps the flag
explicitly in the readback. We don't silently up-size to avoid a flag — the
flag is information.

---

## 3. Persona pack selection

Pick the pack that matches the **audience**, not the asker.

### Three rules

1. **Audience ≠ panel.** If the user is an indie dev choosing pricing for
   indie devs, the pack is `developer` / `indie-builders`, not "self."
   Picking your own cohort is the most common failure mode — "audience =
   panel = self" is a degenerate case Quinn declines:
   > "Panels are for informing decisions where you're not the audience.
   > This is a 'try it' call, not a panel call."

2. **Mismatch surfaces as `out_of_distribution`** (spec §3). If the only
   available pack doesn't cover the audience, run anyway and explicitly
   call the limit at readout — *do not* silently substitute a closer-but-
   wrong pack.

3. **Quota drift gets named.** If realized quotas drift >15% from
   requested, `demographic_skew` fires (spec §3). Quinn reports the drifted
   cell explicitly with realized vs requested numbers; doesn't paper over it.

### Multi-audience decisions

When a decision has two distinct audiences (e.g. "name that wins indies
*and* enterprise buyers"), Quinn convenes **two panels**, not one. Names
that win on recall in one cohort often lose in another; collapsing them
into one mixed panel destroys signal.

Quinn surfaces this at Stage 3:

> "If you have a secondary audience in mind, we'd run a second panel on that
> pack — names that win in one cohort often lose in another."

---

## 4. Pack discovery (open question)

`customize.toml` exposes `default_pack` and `panel_template` as static
overrides. Until the MCP server exposes a `list-packs` capability, Quinn
relies on:

1. `default_pack` from customize.toml (team or user level)
2. The `pack_id` the user names explicitly
3. Inference from the audience phrase ("indie devs" → `indie-builders` pack
   if it exists; otherwise `out_of_distribution` at readout)

If pack inference fails ambiguously, Quinn names the available packs and
asks. Do NOT guess on ambiguous pack names — wrong-pack runs are worse than
no-pack runs because they LOOK valid.

---

## 5. Convene-time scripts (sizing)

Stock phrases Quinn uses to surface size mechanics inline. Adapt voice as
needed; do not invent numbers without backing in customize.toml or this map.

**For a hiring-class decision:**
> "Hiring-class — I'd run n≥18 to clear the `small_n` floor. Bump to n=24
> for headroom?"

**For a copy / naming shortlist:**
> "Copy / naming is quick-poll-shaped — n=8 default. The readback will
> explicitly say it's not ship-blocking signal."

**For pricing:**
> "Pricing class — default n=16 panel. Below n=12 trips `small_n`."

**When user requests n>10 for a quick-poll workflow:**
> "n=12+ is panel-shaped, not quick-poll. Want to switch to `convene-panel`?
> The menu code is part of the audit story — we don't silently re-route."

**When user requests n>40:**
> "n=40+ is unusual. Worth running, but check whether the `extension`-shape
> answer is a `convene-panel` at 24 followed by an `extend-panel` on the
> cleavage — usually cheaper and more informative."

---

## 6. Sizing for `extend-panel`

`extend-panel` defaults to `additional_size=0` — re-poll the same personas
on the new question. Persona continuity is the entire reason to use EP
instead of a fresh `run_panel`; deviating from it defeats the purpose.

`additional_size > 0` is allowed when:
- The cleavage-driving cell is undersampled and a broader panel would
  resolve the dissent shape.
- The follow-up question expands the decision class (e.g. moved from
  "shortlist of 4" to "shortlist of 4 + a wildcard").

Quinn surfaces this at Stage 3 of EP:

> "Extension defaults to the same panel — same personas, new question.
> Want me to broaden it (`additional_size > 0`)? Only worth it if you
> think the dissenting cell was undersampled."

---

## 7. What sizing-and-packs does NOT decide

- The actual quota composition. Quotas are spec §3-defined
  (gender/age/region/profession cells) and come from the pack metadata.
  Quinn surfaces the realized quotas at readout but does not propose
  custom quota mixes — that's an explicit user override, not a default.
- Whether to run a panel. That's `references/triggers.md`.
- The verdict shape. That's `references/readback.md`.
