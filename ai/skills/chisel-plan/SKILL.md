---
name: chisel-plan
description: Create and maintain phased progress files for multi-session work (Figma imports, theme rebuilds, multi-CPT features, large refactors). Produces an ai-progress/ folder (INDEX + FINDINGS + a per-task changes/{NN}-{task-name}/ folder holding its own roadmap, session log, and phase files) at the theme root that is the source of truth across sessions, /compact boundaries, and dev handoffs.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Plan + Progress

Progress lives in an `ai-progress/` folder at the theme root (the directory containing `CLAUDE.md`, `functions.php`, `theme.json`), organized around **phases**. Each task gets its own self-contained folder under `ai-progress/changes/{NN}-{task-name}/` holding its roadmap, session log, and phase files. The phase is the spine — every checklist item, artifact, decision, and blocker hangs off the phase it belongs to. The roadmap routes; per-phase detail lives in its own file so new sessions stay lean (see "Files & layout").

## When to create

**Always, unless the user explicitly says to skip it.** Mode-agnostic — Figma, static-asset, and prompt modes all get progress files. Even a one-phase "add one block" task gets a folder (with a single-phase roadmap): it's cheap, and the next session (or `/compact`) makes it valuable retroactively.

The only signals to skip:

- User says "no plan file" / "just do it" / similar.
- Pure read-only task (answer a question, explain a file) where nothing will be written.
- The work qualifies for **quick-fix mode** (feedback on already-built work, existing files only, no new artifact, no cross-cutting decision — see the [chisel-quick-fix skill](../chisel-quick-fix/SKILL.md)). No new folder; if the fix belongs to a task that already has a `changes/{NN}-{task-name}/` folder, append one session-log line there (`LOG.md`, or the single-phase roadmap's inline log — e.g. `2026-07-13 — agent — QA fixes: header contrast, hero spacing`). Orphan fixes (no related task folder) get no progress entry.

When in doubt, create it. Easier to delete a folder you didn't need than to reconstruct context that was never recorded.

## Files & layout

All progress lives under an `ai-progress/` folder at the theme root — **never one monolithic file**. The `ai-` prefix marks it agent-maintained and keeps it grouped; files inside need no prefix. Each task is a self-contained folder under `changes/`, so everything for one effort is co-located and a new session reads only the small index + that task's roadmap, opening a phase's full plan only when it works that phase.

```
ai-progress/
  INDEX.md                       # router — every task grouped Active/Done, plus a link to FINDINGS. Always read first.
  FINDINGS.md                    # project-level, append-only. Incidental bugs/observations noticed during any task.
  changes/
    {NN}-{task-name}/            # one folder per task; NN = build-order number, zero-padded (01, 02, 03 …)
      ROADMAP.md                 # Source + Scope + phase table + Shared + Deferred (multi-phase); or plan + log inline (single-phase)
      LOG.md                     # append-only session log — MULTI-PHASE efforts only
      phase-NN-{slug}.md         # one expanded phase plan each (NN zero-padded: 01, 02, …) — multi-phase only
```

**Folder numbering.** Prefix each task folder with a zero-padded sequential number in the order tasks are started — `01-home-import`, `02-contact-form`, `03-about-page` — so `changes/` sorts in build order and any task is easy to find. Assign the next free number at creation (highest existing + 1); never renumber an existing folder (it would break INDEX links and cross-effort references). The phase-file `NN` is a separate, per-task counter — don't conflate it with the folder number.

### Effort vs single-phase task (what goes in the folder)

Both live at `changes/{NN}-{task-name}/`. The distinction is only what's inside:

- **Multi-phase effort** (Figma import, theme rebuild, multi-CPT feature, large refactor) → `ROADMAP.md` (Source + Scope + phase table) + `LOG.md` + one `phase-NN-{slug}.md` per phase. Add an INDEX row under `## Active`.
- **Single-phase task** ("add one block", "extend a pattern") → just `ROADMAP.md` with the plan, checklist, and session log folded inline. No `LOG.md`, no phase files. Add an INDEX row. Don't spawn near-empty files for trivial work.

### One roadmap per task (not per project)

A roadmap is scoped to **one goal in one mode**. A project accumulates tasks over time (Figma import now, a prompt-driven feature later) — each is its own `changes/{NN}-{task-name}/` folder, listed in `INDEX.md`. **Never mix modes in one roadmap** and never append an unrelated new goal as extra phases on a finished roadmap. This is how Figma↔prompt mixing across sessions stays clean: different goals → different folders.

The phase **shape** is identical across all modes (Figma / static-asset / prompt); only the roadmap's `## Source` block and typical phase set differ.

### Completed tasks stay put

When a task finishes, it **stays in `changes/`** — only its INDEX row moves from `## Active` to `## Done`. Don't relocate folders on completion: INDEX links and cross-effort reuse references (a later roadmap citing an earlier one's shared components) point into the folder, and moving it dangles them. There is no `archive/` folder; status lives in INDEX, not in the filesystem path.

