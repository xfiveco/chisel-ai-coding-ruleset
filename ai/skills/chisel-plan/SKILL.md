---
name: chisel-plan
description: Create and maintain phased progress files for multi-session work (Figma imports, theme rebuilds, multi-CPT features, large refactors). Produces an ai-progress/ folder (INDEX + per-effort roadmap + per-phase files) at the theme root that is the source of truth across sessions, /compact boundaries, and dev handoffs.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Plan + Progress

Progress lives in an `ai-progress/` folder at the theme root (the directory containing `CLAUDE.md`, `functions.php`, `theme.json`), organized around **phases**. The phase is the spine — every checklist item, artifact, decision, and blocker hangs off the phase it belongs to. The roadmap routes; per-phase detail lives in its own file so new sessions stay lean (see "Files & layout").

## When to create

**Always, unless the user explicitly says to skip it.** Mode-agnostic — Figma, static-asset, and prompt modes all get progress files. Even a one-phase "add one block" task gets one (a `task-{slug}.md`): it's cheap, and the next session (or `/compact`) makes it valuable retroactively.

The only signals to skip:

- User says "no plan file" / "just do it" / similar.
- Pure read-only task (answer a question, explain a file) where nothing will be written.

When in doubt, create it. Easier to delete a file you didn't need than to reconstruct context that was never recorded.

## Files & layout

All progress lives under an `ai-progress/` folder at the theme root — **never one monolithic file**. The `ai-` prefix on the folder marks it agent-maintained and keeps it grouped; files inside need no prefix. This keeps each new session lean: the agent reads the small index + roadmap to see what's done, and opens a phase's full plan only when it works that phase.

```
ai-progress/
  INDEX.md                       # router — every effort + standalone task, with status. Always read first.
  {effort-slug}-ROADMAP.md       # one per multi-phase effort: Source + Scope + phase table + Shared + Deferred + Session log
  {effort-slug}/
    phase-NN-{slug}.md           # one expanded phase plan each (NN zero-padded: 01, 02, …)
  task-{slug}.md                 # standalone single-phase work — plan + summary inline, no roadmap
```

### Effort vs task (which to create)

- **Multi-phase effort** (Figma import, theme rebuild, multi-CPT feature, large refactor) → a `{effort-slug}-ROADMAP.md` + a `{effort-slug}/` folder of phase files. Add a line to `INDEX.md`.
- **Single-phase task** ("add one block", "extend a pattern") → a single `ai-progress/task-{slug}.md` (plan + summary inline). Add a line to `INDEX.md`. No roadmap, no folder.

### One roadmap per effort (not per project)

A roadmap is scoped to **one goal in one mode**. A project accumulates efforts over time (Figma import now, a prompt-driven feature later) — each is its own roadmap/task, listed in `INDEX.md`. **Never mix modes in one roadmap** and never append an unrelated new goal as extra phases on a finished roadmap. This is how Figma↔prompt mixing across sessions stays clean: different goals → different files.

The phase **shape** is identical across all modes (Figma / static-asset / prompt); only the roadmap's `## Source` block and typical phase set differ.

## Hard rules

1. **The roadmap is the spine; phase files are its expansions.** The roadmap's `## Phases` table is the single status surface — every phase has exactly one row there with its status + a one-line outcome + a link to its phase file. The phase file holds the expanded plan (goal, touches, checklist, mapping, artifacts, notes). No parallel checklists outside this structure. **The one-line outcome on the roadmap row is mandatory** — it's what lets a future session understand a `[x]` phase without opening its file. Links are one-way (roadmap → phase file); a phase file never needs to duplicate the roadmap.
2. **One status per phase.** Use `[ ] todo` / `[~] in-progress` / `[x] done` / `[!] blocked`. Don't track status in prose.
3. **Re-scoping inserts, not renumbers.** If new work appears mid-flight, add `Phase 7a` between Phase 7 and Phase 8. Never renumber phases that were already completed or referenced in past session log entries.
4. **Absolute dates only.** "Tuesday" → `2026-05-26`. The file is read across sessions; relative dates rot.
5. **Append to session log every working session.** One line per session, even if "no progress". Future-you needs to know where the trail ends.
6. **Trust the file over recollection.** When in doubt, the file wins; update your assumptions, not the file.

## Format

### `INDEX.md` (router — always read first)

Thin and stable. One line per effort/task: status, link, mode, one-phrase state.

```markdown
# Progress Index

<!-- Router for all agent progress. Open the relevant roadmap/task; don't load everything. -->

- `[~]` [Home Figma import](home-import-ROADMAP.md) — Figma · 8 sections · on Phase 6
- `[x]` [Contact form](task-contact-form.md) — prompt · done 2026-06-02
- `[ ]` [About page](about-import-ROADMAP.md) — Figma · not started
```

