# Chisel AI Coding Ruleset

A ruleset for AI coding agents (Claude Code, Cursor, etc.) working inside a Chisel WordPress theme. Ships with every new Chisel project and is read by the agent at the start of every task.

This README is for **humans maintaining the rules**. The agent itself reads [`CLAUDE.md`](CLAUDE.md) directly — it's the always-loaded entry point that carries the load-bearing rules and routes to the matching reference + skill in [`ai/`](ai/).

## What it covers

- Architecture: core/custom split, where hooks belong, where Twig lives.
- Design tokens, theme.json, design tool helpers.
- Block / pattern / ACF block / CPT scaffolding.
- Gutenberg content insertion via the `xfive-mcp` MCP server.
- Spacing, base styles, image handling.
- Three task modes — Figma URL, static asset/mockup, prompt-only.

## How a developer uses it

1. New Chisel project arrives with `CLAUDE.md`, the [`ai/`](ai/) folder, and [`ai-new-session-prompt.md`](ai-new-session-prompt.md) at the theme root.
2. Launch the agent from inside the theme directory (so the `chisel-*` skills auto-discover), then paste the contents of `ai-new-session-prompt.md` at the start of every new agent session and describe the task.
3. The agent loads `CLAUDE.md`, then the matching reference + skill for the task, and works in pause-and-review phases. Progress is tracked under an `ai-progress/` folder it creates at the theme root (an `INDEX.md` router → per-effort roadmap → per-phase files); on a new session it reads `ai-progress/INDEX.md` first to resume in-flight work.

The `xfive-mcp` WordPress plugin must be installed and its MCP server registered in the agent client — content insertion goes through MCP tools, not PHP seeds / WP-CLI / manual paste.

## Structure

```
CLAUDE.md                          # Entry point — always loaded; carries load-bearing rules + routing index
ai-new-session-prompt.md           # Paste-at-start prompt (phased workflow)
README.md                          # This file
ai-progress/                       # Agent-created at runtime (not shipped): INDEX + per-effort roadmap + per-phase files
ai/
├── rules/
│   ├── reference/                 # The "what" — descriptive facts
│   │   ├── file-locations.md
│   │   ├── design-tokens.md
│   │   ├── blocks.md
│   │   ├── section-mapping-decisions.md
│   │   ├── screen-build-order.md
│   │   ├── figma-import-template.md
│   │   ├── progress-template.md
│   │   ├── mcp-workflow.md
│   │   ├── coding-conventions.md
│   │   ├── twig-templating.md
│   │   ├── assets-and-scripts.md
│   │   ├── rest-api.md
│   │   └── woocommerce.md
│   └── templates/                 # Copy-paste code scaffolds
│       ├── pattern-markup.md
│       ├── custom-block-template.md
│       └── block-mod-template.md
└── skills/                        # The "how" — ordered procedures (auto-discovered slash commands)
    ├── chisel-figma-to-chisel/    #   Orchestrator for Figma imports
    ├── chisel-plan/               #   Creates progress files under ai-progress/ (INDEX + per-effort roadmap + per-phase files)
    ├── chisel-setup-theme-json/
    ├── chisel-theme-json/
    ├── chisel-adapt-base-styles/
    ├── chisel-adapt-header-footer/
    ├── chisel-create-pattern/
    ├── chisel-create-block/
    ├── chisel-create-acf-block/
    ├── chisel-extend-core-block/
    ├── chisel-create-component/
    ├── chisel-create-cpt/
    └── chisel-create-acf-options/
```

Each skill lives as `ai/skills/{name}/SKILL.md` — Claude Code auto-discovers them at session start and exposes each as a `/{name}` slash command (e.g. `/chisel-create-pattern`).

**Loading flow:** `CLAUDE.md` (always) → matching `reference/` doc (the "what") → matching `skills/` doc (the "how") → `templates/` (only when copying code).

Reference owns descriptive facts (file structures, decision ladders, token inventory). Skills focus on ordered procedure. Each skill routes back to its reference partner — load the reference first.

## Maintenance principles

When adding or editing rules, keep these in mind:

1. **Don't pin specific values.** The rules describe Chisel's **starter** state. Once a project adapts the theme (button padding, border-radius, color hex, spacer slugs), specific values drift. Documented values that go stale make the rules read like factual claims about current state and erode trust. Prefer: "open the file, compare to the spec, update what diverges" over "Chisel defaults are X / Y / Z." Only stable invariants belong in rules — protected slug *names* (e.g. `primary`, `tiny`), namespace conventions, file-layout patterns, build-pipeline mechanics.

2. **Reference vs skill.** If you're adding a fact (file list, decision table, token inventory) → reference. If you're adding a procedure (ordered steps to produce something) → skill. Each skill should link to its reference partner, and each reference should route back to the skill(s) that consume it.

3. **Auto-generated files.** Every `_index.scss` barrel in `src/styles/{components,blocks,objects,elements,utilities,generic,vendor,...}` is auto-generated by `chisel-scripts`. Never tell the agent to edit them by hand. `src/design/settings/_index.scss` is the exception — it's hand-edited.

4. **Hooks live in classes, not `functions.php`.** All `add_filter` / `add_action` calls belong in a `custom/app/WP/{Feature}.php` class using the `HooksSingleton` trait. `custom/functions.php` is a bootstrap list of `get_instance()` calls only. Any rule example showing hooks in `functions.php` is wrong.

5. **MCP-only content insertion.** Gutenberg content goes through `xfive-mcp-chisel` MCP tools. Never tell the agent to write PHP seed scripts, WP-CLI commands, or "ask the user to paste."

6. **Verify before asserting.** When adding a fact that names a specific file, class, function, or hook, open the file or grep `core/` to confirm it exists. Rules that name non-existent symbols silently break agents.

7. **Keep examples generic.** Templates and code examples should use `{block-name}` / `{slug}` placeholders, not project-specific names. The rules ship to every Chisel project — examples must be portable.

## Updating across projects

Rules are versioned with the Chisel starter theme. When a project runs `npm run update-chisel`, the `core/` folder is overwritten but **the `ai/` folder is part of the project layer** — project-specific edits to `ai/` are preserved. Updates to the canonical ruleset flow out to new projects on theme install (via the Chisel generator) and require manual merge for existing projects.

If you change a rule that's load-bearing (architecture, hook routing, content-insertion mechanics), bump the rules in `CLAUDE.md` and call it out in the Chisel changelog so existing projects can sync.

## Reporting issues

Real-world validation comes from running tasks against the rules and finding gaps. If an agent stumbles on a task (missing context, contradictory instructions, dead file reference), open an issue with:

- The task you gave the agent (verbatim from `ai-new-session-prompt.md` + task description)
- What the agent did wrong (or asked about that wasn't in the rules)
- Which rule doc(s) should cover it
