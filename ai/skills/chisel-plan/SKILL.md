---
name: chisel-plan
description: Create and maintain a phased plan + progress file for multi-session work (Figma imports, theme rebuilds, multi-CPT features, large refactors). Produces a single file at the theme root that is the source of truth across sessions, /compact boundaries, and dev handoffs.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Plan + Progress

One file at the theme root (the directory containing `CLAUDE.md`, `functions.php`, `theme.json`), organized around **phases**. The phase is the spine — every checklist item, artifact, decision, and blocker hangs off the phase it belongs to. No parallel checklists elsewhere in the file (causes drift).

## When to create

**Always, unless the user explicitly says to skip it.** Mode-agnostic — Figma, static-asset, and prompt modes all get a progress file. Even a one-phase "add one block" task gets one: a one-phase file is cheap, and the next session (or `/compact`) makes it valuable retroactively.

The only signals to skip:

- User says "no plan file" / "just do it" / similar.
- Pure read-only task (answer a question, explain a file) where nothing will be written.

When in doubt, create it. Easier to delete a file you didn't need than to reconstruct context that was never recorded.

## Filename

- **Figma-driven import** → `FIGMA_IMPORT_PROGRESS.md`
- **Anything else** (theme rebuild, multi-CPT feature, large refactor, content migration) → `PROGRESS.md`

The shape is the same; only the `## Source` block and the typical phase set differ.

## Hard rules

1. **Phases are the spine.** No parallel checklists outside `## Phases`. A "Theme setup checklist" sitting next to the phase list will drift — fold its checkboxes into the phase that produces them.
2. **One status per phase.** Use `[ ] todo` / `[~] in-progress` / `[x] done` / `[!] blocked`. Don't track status in prose.
3. **Re-scoping inserts, not renumbers.** If new work appears mid-flight, add `Phase 7a` between Phase 7 and Phase 8. Never renumber phases that were already completed or referenced in past session log entries.
4. **Absolute dates only.** "Tuesday" → `2026-05-26`. The file is read across sessions; relative dates rot.
5. **Append to session log every working session.** One line per session, even if "no progress". Future-you needs to know where the trail ends.
6. **Trust the file over recollection.** When in doubt, the file wins; update your assumptions, not the file.

## Format

```markdown
# {Project / Screen} Progress

<!-- Source of truth across sessions. Both agents and devs edit it.
     When in doubt, this file wins. -->

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

- **In scope:** {bulleted list of what the work covers}
- **Out of scope:** {explicit exclusions — defers, "not this iteration"}
- **Locked decisions:** {key answers from scope/plan questions — palette overhaul, ACF vs pattern for X, wideSize=1296, etc.}

## Phases

### Phase N — {one-line title}  `[ ] | [~] | [x] | [!]`

- **Goal:** one sentence.
- **Touches:** files, blocks, tokens, MCP tools.
- **Checklist:**
  - [ ] sub-step
  - [ ] sub-step
- **Artifacts produced:** {pattern slugs, block names, CPTs, theme.json deltas}
- **Notes / blockers:** {optional — only if non-empty}

(repeat per phase)

## Shared / cross-phase

- {Item used by multiple phases — e.g. shared component, recurring decoration treatment, font file added once and used everywhere}

## Deferred / follow-ups

- {item} — {why deferred, what unblocks it}

## Session log

- {YYYY-MM-DD} — {dev or "agent"} — {what was accomplished, next step}
```

## Mode-specific guidance

### Figma mode — typical phase set

For a single screen, phases generally look like:

1. **Scope** — sections inventoried, decisions locked
2. **Inspect + tokens proposal** — `get_metadata` + `get_variable_defs`, draft theme.json deltas, create this progress file
3. **theme.json bootstrap** — apply deltas, download fonts
4. **Adapt base styles** — buttons, typography, links, forms, body
5. **Header + footer** — Twig + nav menu
6. **Create page + set as front page** (MCP)
7..N. **Section patterns / blocks** — one phase per top-to-bottom section
N+1. **Verification** — build-scripts, browser pass, screen-build-order checklist

Multi-screen imports add a `## Screens` array at the top of `## Source` and reuse Phases 7+ per screen, suffixed `7-home`, `7-about`, etc.

### Non-Figma mode — typical phase set

Tailor to the work. Defaults:

- **Greenfield feature** — design tokens → content model (CPTs) → view model (Twig context, helpers) → templates → editor experience (block styles, ACF) → styling → behavior (JS) → build verification
- **Rebuild** — audit current state → token migration → base style migration → component-by-component port → URL-by-URL parity check → cutover
- **Large refactor** — inventory current usages → introduce new abstraction → migrate callers in batches → remove old abstraction → cleanup

If unsure, see the [feature-building sequence](../../rules/reference/section-mapping-decisions.md#recommended-feature-building-sequence) for the canonical default.

## Procedure

### Creating the file (first time)

1. Confirm thresholds met (see "When to create" above). If not, don't create — work from conversation context.
2. Ask the user any open scope questions (in/out of scope, locked decisions) before writing.
3. Write the file at the theme root using the format above.
4. Draft Phase 1 (Scope) fully, then Phases 2..N as titles + one-line goals only — flesh out each phase's checklist when you reach it.
5. Append the first session log entry.

### Updating mid-work

- **Starting a phase:** flip its status to `[~]`. Add checklist items if Phase was created as a title-only stub.
- **Completing a checklist item:** flip `[ ]` → `[x]` immediately. Don't batch.
- **Completing a phase:** flip its status to `[x]`. Add the artifacts produced. Append a session log line.
- **Hitting a blocker:** flip status to `[!]`, add a `Notes / blockers` bullet on the phase explaining what's blocked and what unblocks it.
- **Discovering new work mid-flight:** insert `Phase Na` (don't renumber). If the new phase is a prerequisite for an already-started phase, document the dependency in `Notes / blockers`.

### At session end

Append a session log line, even if the phase isn't done:

```
- 2026-05-26 — agent — Phase 4 done (button mixins, form fields, block styles). Phase 5 next; need Figma context on header (8192:613) + footer (8192:753).
```

Bias toward "what's next" so the next session can resume cold.

### Re-reading at session start

1. Read `## Source` and `## Scope` for context.
2. Find the first phase whose status is `[~]` or `[ ]` after a `[x]` — that's where to resume.
3. Read its checklist + notes before doing any new work.
4. If anything in the file conflicts with what you remember: trust the file, update your assumptions.

## Output

When you create or update the plan, briefly summarize:

- File created / updated: path
- Phases added / completed / blocked
- Open questions for the user

## Anti-patterns

- ❌ A "checklist" section parallel to the phases. (Drifts. Fold into phases.)
- ❌ Tracking individual files touched outside the relevant phase. (Belongs in `Touches:` line of the phase.)
- ❌ Multiple "session log" sections. (One, append-only, bottom of file.)
- ❌ Renumbering phases after a re-scope. (Breaks references in past session log entries.)
- ❌ Using the file as a TODO dumping ground. (`Deferred / follow-ups` is for items unblocked by external action; everything else belongs in the phase that handles it.)
- ❌ Relative dates ("yesterday", "next Tuesday"). (Use `YYYY-MM-DD`.)
