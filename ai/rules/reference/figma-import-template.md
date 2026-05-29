# Figma import progress (ai-progress/)

The format and procedure for progress files is owned by the **`chisel-plan` skill** — see [ai/skills/chisel-plan/SKILL.md](../../skills/chisel-plan/SKILL.md).

That skill covers:

- When to create progress files (thresholds)
- Files & layout (`ai-progress/INDEX.md` + per-effort `{effort-slug}-ROADMAP.md` + `{effort-slug}/phase-NN-{slug}.md`; standalone `task-{slug}.md`)
- One roadmap per effort/goal; never one monolithic file; never mixed modes
- Format templates (INDEX, roadmap with Figma-specific `## Source`, phase file, task file)
- Typical Figma single-screen and multi-screen phase sets
- Hard rules (roadmap = router with one-line outcomes, one status per phase, insert-don't-renumber, absolute dates, etc.)
- Procedure for create / update / session start + end
- Anti-patterns

The same layout serves every mode (Figma / static-asset / prompt) — see [progress-template.md](progress-template.md).
