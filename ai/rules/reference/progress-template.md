# Progress files (ai-progress/)

The format and procedure for progress files is owned by the **`chisel-plan` skill** — see [ai/skills/chisel-plan/SKILL.md](../../skills/chisel-plan/SKILL.md).

That skill covers:

- When to create progress files (thresholds)
- Files & layout (`ai-progress/INDEX.md` + per-effort `{effort-slug}-ROADMAP.md` + `{effort-slug}/phase-NN-{slug}.md`; standalone one-off work → `task-{slug}.md`)
- One roadmap per effort/goal (theme rebuild, multi-CPT feature, large refactor, content migration); never one monolithic file; never mixed modes
- Format templates (INDEX, roadmap with mode-appropriate `## Source`, phase file, task file)
- Typical phase sets (greenfield feature, rebuild, refactor)
- Hard rules (roadmap = router with one-line outcomes, one status per phase, insert-don't-renumber, absolute dates, etc.)
- Procedure for create / update / session start + end
- Anti-patterns

For Figma imports specifically, see [figma-import-template.md](figma-import-template.md).