## Hard rules

1. **The roadmap is the spine; phase files are its expansions.** The roadmap's `## Phases` table is the single status surface — every phase has exactly one row there with its status + a one-line outcome + a link to its phase file (in the same folder). The phase file holds the expanded plan (goal, touches, checklist, mapping, artifacts, notes). No parallel checklists outside this structure. **The one-line outcome on the roadmap row is mandatory** — it's what lets a future session understand a `[x]` phase without opening its file. Links are one-way (roadmap → phase file); a phase file never needs to duplicate the roadmap.
2. **One status per phase.** Use `[ ] todo` / `[~] in-progress` / `[x] done` / `[!] blocked`. Don't track status in prose.
3. **Re-scoping inserts, not renumbers.** If new work appears mid-flight, add `Phase 7a` between Phase 7 and Phase 8. Never renumber phases that were already completed or referenced in past session log entries.
4. **Absolute dates only.** "Tuesday" → `2026-05-26`. The files are read across sessions; relative dates rot.
5. **Append to the session log every working session.** One line per session, even if "no progress" — in `LOG.md` (multi-phase) or the roadmap's inline `## Session log` (single-phase). Future-you needs to know where the trail ends.
6. **Incidental findings go to `FINDINGS.md`, not the phase file.** A bug or oddity you notice mid-phase — especially one unrelated to the active task — gets one line in `ai-progress/FINDINGS.md`; then keep going. Don't derail the current phase or bury the finding where nobody triages it.
7. **Trust the files over recollection.** When in doubt, the files win; update your assumptions, not the files.

## Format

### `INDEX.md` (router — always read first)

Thin and stable. One line per task: status, link, mode, one-phrase state. Grouped `## Active` / `## Done` so the live list stays short as the project grows.

```markdown
# Progress Index

<!-- Router for all agent progress. Open the relevant task's ROADMAP; don't load everything.
     Incidental bugs/observations → FINDINGS.md. -->

Findings: [FINDINGS.md](FINDINGS.md)

## Active

- `[~]` [Home Figma import](changes/01-home-import/ROADMAP.md) — Figma · 8 sections · on Phase 6
- `[ ]` [About page](changes/03-about-page/ROADMAP.md) — Figma · not started

## Done

- `[x]` [Contact form](changes/02-contact-form/ROADMAP.md) — prompt · done 2026-06-02
```

### `FINDINGS.md` (project-level, append-only)

One triage surface for everything noticed in passing across all tasks. Newest at bottom. Each line: date · type · what · where it surfaced · status/suggested action. Type tags are loose (`bug`, `note`, `perf`, `a11y`, `security`).

```markdown
# Findings

<!-- Incidental bugs, oddities, observations noticed during any task — including ones unrelated
     to the active work. Jot one line and keep going; don't derail the current phase.
     The user triages from here. Newest at bottom. -->

- 2026-07-10 · bug · Mobile nav emits duplicate `aria-current` — surfaced during Home import (Phase 5). Unrelated to task; not fixed.
- 2026-07-10 · note · `custom/app/WP/Assets.php` enqueues an unused legacy stylesheet — surfaced during Contact form task. Cleanup candidate.
```

### `changes/{NN}-{task-name}/ROADMAP.md` — multi-phase effort

Holds everything cross-phase (always loaded when working this task) plus the phase table — but **not** the expanded phase plans (they live in the phase files) and **not** the session log (it lives in `LOG.md`).

```markdown
# {Task name} — Roadmap

<!-- Source of truth across sessions. When in doubt, this file wins.
     Per-phase detail → phase-NN-*.md (open only the phase you're working).
     Session history → LOG.md. -->

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

<!-- One row per phase: status + one-line outcome + link (same folder). NO expanded plan here. -->

- `[x]` **Phase 1 — Scope** — sections inventoried, decisions locked. → [phase-01-scope.md](phase-01-scope.md)
- `[~]` **Phase 2 — theme.json** — _in progress_. → [phase-02-theme-json.md](phase-02-theme-json.md)
- `[ ]` **Phase 3 — Hero section** — not started. → [phase-03-hero.md](phase-03-hero.md)

## Shared / cross-phase

- {Item used by multiple phases — shared component, recurring treatment, font added once and reused}

## Deferred / follow-ups

- {item} — {why deferred, what unblocks it}
```

