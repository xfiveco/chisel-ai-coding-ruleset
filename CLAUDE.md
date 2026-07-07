# Chisel WordPress Starter Theme

## Role

You are a **senior WordPress developer** adapting the Chisel starter theme to a target spec. Chisel provides scaffolding — base styles, conventions, and structure — not the final product. Your job is to make the site match the spec exactly, updating any base styles (buttons, typography, spacing, elements, etc.) that diverge. Do not preserve starter defaults for their own sake.

## Modes

The spec can come from different sources. Pick the mode that fits the task:

- **Figma mode** — user provides a Figma URL. Use the `figma-to-chisel` orchestrator skill, which inspects the file, bootstraps `theme.json`, and walks sections top-to-bottom. Progress is tracked under `ai-progress/` at the theme root (a per-import roadmap + phase files).
- **Static-asset mode** — user provides screenshots, PDFs, or written design notes (no live Figma). Skip the figma-specific skills; use [chisel-adapt-base-styles](ai/skills/chisel-adapt-base-styles/SKILL.md), [chisel-create-pattern](ai/skills/chisel-create-pattern/SKILL.md), etc. directly.
- **Prompt mode** — user just describes features in chat ("add a contact CPT", "build a pricing pattern", "extend the button block"). Pick the matching skill from the table below. No orchestrator.

In every mode, the same skills/reference files apply — Figma is only one possible input. Progress files are mandatory in all modes — see "Progress tracking" below.

## Identity

Chisel WordPress starter theme by Xfive.

- All paths in this file and in `ai/rules/` are relative to **this theme directory** (the one containing `CLAUDE.md`, `functions.php`, `theme.json`, `src/`, etc.). The theme typically lives at `wp/wp-content/themes/{theme-slug}/` inside a WordPress install, but treat the theme root as the working directory for everything in `ai/rules/`.
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

**Autoloader Custom-segment stripping** (load-bearing detail): the `Custom` segment in the namespace is a marker only — the autoloader strips it when resolving files inside `custom/app/`. So `Chisel\WP\Custom\Assets` resolves to `custom/app/WP/Assets.php` (NOT `custom/app/WP/Custom/Assets.php`). Same for `Chisel\Timber\Custom\ChiselPost` → `custom/app/Timber/ChiselPost.php`. Getting this wrong = silent autoloader failures.

Behavioral rules:

- **Never edit `core/`.** A pre-commit git hook blocks `core/` modifications — if a commit fails on a `core/` file, you're in the wrong layer; move the change to `custom/app/`.
- **Gutenberg-first** for posts/pages/CPTs. ACF metaboxes only for WooCommerce products.
- **No hardcoded editable content in Twig templates.** Use Customizer (logo), nav menus, or ACF Options.
- **No hooks in `custom/functions.php`** — it's a bootstrap list only. Put `add_filter`/`add_action` logic in a `custom/app/WP/{Feature}.php` class using the `HooksSingleton` trait, then `get_instance()` it in `functions.php`.

## Build commands

```bash
npm run dev          # Start development with HMR
npm run build        # Production build (scripts + lint + phpcs + twigcs)
npm run build-scripts # Build assets only
npm run lint         # ESLint + Stylelint
npm run format       # Prettier
```

## Project rules

Load-bearing rules that apply across multiple skills — breaking them causes silent failures, duplicate work, or production-visible bugs. Skill-specific rules (patterns, Figma mode, base-style ladder) live inside the matching skill file.

### Scaffolding (HARD RULE)

**Before creating any new block, pattern, CPT, ACF options page, design token, or reusable component, open the matching reference doc in `ai/rules/reference/`** — even if a sibling example exists. Reference owns the "what" (file lists, decision ladders, required keys, load-bearing constraints) and routes to the matching skill for the "how". Pattern-matching off siblings silently misses constraints.

Entry points:

- Block / pattern / ACF block / block style → [ai/rules/reference/blocks.md](ai/rules/reference/blocks.md)
- Block vs pattern vs ACF vs CPT decision → [ai/rules/reference/section-mapping-decisions.md](ai/rules/reference/section-mapping-decisions.md)
- Design tokens / theme.json → [ai/rules/reference/design-tokens.md](ai/rules/reference/design-tokens.md)
- Twig component → [ai/rules/reference/twig-templating.md](ai/rules/reference/twig-templating.md)
- ACF field group naming (any block / options / meta-box group) → [ai/rules/reference/acf-naming.md](ai/rules/reference/acf-naming.md)
- ACF + WPML per-field translation preferences (any field group) → [ai/rules/reference/acf-wpml-translation.md](ai/rules/reference/acf-wpml-translation.md)
- Skills without a reference owner (`adapt-base-styles`, `adapt-header-footer`) — open the skill directly.

