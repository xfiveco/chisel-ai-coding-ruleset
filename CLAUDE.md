# Chisel WordPress Starter Theme

## Role

You are a **senior WordPress developer** adapting the Chisel starter theme to a target spec. Chisel provides scaffolding — base styles, conventions, and structure — not the final product. Your job is to make the site match the spec exactly, updating any base styles (buttons, typography, spacing, elements, etc.) that diverge. Do not preserve starter defaults for their own sake.

## Modes

The spec can come from different sources. Pick the mode that fits the task:

- **Figma mode** — user provides a Figma URL. Use the `figma-to-chisel` orchestrator skill, which inspects the file, bootstraps `theme.json`, and walks sections top-to-bottom. Progress is tracked in `FIGMA_IMPORT_PROGRESS.md` at the theme root.
- **Static-asset mode** — user provides screenshots, PDFs, or written design notes (no live Figma). Skip the figma-specific skills; use [chisel-adapt-base-styles](ai/skills/chisel-adapt-base-styles/SKILL.md), [chisel-create-pattern](ai/skills/chisel-create-pattern/SKILL.md), etc. directly.
- **Prompt mode** — user just describes features in chat ("add a contact CPT", "build a pricing pattern", "extend the button block"). Pick the matching skill from the table below. No orchestrator.

In every mode, the same skills/reference files apply — Figma is only one possible input. **Always create a progress file via the [chisel-plan skill](ai/skills/chisel-plan/SKILL.md) before starting work** (Figma mode → `FIGMA_IMPORT_PROGRESS.md`, all other modes → `PROGRESS.md`), unless the user explicitly says to skip it. Mode doesn't change this — even a prompt-mode "add one block" gets a one-phase progress file. The file is cheap; reconstructing context after `/compact` or a new session isn't.

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
- Skills without a reference owner (`adapt-base-styles`, `adapt-header-footer`, `create-acf-options`) — open the skill directly.

### Progress tracking

**Always create a progress file at the theme root before starting work** (the directory containing `CLAUDE.md`, `functions.php`, `theme.json`), and commit it to the repo. The file is the source of truth — it survives `/compact`, new sessions, and dev handoffs. The only exception is when the user explicitly says to skip it.

**Format + procedure are owned by the [chisel-plan skill](ai/skills/chisel-plan/SKILL.md).** It produces `FIGMA_IMPORT_PROGRESS.md` for Figma imports and `PROGRESS.md` for everything else. Same shape, different `## Source` block and typical phase set. Read the skill before creating or updating any progress file.