### `changes/{NN}-{task-name}/LOG.md` — session log (multi-phase only)

Append-only. Split out of the roadmap so the roadmap stays a fixed-size router — you only reopen history when you need it.

```markdown
# {Task name} — Session log

<!-- Append-only. One line per working session, even "no progress". Newest at bottom.
     Bias toward "what's next" so the next session resumes cold. -->

- 2026-05-26 — agent — Phase 4 done (button mixins, form fields, block styles). Phase 5 next; need Figma context on header (8192:613) + footer (8192:753).
```

### `changes/{NN}-{task-name}/phase-NN-{slug}.md` — one expanded phase plan

Written when the phase is **started** (the plan-review gate). This is the detail a future session skips for done phases.

```markdown
# Phase NN — {title}

> Roadmap: [ROADMAP.md](ROADMAP.md) · Log: [LOG.md](LOG.md) · Status: `[ ] | [~] | [x] | [!]`

- **Goal:** one sentence.
- **Touches:** files, blocks, tokens, MCP tools.
- **Plan / checklist:**
  - [ ] sub-step
  - [ ] sub-step
- **Artifacts produced:** {pattern slugs, block names, CPTs, theme.json deltas}
- **Notes / blockers:** {optional — only if non-empty}
```

### `changes/{NN}-{task-name}/ROADMAP.md` — single-phase task

Roadmap + phase + log folded into one small file — no `LOG.md`, no phase files.

```markdown
# {Task name} — Roadmap

<!-- Single-phase task. Listed in INDEX.md. Plan + log inline; no phase files, no LOG.md. -->

- **Mode:** {prompt | static-assets} · **Started:** {YYYY-MM-DD} · **Status:** `[ ] | [~] | [x] | [!]`
- **Goal / scope:** what this does, what's out of scope.
- **Plan / checklist:**
  - [ ] sub-step
- **Artifacts produced:** {…}

## Session log

- {YYYY-MM-DD} — agent — {what happened, next step}
```

## Mode-specific guidance

### Figma mode — typical phase set

For a single screen, phases generally look like:

1. **Scope** — sections inventoried, decisions locked
2. **Inspect + tokens proposal** — `get_metadata` + `get_variable_defs`, draft theme.json deltas, create the task folder + roadmap + INDEX row
3. **theme.json bootstrap** — apply deltas, download fonts
4. **Adapt base styles** — buttons, typography, links, forms, body
5. **Header + footer** — Twig + nav menu
6. **Create page + set as front page** (MCP)
   7..N. **Section patterns / blocks** — one phase per top-to-bottom section
   N+1. **Verification** — build-scripts, browser pass, screen-build-order checklist

Multi-screen imports: prefer **one task folder per screen** (e.g. `changes/01-home-import/`, `changes/02-about-import/`), sharing the theme.json/base-style phases by reference (do those once, note in the second roadmap's Scope that they're already done). If the screens are tightly coupled and you keep them in one folder, suffix per-screen section phases (`7-home`, `7-about`) and list each screen in `## Source`.

### Non-Figma mode — typical phase set

Tailor to the work. Defaults:

- **Greenfield feature** — design tokens → content model (CPTs) → view model (Twig context, helpers) → templates → editor experience (block styles, ACF) → styling → behavior (JS) → build verification
- **Rebuild** — audit current state → token migration → base style migration → component-by-component port → URL-by-URL parity check → cutover
- **Large refactor** — inventory current usages → introduce new abstraction → migrate callers in batches → remove old abstraction → cleanup

