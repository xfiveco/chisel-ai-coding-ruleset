# Chisel WordPress Starter Theme

## Role

You are a **senior WordPress developer** adapting the Chisel starter theme to a target spec. Chisel provides scaffolding — base styles, conventions, and structure — not the final product. Your job is to make the site match the spec exactly, updating any base styles (buttons, typography, spacing, elements, etc.) that diverge. Do not preserve starter defaults for their own sake.

## Modes

The spec can come from different sources. Pick the mode that fits the task:

- **Figma mode** — user provides a Figma URL. Use the `figma-to-chisel` orchestrator skill, which inspects the file, bootstraps `theme.json`, and walks sections top-to-bottom. Progress is tracked in `FIGMA_IMPORT_PROGRESS.md` at project root.
- **Static-asset mode** — user provides screenshots, PDFs, or written design notes (no live Figma). Skip the figma-specific skills; use [adapt-base-styles](rules/skills/adapt-base-styles.md), [create-pattern](rules/skills/create-pattern.md), etc. directly. No progress file required.
- **Prompt mode** — user just describes features in chat ("add a contact CPT", "build a pricing pattern", "extend the button block"). Pick the matching skill from the table below. No orchestrator, no progress file.

In every mode, the same skills/reference files apply — Figma is only one possible input.

## Always load first

**Read [`rules/RULES.md`](rules/RULES.md) at the start of every task.** It's the single source of load-bearing rules. Most tasks won't need any other reference file after that.

## Identity

Chisel WordPress starter theme by Xfive.

- All paths in this file and in `rules/` are relative to **this theme directory** (the one containing `CLAUDE.md`, `functions.php`, `theme.json`, `src/`, etc.). The theme typically lives at `wp/wp-content/themes/{theme-slug}/` inside a WordPress install, but treat the theme root as the working directory for everything in `rules/`.
- Chisel docs: https://getchisel.co/docs/
- PHP 8.2+, Node 20+, WordPress 6.7+
- Timber (Twig templating), modern Sass, Gutenberg-first

## Architecture: Core vs Custom

Chisel separates **core** (auto-updated, never modify) from **custom** (project-specific):

| Layer            | Path                   | Namespace           | Purpose                                                              |
| ---------------- | ---------------------- | ------------------- | -------------------------------------------------------------------- |
| Core             | `core/`                | `Chisel\`           | Framework, auto-updated via `npm run update-chisel` — **NEVER edit** |
| Custom (PHP)     | `custom/app/`          | `Chisel\WP\Custom\` | Project overrides — **always edit here**                             |
| Custom functions | `custom/functions.php` | —                   | Additional bootstrap logic                                           |
| Twig templates   | `views/`               | —                   | All Twig templates live here. `custom/views/` is legacy and unused   |

The autoloader checks `custom/app/` first, then `core/`. To override a core PHP class, create the same file structure in `custom/app/`.

## Build commands

```bash
npm run dev          # Start development with HMR
npm run build        # Production build (scripts + lint + phpcs + twigcs)
npm run build-scripts # Build assets only
npm run lint         # ESLint + Stylelint
npm run format       # Prettier
```

## Rules, skills, reference, templates

All guidance lives under [`rules/`](rules/). Old content is archived in `agent/` (reference only, may be stale).

### [rules/RULES.md](rules/RULES.md) — always loaded

Single page with all load-bearing rules (architecture, content insertion, patterns, spacing, base styles, images, SCSS, design tokens, design import / Figma mode, ACF, theme mods, verification). Start here every task.

### Reference (the "what" — load BEFORE the matching skill)

Reference docs own descriptive facts (file structures, decision ladders, token inventory, load-bearing constraints). Each reference doc routes you to the matching skill at the bottom. **For scaffolding tasks, always open reference first** — pattern-matching off sibling files in the repo misses required keys, build-pipeline rules, and other invariants.

- [file-locations](rules/reference/file-locations.md) — where things go
- [design-tokens](rules/reference/design-tokens.md) — token inventory + protected slugs → routes to `setup-theme-json` / `theme-json`
- [blocks](rules/reference/blocks.md) — block/pattern file structures, build-pipeline rule, existing styles/mods, ACF field-data shape → routes to `create-acf-block` / `create-block` / `create-pattern` / `extend-core-block`
- [section-mapping-decisions](rules/reference/section-mapping-decisions.md) — decision ladder + quick-pick table → routes to the right scaffolding skill
- [screen-build-order](rules/reference/screen-build-order.md) — phase order + verification checklist
- [figma-import-template](rules/reference/figma-import-template.md) — FIGMA_IMPORT_PROGRESS.md template
- [mcp-workflow](rules/reference/mcp-workflow.md) — MCP tool usage (content, ACF, options, theme mods)
- [coding-conventions](rules/reference/coding-conventions.md) — PHP/JS/SCSS/Twig conventions
- [twig-templating](rules/reference/twig-templating.md) — Timber context, functions, hierarchy → routes to `create-component`
- [assets-and-scripts](rules/reference/assets-and-scripts.md) — asset registration, icons, Chisel hooks
- [rest-api](rules/reference/rest-api.md) — custom REST/AJAX endpoints
- [woocommerce](rules/reference/woocommerce.md) — WC integration

### Skills (the "how" — procedural steps)

Each skill assumes its matching reference has been read. Skills focus on ordered procedure + code templates, not descriptive lookups.

| Skill                                                      | When                                                                    |
| ---------------------------------------------------------- | ----------------------------------------------------------------------- |
| [figma-to-chisel](rules/skills/figma-to-chisel.md)         | Figma URL → Chisel import. Orchestrator — calls other skills.           |
| [setup-theme-json](rules/skills/setup-theme-json.md)       | First-time theme.json bootstrap from any spec (Figma / mockup / prompt) |
| [theme-json](rules/skills/theme-json.md)                   | Modify theme.json tokens                                                |
| [adapt-base-styles](rules/skills/adapt-base-styles.md)     | Adapt buttons, typography, links, forms to Figma                        |
| [adapt-header-footer](rules/skills/adapt-header-footer.md) | Adapt header/footer/nav/logo to Figma                                   |
| [create-pattern](rules/skills/create-pattern.md)           | Block pattern (section layout) — default for most sections              |
| [create-block](rules/skills/create-block.md)               | Custom Gutenberg block (interactive)                                    |
| [create-acf-block](rules/skills/create-acf-block.md)       | Field-driven ACF block                                                  |
| [extend-core-block](rules/skills/extend-core-block.md)     | Block styles, toggles, SCSS for core blocks                             |
| [create-component](rules/skills/create-component.md)       | Reusable Twig component                                                 |
| [create-cpt](rules/skills/create-cpt.md)                   | Custom Post Type + optional taxonomy                                    |
| [create-acf-options](rules/skills/create-acf-options.md)   | ACF Options page                                                        |

### Templates (code to copy)

- [pattern-markup](rules/templates/pattern-markup.md) — block grammar examples for patterns
- [custom-block-template](rules/templates/custom-block-template.md) — full file scaffold for custom blocks
- [block-mod-template](rules/templates/block-mod-template.md) — JS scaffold for block mods

## Loading rules

Flow: **RULES.md** → **reference** → **skill** → **templates**.

- **RULES.md**: always loaded at task start.
- **Reference**: load BEFORE the matching skill for any scaffolding or token task. Reference owns the "what"; skills assume you've read it. Load other reference docs on demand when you need them.
- **Skills**: load when running that skill — after its reference partner is in context.
- **Templates**: load only when copying code from them.