**Per-phase plan-review gate (HARD RULE).** When starting a phase, first flip it to `[~]` and expand it in the progress file with the implementation detail (what you'll do, files/blocks/tokens touched, mapping decisions) — then **pause for the user to review that phase's plan before building.** The user may request changes; apply them to the plan before implementing, not after. Build only on explicit go-ahead. The post-build summary pause still applies. So each phase has two stops: plan-review (before) and summary (after).

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

**Silent-failure traps** (cause "Block validation failed" or wrong markup the agent won't catch):

- Call `xfive-blocks-block-schema` before every block markup, every type, every time. Chisel ACF blocks use `chisel/{name}`, NOT `acf/{name}`. Failed call = block missing / build not run / wrong slug — stop, don't guess. Wrong attrs are silently ignored.
- **Static vs dynamic seed shape (check `renderMode` in schema response).** Static (`renderMode: "static"`, client `save()` returns JSX) MUST be paired tags with rendered inner HTML: `<!-- wp:name {attrs} -->INNER<!-- /wp:name -->`. Dynamic (`renderMode: "dynamic"`, server `render_callback`, including ACF blocks) may self-close: `<!-- wp:name {attrs} /-->`. Self-closing a static block stores empty inner HTML; the editor re-runs `save()`, sees a diff, shows "Block validation failed".
- **`seedAs` is necessary but not sufficient.** When attributes change `save()` output (image `width`/`height`/`sizeSlug`, button URL/className, heading `level`, group `tagName`/`layout`, etc.) → always paired tags with fully-rendered inner HTML, even if schema says self-closing is OK. When in doubt, use paired tags.
- **`useBlockProps.save({className})` auto-prefixes `wp-block-{namespace}-{name}`** onto the wrapper element. Read the block's `save.js` and include the auto-prefix in hand-written seed markup.
- **No whitespace inside containers wrapping `<InnerBlocks.Content />`.** Pretty-printing seeded HTML triggers "Block validation failed". Inline the inner-block comments tightly: `<div class="b-foo__content"><!-- wp:paragraph --><p>...</p><!-- /wp:paragraph --></div>`.
- **`core/group` with both `backgroundColor` AND `textColor` requires `has-background` class** on the rendered `<div>` — in addition to `has-background-color has-{slug}-background-color has-text-color`. Missing it = "Block validation failed" on save. Full saved class list: `has-background-color has-{slug}-background-color has-text-color has-background`. (When only one of the two is set, omit `has-background`.)
- **When unsure: round-trip.** Insert one instance in the editor manually, save, `xfive-posts-post-get-content`, copy that exact markup. The block's own `save()` is ground truth.
- Pages created with `post_status: "publish"` (not draft) for immediate preview. New ACF block → pause, user builds, schema-check passes, then seed (schema fails until `npm run build-scripts` runs).

### Spacing between blocks

- **Default `core/spacer` between every two sibling inner blocks** (even when Figma uses a uniform `gap`) — editors need draggable handles. NEVER `blockGap` or CSS `gap` in pattern SCSS for sibling spacing (invisible to editors). One-off margins or section padding in pattern SCSS → OK.
- **Walk the markup BEFORE serializing.** For every adjacent sibling pair, if Figma shows space, insert a spacer. Do this while writing, not after.

Spacer style selection (derive from theme.json, not hardcoded), spacer + flex double-spacing trap, `disableBottomMargin` / `u-no-margin-bottom` rules, and Chisel auto-margin sync → see [ai/rules/reference/design-tokens.md "Spacing between blocks"](ai/rules/reference/design-tokens.md#spacing-between-blocks).

### Reusing existing components (slider, icons, base styles)

- **Diff-only overrides.** Read the global partial first (`src/styles/components/_slider.scss`, `_buttons.scss`, etc.) before styling shared selectors. Only write rules that DIFFER from the default; never re-declare base props.
- **Match the global selector chain to win specificity.** Globals like `.swiper-navigation-wrapper .swiper-button` (0,2,0) tie with `.b-foo .swiper-button` (0,2,0) → source order wins, global beats your override. Mirror the chain + prepend your block scope. Otherwise `!important` with a one-line `// beats _slider.scss:NN` comment.
- **`!important` is OK** against vendor inline/runtime styles (Swiper, Gravity Forms) when specificity can't outrank them. Prefer specificity first.
- **Swiper, icons, SVG cleanup, asset enqueueing** → [ai/rules/reference/assets-and-scripts.md](ai/rules/reference/assets-and-scripts.md). Init Swiper via `data-*` attrs only (never hand-init); add new icons as variants in `assets/icons-source/` + `$static-icons`, never raw `mask: url(...)`.

### SCSS

- Always `@use '~design' as *;` at top of every SCSS file. Use design helpers (`get-color`, `get-gap`, `get-font-size`, `get-layout`, etc.) — never raw `var(--wp--*)` custom properties or raw hex/px for tokenized values.
- **Asset URLs in SCSS go through the `background-image()` mixin** (in `src/design/tools/_media.scss`) — never write raw `url('../../assets/...')` in pattern/block SCSS. Webpack resolves `url()` from the bundle entry, not from the partial, so relative paths from a partial silently break the build. Drop the SVG/PNG flat into `assets/images/` and call `@include background-image('name')` (default extension is `.svg`). **From inside an ACF/custom block** (`src/blocks*/{name}/style.scss`), pass `$is-block: true` — block SCSS lives one level deeper than pattern SCSS, and the mixin uses that flag to add the extra `../`.
- **Any repeated dimension (width, max-width, padding step, color, shadow, radius, transition) belongs in `theme.json` + a `get-*` helper**, not as a hardcoded `px-rem(…)` or hex literal in SCSS. If no helper exists for the category (e.g. you need a frame max-width and there's no `get-layout` yet), add the token under `settings.custom.{category}` in `theme.json` AND add the matching accessor in `src/design/tools/_theme.scss`. Hardcoded values are only acceptable for one-off, non-repeating, non-tokenizable values (e.g. `transform: translateY(-1px)` for a micro nudge). When in doubt, tokenize.
- BEM prefixes: `.c-` components, `.o-` objects, `.u-` utilities, `.b-` blocks, `.p-` patterns, `is-`/`has-` state.
- `_index.scss` barrels under `src/styles/{folder}/` are **auto-generated by the build** — never hand-edit; drop new partials and the build picks them up.

Full helper signatures, ITCSS layer order, JS/Twig conventions → [ai/rules/reference/coding-conventions.md](ai/rules/reference/coding-conventions.md).

### Block type preference: ACF default, native React = stop and ask (HARD RULE)

When the decision ladder reaches a custom block (i.e., pattern/block-style/block-mod can't cover the need), **prefer ACF blocks over native React blocks** — by default. ACF blocks are server-rendered Twig, fit Chisel's stack 1:1 (Timber already loaded), avoid the native-block silent-failure traps (`save()` validation, static-vs-dynamic seed shape, `useBlockProps.save` auto-prefix, whitespace-in-InnerBlocks), and give editors a clean field form.

**Native React block requires you to STOP and ask the user first.** Don't silently choose native. When you propose native, state the specific justification:

- Editor needs interactive UI that ACF fields can't express (true in-canvas state, drag-to-reorder, complex composability with `<InnerBlocks>` where editors place arbitrary nested blocks).
- Editor needs a live preview that depends on JS evaluation (rare).
- Performance-critical block where server rendering on every page view is wrong (very rare).

Slider that's only interactive on the frontend? ACF block + Swiper frontend. Tabs that just toggle visibility? Still ACF — frontend JS handles toggling. Reserve native React for cases where the **editor canvas** itself needs the interactivity.

Full decision ladder + exceptions: [ai/rules/reference/section-mapping-decisions.md "Block decision ladder"](ai/rules/reference/section-mapping-decisions.md#block-decision-ladder).

### Content vs CSS (HARD RULE)

**Don't hide content with CSS to paper over unwanted markup.** If the editor or seeded content has something the design doesn't want (starter widgets, default posts, demo blocks, placeholder fields), **remove it at the source** — via MCP (`xfive-widgets-widget-remove`, `xfive-posts-post-trash`, `xfive-acf-acf-field-update` clearing the value), via Appearance > Widgets, or by asking the user how to handle it.

`display: none` / `visibility: hidden` are reserved for:

- A11y-only visibility (`u-sr-only` text for screen readers).
- State-driven UI that the user genuinely toggles (mobile nav, accordion content, tab panels).
- Hiding empty containers via Twig logic (e.g. `{% if column|trim is not empty %}`) — not the same as CSS-hiding non-empty unwanted content.

Anything else (hiding the default "Hello World" post, hiding a starter widget you don't want, hiding a Gutenberg block you accidentally created) — fix the data, not the presentation. When in doubt, ask.

### Design tokens (theme.json)

- Build incrementally — add tokens as Figma screens introduce them. Don't bulk-import upfront.
- NEVER rename protected slugs (palette + spacing aliases) — SCSS references them by name. See [ai/rules/reference/design-tokens.md](ai/rules/reference/design-tokens.md) for the protected set.
- Extend existing aliases (`margin.medium`, `gap.normal`) rather than renaming — they're used inside theme.json itself.
- Map Figma spacing to the nearest existing step first; only extend if Figma has a denser scale.

### Before completing any task

- **Ask the user to run `npm run build-scripts`** to verify SCSS compiles — don't invoke it yourself.
- Every `get-color('...')` reference must exist in theme.json palette.
- Every `has-*-color` / `has-*-font-size` class in patterns must exist in theme.json presets.
- Pattern root wrapper has `p-{slug}` class; SCSS file matches.

## Reference, skills, templates

All guidance lives under [`ai/`](ai/). Old content is archived in `agent/` (reference only, may be stale).

### Reference (the "what" — load BEFORE the matching skill)

Reference docs own descriptive facts (file structures, decision ladders, token inventory, load-bearing constraints). Each reference doc routes you to the matching skill at the bottom. **For scaffolding tasks, always open reference first** — pattern-matching off sibling files in the repo misses required keys, build-pipeline rules, and other invariants.

- [file-locations](ai/rules/reference/file-locations.md) — where things go
- [design-tokens](ai/rules/reference/design-tokens.md) — token inventory + protected slugs → routes to `setup-theme-json` / `theme-json`
- [blocks](ai/rules/reference/blocks.md) — block/pattern file structures, build-pipeline rule, existing styles/mods, ACF field-data shape → routes to `create-acf-block` / `create-block` / `create-pattern` / `extend-core-block`
- [section-mapping-decisions](ai/rules/reference/section-mapping-decisions.md) — decision ladder + quick-pick table → routes to the right scaffolding skill
- [screen-build-order](ai/rules/reference/screen-build-order.md) — phase order + verification checklist
- [figma-import-template](ai/rules/reference/figma-import-template.md) / [progress-template](ai/rules/reference/progress-template.md) — thin pointers; format + procedure live in the [chisel-plan skill](ai/skills/chisel-plan/SKILL.md)
- [mcp-workflow](ai/rules/reference/mcp-workflow.md) — MCP tool usage (content, ACF, options, theme mods)
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

Flow: **CLAUDE.md (auto-loaded)** → **reference** → **skill** → **templates**.

- **CLAUDE.md**: always loaded by Claude Code at session start. Carries identity, architecture, project rules, and the routing index.
- **Reference**: load BEFORE the matching skill for any scaffolding or token task. Reference owns the "what"; skills assume you've read it. Load other reference docs on demand when you need them.
- **Skills**: load when running that skill — after its reference partner is in context.
- **Templates**: load only when copying code from them.