If unsure, see the [feature-building sequence](../../rules/reference/section-mapping-decisions.md#recommended-feature-building-sequence) for the canonical default.

## Procedure

### Creating progress (first time for a task)

1. Confirm thresholds met (see "When to create" above). If not, don't create — work from conversation context.
2. Ask the user any open scope questions (in/out of scope, locked decisions) before writing.
3. Ensure `ai-progress/INDEX.md` and `ai-progress/FINDINGS.md` exist (create them if this is the project's first task), and add an INDEX row under `## Active`.
4. Create the `ai-progress/changes/{NN}-{task-name}/` folder.
   **Multi-phase effort:** write `ROADMAP.md` (Source + Scope + a `## Phases` table of rows — status + title + link only, no expanded plans yet) and `LOG.md` (first session entry). Do NOT pre-write phase files — each is authored when its phase starts (the plan-review gate).
   **Single-phase task:** write `ROADMAP.md` only (plan + checklist + inline session log).

### Updating mid-work

- **Starting a phase (plan-review gate):** flip its roadmap row to `[~]`, then **author `phase-NN-{slug}.md`** with the expanded plan (goal, touches, checklist, mapping, notes). Pause for user review before building.
- **Completing a checklist item:** flip `[ ]` → `[x]` in the **phase file** immediately. Don't batch.
- **Completing a phase:** flip the **roadmap row** to `[x]` and write its **one-line outcome** there (mandatory — see Hard rule 1); set the phase file's status + artifacts. Append a session log line to `LOG.md`.
- **Noticing an incidental bug/oddity:** add one line to `ai-progress/FINDINGS.md` (Hard rule 6) and keep going — do not expand it into the phase file.
- **Hitting a blocker:** flip status to `[!]` on the roadmap row, add a `Notes / blockers` bullet in the phase file explaining what's blocked and what unblocks it.
- **Discovering new work mid-flight:** insert `Phase Na` (don't renumber) — new roadmap row + new `phase-Na-*.md`. If it's a prerequisite for an already-started phase, note the dependency.
- **A whole new goal appears** (different mode or unrelated feature): don't bolt phases onto this roadmap — create a new `changes/{NN}-{task-name}/` folder per "One roadmap per task" and add an INDEX row.
- **Finishing the whole task:** move its INDEX row from `## Active` to `## Done` with the done date. Leave the folder in place.

### At session end

Append a session log line to the active task's `LOG.md` (or the single-phase roadmap's inline log), even if the phase isn't done:

```
- 2026-05-26 — agent — Phase 4 done (button mixins, form fields, block styles). Phase 5 next; need Figma context on header (8192:613) + footer (8192:753).
```

Bias toward "what's next" so the next session can resume cold. Also update the INDEX row's one-phrase state (e.g. `on Phase 5`).

### Re-reading at session start

1. Read `ai-progress/INDEX.md` — pick the active/relevant task from `## Active`.
2. Open its `changes/{NN}-{task-name}/ROADMAP.md`; read `## Source`, `## Scope`, and the phase table. The `[x]` rows' one-line outcomes tell you what's done — **don't open completed phase files.**
3. Find the first phase row that is `[~]`, or `[ ]` after a `[x]` — open **only that** `phase-NN-*.md` and read its checklist + notes. Open `LOG.md` only if you need the recent trail.
4. If anything conflicts with what you remember: trust the files, update your assumptions.

## Output

When you create or update the plan, briefly summarize:

- File(s) created / updated: path(s)
- Phases added / completed / blocked
- Open questions for the user

## Anti-patterns

- ❌ One monolithic progress file. (Use `ai-progress/` — INDEX + FINDINGS + per-task `changes/{NN}-{task-name}/` folders.)
- ❌ Roadmap files at the `ai-progress/` root. (Every task's roadmap lives inside its own `changes/{NN}-{task-name}/` folder.)
- ❌ Expanded phase plans inside the roadmap. (Roadmap rows are status + one-line outcome + link only; detail lives in the phase file.)
- ❌ A `[x]` roadmap row with no one-line outcome. (Forces future sessions to reopen the phase file — defeats the split.)
- ❌ Pre-writing all phase files up front. (Author each at its plan-review gate, when the phase starts.)
- ❌ Session log inside the roadmap for a multi-phase effort. (It grows unboundedly — put it in `LOG.md` so the roadmap stays a fixed-size router.)
- ❌ Burying an incidental/unrelated finding in a phase file. (One line in `FINDINGS.md`, then keep going.)
- ❌ Moving a completed task folder to an archive/ path. (Breaks INDEX + reuse links; mark it Done in INDEX instead.)
- ❌ Appending an unrelated new goal as extra phases on an existing roadmap. (New task → new folder + INDEX row.)
- ❌ Mixing modes (Figma + prompt) in one roadmap. (One goal, one mode, per roadmap.)
- ❌ A "checklist" section parallel to the phase table. (Drifts. Fold into the phase file.)
- ❌ Renumbering phases after a re-scope. (Breaks references in past session log entries — insert `Na` instead.)
- ❌ Relative dates ("yesterday", "next Tuesday"). (Use `YYYY-MM-DD`.)