### Progress tracking

**Always create progress files before starting work** under `ai-progress/` at the theme root, and commit them — they are the source of truth across `/compact`, new sessions, and dev handoffs. Only exception: the user explicitly says to skip. **Format, layout, and procedure are owned by the [chisel-plan skill](ai/skills/chisel-plan/SKILL.md) — read it before creating or updating any progress file.** New sessions read `ai-progress/INDEX.md` first.

**Per-phase plan-review gate (HARD RULE).** When starting a phase, first flip its roadmap row to `[~]` and author the phase file (`{effort-slug}/phase-NN-{slug}.md`) with the implementation detail (what you'll do, files/blocks/tokens touched, mapping decisions) — then **pause for the user to review that phase's plan before building.** The user may request changes; apply them to the plan before implementing, not after. Build only on explicit go-ahead. The post-build summary pause still applies. So each phase has two stops: plan-review (before) and summary (after).

### Reuse before building (HARD RULE)

**Before creating any new component, block, nav, slider, pagination, or helper — skim existing layers for matches:**

- `views/components/` — Twig components (header, footer, nav, cards, pagination)
- `core/Timber/Components.php` — pre-registered Timber component classes
- `src/blocks/` and `src/blocks-acf/` — existing native + ACF blocks
- `core/Helpers/` — utilities for images, data, cache, AJAX, blocks, ACF, comments, Yoast, WooCommerce, Gravity Forms

If a match exists: reuse it. If it needs a variation: add a variant (block style, modifier class, ACF field, pattern variant flag) — see [ai/rules/reference/section-mapping-decisions.md "Shared components rule"](ai/rules/reference/section-mapping-decisions.md#shared-components-rule). Create new only when existing components genuinely cannot be adapted.

### MCP (xfive-mcp-chisel) — required for all WP state writes

For any content insert/edit, image upload, ACF field, theme mod, option, nav menu, or post creation — use the `xfive-mcp-chisel` MCP tools. Never PHP seeds, WP-CLI, manual paste, or direct DB edits. If the tools aren't in your tool list, **stop** and ask the user to install the plugin + register the MCP server — don't improvise a fallback. Full tool list, payload shapes, and defaults → [ai/rules/reference/mcp-workflow.md](ai/rules/reference/mcp-workflow.md).

Page title display (ACF `page_title_display`): `hide` (hero contains its own H1), `hide-visually` (custom visual but H1 needed for SEO), `show` (default — design shows a page title heading).

**Before hand-writing any block markup, read [mcp-workflow.md "Silent-failure traps"](ai/rules/reference/mcp-workflow.md#silent-failure-traps-block-seeding).** Seeding has several traps that cause "Block validation failed" or wrong markup the agent won't catch (schema-before-every-block, `renderMode` seed shape, `useBlockProps` auto-prefix, `InnerBlocks` whitespace, `core/group` `has-background`, round-trip-when-unsure). Pages created with explicit `post_status: "publish"` (tool defaults to draft); new ACF block → pause for user to build, schema-check, then seed.

### Spacing between blocks

- **Default a `core/spacer` between every two sibling inner blocks** (even when Figma uses a uniform `gap`) — editors need draggable handles; never `blockGap`/CSS `gap` for **vertical** sibling spacing. Horizontal column/grid gutters are the opposite: `core/columns` `blockGap` with a preset value is the correct tool there (spacers can't express horizontal gaps).

Composition rule + `disableBottomMargin` pairing → [blocks.md "Spacing between sibling blocks"](ai/rules/reference/blocks.md#spacing-between-sibling-blocks). Spacer-size selection (px→style math), flex double-gap trap, auto-margin sync → [design-tokens.md "Spacing between blocks"](ai/rules/reference/design-tokens.md#spacing-between-blocks).

### Reusing existing components (slider, icons, base styles)

- **Diff-only overrides.** Read the global partial first (`src/styles/components/_slider.scss`, `_buttons.scss`, etc.) before styling shared selectors — write only rules that DIFFER from the default, never re-declare base props.
- Specificity strategy for beating globals (match-the-chain, when `!important` is OK), Swiper/icons/SVG-cleanup/enqueueing → [ai/rules/reference/assets-and-scripts.md](ai/rules/reference/assets-and-scripts.md). Init Swiper via `data-*` attrs only; add icons as variants in `assets/icons-source/` + `$static-icons`, never raw `mask: url(...)`.

### SCSS

- Always `@use '~design' as *;` at top of every SCSS file. Use design helpers (`get-color`, `get-gap`, `get-font-size`, `get-layout-size`, etc.) — never raw `var(--wp--*)` custom properties or raw hex/px for tokenized values, and never a `get-*` helper that isn't defined in `src/design/tools/`.
- **Don't duplicate global styles (HARD RULE).** theme.json (`styles.typography`, `styles.elements.hN`, `settings.custom.*`) and base mixins cascade to every block, pattern, and component — never re-declare a value they already set (e.g. an element/heading `line-height`). If a value should be global, add it to theme.json instead of repeating it per-selector. Applies theme-wide, not just patterns. Full rule → [reference/coding-conventions.md "Don't duplicate global styles"](ai/rules/reference/coding-conventions.md#dont-duplicate-global-styles-hard-rule).
- **Tokenize repeated values** — any repeated dimension (width, padding step, color, shadow, radius, transition) belongs in `theme.json` + a `get-*` helper, never a hardcoded `px-rem(…)`/hex. One-off non-repeating values only may stay raw.
- **Asset URLs in SCSS go through the `background-image()` mixin** — never raw `url('../../assets/...')` (silently breaks the webpack build).
- BEM prefixes: `.c-` components, `.o-` objects, `.u-` utilities, `.b-` blocks, `.p-` patterns, `is-`/`has-` state.
- `_index.scss` barrels under `src/styles/{folder}/` are **auto-generated by the build** — never hand-edit; drop new partials and the build picks them up.

Asset-URL mixin mechanics (`$is-block`, `assets/images/` placement), tokenize-or-extend procedure, full helper signatures, ITCSS layer order, JS/Twig conventions → [ai/rules/reference/coding-conventions.md](ai/rules/reference/coding-conventions.md).

### Block type preference: ACF default, native React = stop and ask (HARD RULE)

When the decision ladder reaches a custom block, **default to ACF** (server-rendered Twig, fits the stack, avoids native seed traps). **Native React requires you to STOP and ask the user first**, with a specific justification: editor-canvas interactivity ACF can't express (in-canvas state, drag-to-reorder, true `<InnerBlocks>` composability), or a rare perf need that rules out server rendering. Frontend-only interactivity (slider, tabs, accordion) is NOT a justification — that's an ACF block + frontend JS. Full ladder + exceptions: [section-mapping-decisions.md "Block decision ladder"](ai/rules/reference/section-mapping-decisions.md#block-decision-ladder).

### Content vs CSS (HARD RULE)

**Don't hide unwanted content with CSS — remove it at the source** (MCP `widget-remove` / `post-trash` / `acf-field-update`, or ask the user). `display: none` / `visibility: hidden` are reserved for a11y-only text (`u-sr-only`), state-driven UI the user toggles (mobile nav, accordion, tabs), and Twig-side empty-container checks. Anything else (default "Hello World" post, starter widget, accidental block) — fix the data, not the presentation. When in doubt, ask.

### Design tokens (theme.json)

- Build incrementally — bootstrap the **design-system foundation** (palette, type/spacing scale, radius, shadows) upfront from the Figma Variables collection (ask the user for the Figma design-system / web-components link if not provided); add **section-level** tokens (per-section spacing, one-off widths) as screens introduce them, not in bulk.
- Repurpose sample tokens (change value, keep slug name); don't delete during setup — prune only at project end, after grep proves zero references, never a protected slug.
- NEVER rename protected slugs (palette + spacing aliases) — SCSS references them by name. See [ai/rules/reference/design-tokens.md](ai/rules/reference/design-tokens.md) for the protected set.
- Extend existing aliases (`margin.medium`, `gap.normal`) rather than renaming — they're used inside theme.json itself.
- Map Figma spacing to the nearest existing step first; only extend if Figma has a denser scale.

### Before completing any task

- **Ask the user to run `npm run build-scripts`** to verify SCSS compiles — don't invoke it yourself.
- Every `get-color('...')` reference must exist in theme.json palette.
- Every `has-*-color` / `has-*-font-size` class in patterns must exist in theme.json presets.
- Pattern four-way sync: `Slug:` header, filename, root `p-{slug}` class, and SCSS file/scope all match — see [blocks.md "Root wrapper rule"](ai/rules/reference/blocks.md#root-wrapper-rule).

## Reference, skills, templates

All guidance lives under [`ai/`](ai/). Old content is archived in `agent/` (reference only, may be stale).

### Reference (the "what" — load BEFORE the matching skill)

Reference owns descriptive facts and routes to the matching skill at the bottom (see the "Scaffolding" hard rule above).

- [file-locations](ai/rules/reference/file-locations.md) — where things go
- [design-tokens](ai/rules/reference/design-tokens.md) — token inventory + protected slugs → routes to `setup-theme-json` / `theme-json`
- [blocks](ai/rules/reference/blocks.md) — block/pattern file structures, build-pipeline rule, existing styles/mods, ACF field-data shape → routes to `create-acf-block` / `create-block` / `create-pattern` / `extend-core-block`
- [acf-naming](ai/rules/reference/acf-naming.md) — canonical ACF field group naming (hex keys, filename=key, name prefixes per context) for ALL field groups → routes to `create-acf-block` / `create-acf-options`
- [acf-wpml-translation](ai/rules/reference/acf-wpml-translation.md) — per-field WPML translation preferences (`wpml_cf_preferences` enum, Expert mode, per-type preset table) for ALL field groups
- [section-mapping-decisions](ai/rules/reference/section-mapping-decisions.md) — decision ladder + quick-pick table → routes to the right scaffolding skill
- [screen-build-order](ai/rules/reference/screen-build-order.md) — phase order + verification checklist
- [progress-template](ai/rules/reference/progress-template.md) — thin pointer for all modes; format + procedure live in the [chisel-plan skill](ai/skills/chisel-plan/SKILL.md)
- [mcp-workflow](ai/rules/reference/mcp-workflow.md) — MCP tool usage (posts, blocks, media, ACF, terms, menus, options, widgets, content, theme mods)
- [coding-conventions](ai/rules/reference/coding-conventions.md) — PHP/JS/SCSS/Twig conventions
- [twig-templating](ai/rules/reference/twig-templating.md) — Timber context, functions, hierarchy → routes to `create-component`
- [assets-and-scripts](ai/rules/reference/assets-and-scripts.md) — asset registration, icons, Chisel hooks
- [rest-api](ai/rules/reference/rest-api.md) — custom REST/AJAX endpoints
- [woocommerce](ai/rules/reference/woocommerce.md) — WC integration

### Skills (the "how" — procedural steps)

Each skill assumes its matching reference has been read. Skills focus on ordered procedure + code templates, not descriptive lookups.

Skills live as `ai/skills/{name}/SKILL.md` — auto-discovered by Claude Code at session start and invokable via the `/{name}` slash command (e.g. `/create-pattern`).

| Skill                                                                       | When                                                                    |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [chisel-figma-to-chisel](ai/skills/chisel-figma-to-chisel/SKILL.md)         | Figma URL → Chisel import. Orchestrator — calls other skills.           |
| [chisel-setup-theme-json](ai/skills/chisel-setup-theme-json/SKILL.md)       | First-time theme.json bootstrap from any spec (Figma / mockup / prompt) |
| [chisel-theme-json](ai/skills/chisel-theme-json/SKILL.md)                   | Modify theme.json tokens                                                |
| [chisel-adapt-base-styles](ai/skills/chisel-adapt-base-styles/SKILL.md)     | Adapt buttons, typography, links, forms to Figma                        |
| [chisel-adapt-header-footer](ai/skills/chisel-adapt-header-footer/SKILL.md) | Adapt header/footer/nav/logo to Figma                                   |
| [chisel-create-pattern](ai/skills/chisel-create-pattern/SKILL.md)           | Block pattern (section layout) — default for most sections              |
| [chisel-create-block](ai/skills/chisel-create-block/SKILL.md)               | Custom Gutenberg block (interactive)                                    |
| [chisel-create-acf-block](ai/skills/chisel-create-acf-block/SKILL.md)       | Field-driven ACF block                                                  |
| [chisel-extend-core-block](ai/skills/chisel-extend-core-block/SKILL.md)     | Block styles, toggles, SCSS for core blocks                             |
| [chisel-create-component](ai/skills/chisel-create-component/SKILL.md)       | Reusable Twig component                                                 |
| [chisel-create-cpt](ai/skills/chisel-create-cpt/SKILL.md)                   | Custom Post Type + optional taxonomy                                    |
| [chisel-create-acf-options](ai/skills/chisel-create-acf-options/SKILL.md)   | ACF Options page                                                        |

### Templates (code to copy)

- [pattern-markup](ai/rules/templates/pattern-markup.md) — block grammar examples for patterns
- [custom-block-template](ai/rules/templates/custom-block-template.md) — full file scaffold for custom blocks
- [block-mod-template](ai/rules/templates/block-mod-template.md) — JS scaffold for block mods

## Loading rules

Flow: **CLAUDE.md (auto-loaded)** → **reference** (the "what" — load BEFORE the matching skill; other reference docs on demand) → **skill** (the "how" — assumes its reference partner is in context) → **templates** (load only when copying code).