### `{effort-slug}-ROADMAP.md` (multi-phase effort)

Holds everything cross-phase (always loaded when working this effort) plus the phase table — but **not** the expanded phase plans, which live in the phase files.

```markdown
# {Effort name} — Roadmap

<!-- Source of truth across sessions. When in doubt, this file wins.
     Per-phase detail lives in {effort-slug}/phase-NN-*.md — open only the phase you're working. -->

## Source

<!-- Figma mode -->

- Mode: Figma
- File key: `{fileKey}`
- File name: {Figma file name}
- URL: {full URL}
- Root node: `{nodeId}` ({screen name})

<!-- Non-Figma mode -->

- Mode: {static-assets | prompt | rebuild | migration}
- Spec source: {screenshots in /docs, Linear epic FOO-123, prompt in chat, etc.}
- Started: {YYYY-MM-DD}
- Owner(s): {dev names, or "team"}

## Scope

- **In scope:** {what the work covers}
- **Out of scope:** {explicit exclusions}
- **Locked decisions:** {key answers — palette overhaul, ACF vs pattern for X, wideSize=1296, etc.}

## Phases

<!-- One row per phase: status + one-line outcome + link. NO expanded plan here. -->

- `[x]` **Phase 1 — Scope** — sections inventoried, decisions locked. → [phase-01-scope.md]({effort-slug}/phase-01-scope.md)
- `[~]` **Phase 2 — theme.json** — _in progress_. → [phase-02-theme-json.md]({effort-slug}/phase-02-theme-json.md)
- `[ ]` **Phase 3 — Hero section** — not started. → [phase-03-hero.md]({effort-slug}/phase-03-hero.md)

## Shared / cross-phase

- {Item used by multiple phases — shared component, recurring treatment, font added once and reused}

## Deferred / follow-ups

- {item} — {why deferred, what unblocks it}

## Session log

- {YYYY-MM-DD} — {dev or "agent"} — {what was accomplished, next step}
```

### `{effort-slug}/phase-NN-{slug}.md` (one expanded phase plan)

Written when the phase is **started** (the plan-review gate). This is the detail a future session skips for done phases.

```markdown
# Phase NN — {title}

> Roadmap: [{effort-slug}-ROADMAP.md](../{effort-slug}-ROADMAP.md) · Status: `[ ] | [~] | [x] | [!]`

- **Goal:** one sentence.
- **Touches:** files, blocks, tokens, MCP tools.
- **Plan / checklist:**
  - [ ] sub-step
  - [ ] sub-step
- **Artifacts produced:** {pattern slugs, block names, CPTs, theme.json deltas}
- **Notes / blockers:** {optional — only if non-empty}
```

### `task-{slug}.md` (standalone single-phase task)

Roadmap + phase folded into one small file — no separate roadmap.

```markdown
# Task — {title}

<!-- Standalone single-phase task. Listed in INDEX.md. -->

- **Mode:** {prompt | static-assets} · **Started:** {YYYY-MM-DD} · **Status:** `[ ] | [~] | [x] | [!]`
- **Goal / scope:** what this does, what's out of scope.
- **Plan / checklist:**
  - [ ] sub-step
- **Artifacts produced:** {…}
- **Session log:**
  - {YYYY-MM-DD} — agent — {what happened, next step}
```

## Mode-specific guidance

### Figma mode — typical phase set

For a single screen, phases generally look like:

1. **Scope** — sections inventoried, decisions locked
2. **Inspect + tokens proposal** — `get_metadata` + `get_variable_defs`, draft theme.json deltas, create the roadmap + INDEX row
3. **theme.json bootstrap** — apply deltas, download fonts
4. **Adapt base styles** — buttons, typography, links, forms, body
5. **Header + footer** — Twig + nav menu
6. **Create page + set as front page** (MCP)
   7..N. **Section patterns / blocks** — one phase per top-to-bottom section
   N+1. **Verification** — build-scripts, browser pass, screen-build-order checklist

Multi-screen imports: prefer **one roadmap per screen** (e.g. `home-import-ROADMAP.md`, `about-import-ROADMAP.md`), sharing the theme.json/base-style phases by reference (do those once, note in the second roadmap's Scope that they're already done). If the screens are tightly coupled and you keep them in one roadmap, suffix per-screen section phases (`7-home`, `7-about`) and list each screen in `## Source`.

### Non-Figma mode — typical phase set

Tailor to the work. Defaults:

