# Chisel Project Rules

**Load this file at the start of every Chisel task.** These are load-bearing rules — breaking them causes silent failures, duplicate work, or production-visible bugs. Each rule links to the reference doc that explains it in full.

## Architecture

- Core in `core/`, never edit. Customize in `custom/app/` (same structure, namespace `Chisel\WP\Custom\`). Autoloader checks custom first. A pre-commit git hook blocks `core/` modifications — if a commit fails on a `core/` file, you're in the wrong layer; move the change to `custom/app/`.
- Gutenberg-first for posts/pages/CPTs. ACF metaboxes only for WooCommerce products.
- No hardcoded editable content in Twig templates. Use Customizer (logo), nav menus, or ACF Options.
- **No hooks in `custom/functions.php`** — it's a bootstrap list only. Put `add_filter`/`add_action` logic in a `custom/app/WP/{Feature}.php` class using the `HooksSingleton` trait, then `get_instance()` it in `functions.php`.

## Scaffolding (HARD RULE)

**Before creating any new block, pattern, CPT, ACF options page, design token, or reusable component, open the matching reference doc in `rules/reference/`** — even if a sibling example exists. Reference owns the "what" (file lists, decision ladders, required keys, load-bearing constraints) and routes to the matching skill for the "how". Pattern-matching off siblings silently misses constraints.

Entry points:

- Block / pattern / ACF block / block style → [reference/blocks.md](reference/blocks.md)
- Block vs pattern vs ACF vs CPT decision → [reference/section-mapping-decisions.md](reference/section-mapping-decisions.md)
- Design tokens / theme.json → [reference/design-tokens.md](reference/design-tokens.md)
- Twig component → [reference/twig-templating.md](reference/twig-templating.md)
- Skills without a reference owner (`adapt-base-styles`, `adapt-header-footer`, `create-acf-options`) — open the skill directly.

Full skill directory in [CLAUDE.md](../CLAUDE.md).

## MCP (xfive-mcp-chisel) — required for all WP state writes

For any content insert/edit, image upload, ACF field, theme mod, option, nav menu, or post creation — use the `xfive-mcp-chisel` MCP tools. Never PHP seeds, WP-CLI, manual paste, or direct DB edits. If the tools aren't in your tool list, **stop** and ask the user to install the plugin + register the MCP server — don't improvise a fallback. Full tool list, payload shapes, and defaults → [reference/mcp-workflow.md](reference/mcp-workflow.md).

Page title display (ACF `page_title_display`): `hide` (hero contains its own H1), `hide-visually` (custom visual but H1 needed for SEO), `show` (default — design shows a page title heading).

**Silent-failure traps** (cause "Block validation failed" or wrong markup the agent won't catch):

- Call `xfive-blocks-block-schema` before every block markup, every type, every time. Chisel ACF blocks use `chisel/{name}`, NOT `acf/{name}`. Failed call = block missing / build not run / wrong slug — stop, don't guess. Wrong attrs are silently ignored.
- **Static vs dynamic seed shape (check `renderMode` in schema response).** Static (`renderMode: "static"`, client `save()` returns JSX) MUST be paired tags with rendered inner HTML: `<!-- wp:name {attrs} -->INNER<!-- /wp:name -->`. Dynamic (`renderMode: "dynamic"`, server `render_callback`, including ACF blocks) may self-close: `<!-- wp:name {attrs} /-->`. Self-closing a static block stores empty inner HTML; the editor re-runs `save()`, sees a diff, shows "Block validation failed".
- **`seedAs` is necessary but not sufficient.** When attributes change `save()` output (image `width`/`height`/`sizeSlug`, button URL/className, heading `level`, group `tagName`/`layout`, etc.) → always paired tags with fully-rendered inner HTML, even if schema says self-closing is OK. When in doubt, use paired tags.
- **`useBlockProps.save({className})` auto-prefixes `wp-block-{namespace}-{name}`** onto the wrapper element. Read the block's `save.js` and include the auto-prefix in hand-written seed markup.
- **No whitespace inside containers wrapping `<InnerBlocks.Content />`.** Pretty-printing seeded HTML triggers "Block validation failed". Inline the inner-block comments tightly: `<div class="b-foo__content"><!-- wp:paragraph --><p>...</p><!-- /wp:paragraph --></div>`.
- **When unsure: round-trip.** Insert one instance in the editor manually, save, `xfive-posts-post-get-content`, copy that exact markup. The block's own `save()` is ground truth.
- Pages created with `post_status: "publish"` (not draft) for immediate preview. New ACF block → pause, user builds, schema-check passes, then seed (schema fails until `npm run build-scripts` runs).

## Patterns

- Every pattern has a single root `core/group` (or `core/cover`) with class `p-{slug}`. Required for scoped SCSS.
- Pattern SCSS at `src/styles/components/_p-{slug}.scss`, scoped under `.p-{slug}`. `_index.scss` is auto-generated — never hand-edit.
- Default to patterns over custom blocks. One pattern per section, not per page — assemble sections into a page via `xfive-posts-post-update-content`.
- For entity-like content (case studies, team, services, events, locations, testimonials worth single-viewing) → run the CPT ladder in [reference/section-mapping-decisions.md](reference/section-mapping-decisions.md#cpt-decision) BEFORE building a repeater block.

## Spacing between blocks

- **Default `core/spacer` between every two sibling inner blocks** (even when Figma uses a uniform `gap`) — editors need draggable handles. NEVER `blockGap` or CSS `gap` in pattern SCSS for sibling spacing (invisible to editors). One-off margins or section padding in pattern SCSS → OK.
- **Walk the markup BEFORE serializing.** For every adjacent sibling pair, if Figma shows space, insert a spacer. Do this while writing, not after.

Spacer style selection (derive from theme.json, not hardcoded), spacer + flex double-spacing trap, `disableBottomMargin` / `u-no-margin-bottom` rules, and Chisel auto-margin sync → see [reference/design-tokens.md "Spacing between blocks"](reference/design-tokens.md#spacing-between-blocks).

## Base styles vs pattern SCSS

- **Documented token values, mixin defaults, and block-style lists in these rules reflect Chisel starter state.** Always read the actual file (`theme.json`, the mixin SCSS, `src/scripts/editor/blocks-styles.js`) for current state before comparing to the spec. Only protected slug _names_ are stable; values are project-specific.
- Global default (all buttons, all headings) → update base mixin in `src/design/tools/` or theme.json. Section-specific → override in pattern SCSS under `.p-{slug}`.
- Map Figma variants to existing Chisel slots (primary/secondary/tertiary). Don't create new variants (white, etc.) while leaving old slots unused.

## Reusing existing components (slider, icons, base styles)

- **Diff-only overrides.** Read the global partial first (`src/styles/components/_slider.scss`, `_buttons.scss`, etc.) before styling shared selectors. Only write rules that DIFFER from the default; never re-declare base props.
- **Match the global selector chain to win specificity.** Globals like `.swiper-navigation-wrapper .swiper-button` (0,2,0) tie with `.b-foo .swiper-button` (0,2,0) → source order wins, global beats your override. Mirror the chain + prepend your block scope. Otherwise `!important` with a one-line `// beats _slider.scss:NN` comment.
- **`!important` is OK** against vendor inline/runtime styles (Swiper, Gravity Forms) when specificity can't outrank them. Prefer specificity first.
- **Swiper, icons, SVG cleanup, asset enqueueing** → [reference/assets-and-scripts.md](reference/assets-and-scripts.md). Init Swiper via `data-*` attrs only (never hand-init); add new icons as variants in `assets/icons-source/` + `$static-icons`, never raw `mask: url(...)`.

## SCSS

- Always `@use '~design' as *;` at top of every SCSS file. Use design helpers (`get-color`, `get-gap`, `get-font-size`, etc.) — never raw `var(--wp--*)` custom properties or raw hex/px for tokenized values.
- BEM prefixes: `.c-` components, `.o-` objects, `.u-` utilities, `.b-` blocks, `.p-` patterns, `is-`/`has-` state.
- `_index.scss` barrels under `src/styles/{folder}/` are **auto-generated by the build** — never hand-edit; drop new partials and the build picks them up.

Full helper signatures, ITCSS layer order, JS/Twig conventions → [reference/coding-conventions.md](reference/coding-conventions.md).

## Design tokens (theme.json)

- Build incrementally — add tokens as Figma screens introduce them. Don't bulk-import upfront.
- NEVER rename protected slugs (palette + spacing aliases) — SCSS references them by name. See [reference/design-tokens.md](reference/design-tokens.md) for the protected set.
- Extend existing aliases (`margin.medium`, `gap.normal`) rather than renaming — they're used inside theme.json itself.
- Map Figma spacing to the nearest existing step first; only extend if Figma has a denser scale.

## Design import (Figma mode)

**Skip this section if the task isn't a Figma import** (static-asset or prompt mode — see [CLAUDE.md](../CLAUDE.md) "Modes").

- **Load [skills/figma-to-chisel.md](skills/figma-to-chisel.md) FIRST.** It orchestrates the rest (theme setup → header/footer → sections → verification) and enforces Phase 0.5: create/update `FIGMA_IMPORT_PROGRESS.md` at project root from [reference/figma-import-template.md](reference/figma-import-template.md).
- `FIGMA_IMPORT_PROGRESS.md` is the source of truth across sessions — update after every section (patterns, blocks, CPTs, phase checkboxes, session log with absolute date). Survives `/compact` and new sessions; your memory does not.
- Load `figma:figma-use` skill before any `mcp__plugin_figma_figma__*` call.
- Sections: top-to-bottom, one at a time, end-to-end (CPT + block + SCSS + images + page wiring + progress file). No cross-section batching. Stop for review after each. Don't batch `get_design_context`.

## Before completing any task

- Run `npm run build-scripts` to verify SCSS compiles.
- Every `get-color('...')` reference must exist in theme.json palette.
- Every `has-*-color` / `has-*-font-size` class in patterns must exist in theme.json presets.
- Pattern root wrapper has `p-{slug}` class; SCSS file matches.