- **Greenfield feature** — design tokens → content model (CPTs) → view model (Twig context, helpers) → templates → editor experience (block styles, ACF) → styling → behavior (JS) → build verification
- **Rebuild** — audit current state → token migration → base style migration → component-by-component port → URL-by-URL parity check → cutover
- **Large refactor** — inventory current usages → introduce new abstraction → migrate callers in batches → remove old abstraction → cleanup

If unsure, see the [feature-building sequence](../../rules/reference/section-mapping-decisions.md#recommended-feature-building-sequence) for the canonical default.

## Procedure

### Creating progress (first time for an effort/task)

1. Confirm thresholds met (see "When to create" above). If not, don't create — work from conversation context.
2. Ask the user any open scope questions (in/out of scope, locked decisions) before writing.
3. Ensure `ai-progress/INDEX.md` exists (create it if this is the project's first effort), and add a row for this effort/task.
4. **Multi-phase effort:** write `ai-progress/{effort-slug}-ROADMAP.md` (Source + Scope + Session log + a `## Phases` table of rows — status + title + link only, no expanded plans yet). Create the `{effort-slug}/` folder. Do NOT pre-write all phase files — each phase file is authored when that phase is started (the plan-review gate).
   **Single-phase task:** write `ai-progress/task-{slug}.md` (plan + summary inline). No roadmap/folder.
5. Append the first session log entry (in the roadmap, or the task file's log).

### Updating mid-work

- **Starting a phase (plan-review gate):** flip its roadmap row to `[~]`, then **author `{effort-slug}/phase-NN-{slug}.md`** with the expanded plan (goal, touches, checklist, mapping, notes). Pause for user review before building.
- **Completing a checklist item:** flip `[ ]` → `[x]` in the **phase file** immediately. Don't batch.
- **Completing a phase:** flip the **roadmap row** to `[x]` and write its **one-line outcome** there (mandatory — see Hard rule 1); set the phase file's status + artifacts. Append a session log line to the roadmap.
- **Hitting a blocker:** flip status to `[!]` on the roadmap row, add a `Notes / blockers` bullet in the phase file explaining what's blocked and what unblocks it.
- **Discovering new work mid-flight:** insert `Phase Na` (don't renumber) — new roadmap row + new `phase-Na-*.md`. If it's a prerequisite for an already-started phase, note the dependency.
- **A whole new goal appears** (different mode or unrelated feature): don't bolt phases onto this roadmap — create a new effort/task per "One roadmap per effort" and add an INDEX row.

### At session end

Append a session log line to the active roadmap (or task file), even if the phase isn't done:

```
- 2026-05-26 — agent — Phase 4 done (button mixins, form fields, block styles). Phase 5 next; need Figma context on header (8192:613) + footer (8192:753).
```

Bias toward "what's next" so the next session can resume cold. Also update the INDEX row's one-phrase state (e.g. `on Phase 5`).

### Re-reading at session start

1. Read `ai-progress/INDEX.md` — pick the active/relevant effort or task.
2. Open its roadmap; read `## Source`, `## Scope`, and the phase table. The `[x]` rows' one-line outcomes tell you what's done — **don't open completed phase files.**
3. Find the first phase row that is `[~]`, or `[ ]` after a `[x]` — open **only that** `phase-NN-*.md` and read its checklist + notes.
4. If anything conflicts with what you remember: trust the files, update your assumptions.

## Output

When you create or update the plan, briefly summarize:

- File(s) created / updated: path(s)
- Phases added / completed / blocked
- Open questions for the user

## Anti-patterns

- ❌ One monolithic progress file. (Use `ai-progress/` — INDEX + per-effort roadmap + per-phase files.)
- ❌ Expanded phase plans inside the roadmap. (Roadmap rows are status + one-line outcome + link only; detail lives in the phase file.)
- ❌ A `[x]` roadmap row with no one-line outcome. (Forces future sessions to reopen the phase file — defeats the split.)
- ❌ Pre-writing all phase files up front. (Author each at its plan-review gate, when the phase starts.)
- ❌ Appending an unrelated new goal as extra phases on an existing roadmap. (New effort → new roadmap/task + INDEX row.)
- ❌ Mixing modes (Figma + prompt) in one roadmap. (One goal, one mode, per roadmap.)
- ❌ A "checklist" section parallel to the phase table. (Drifts. Fold into the phase file.)
- ❌ Multiple "session log" sections. (One per roadmap/task, append-only.)
- ❌ Renumbering phases after a re-scope. (Breaks references in past session log entries — insert `Na` instead.)
- ❌ Relative dates ("yesterday", "next Tuesday"). (Use `YYYY-MM-DD`.)
