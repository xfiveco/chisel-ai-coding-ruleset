# Chisel Project Rules

**Load this file at the start of every Chisel task.** These are load-bearing rules — breaking them causes silent failures, duplicate work, or production-visible bugs. Each rule links to the reference doc that explains it in full.

## Architecture

- Core in `core/`, never edit. Customize in `custom/app/` (same structure, namespace `Chisel\WP\Custom\`). Autoloader checks custom first. A pre-commit git hook blocks `core/` modifications — if a commit fails on a `core/` file, you're in the wrong layer; move the change to `custom/app/`.
- Gutenberg-first for posts/pages/CPTs. ACF metaboxes only for WooCommerce products.
- No hardcoded editable content in Twig templates. Use Customizer (logo), nav menus, or ACF Options.
- **No hooks in `custom/functions.php`** — it's a bootstrap list only. Put `add_filter`/`add_action` logic in a `custom/app/WP/{Feature}.php` class using the `HooksSingleton` trait, then `get_instance()` it in `functions.php`.

## Scaffolding (HARD RULE)

**Before creating any new block, pattern, CPT, ACF options page, design token, or reusable component, OPEN the matching reference doc in `rules/reference/`** — even if a sibling example exists. Reference owns the "what" (file lists, decision ladders, required keys, load-bearing constraints) and links out to the matching skill for the "how". Pattern-matching off sibling files silently misses constraints like "every SCSS file needs a JS entry" or "static blocks must be seeded as paired tags".

Entry points:

- Block / pattern / ACF block / block style — [reference/blocks.md](reference/blocks.md) (file structures, build-pipeline rule) → routes to the relevant skill.
- Choosing block vs pattern vs ACF vs CPT — [reference/section-mapping-decisions.md](reference/section-mapping-decisions.md) (decision ladder) → routes to the relevant skill.
- Design tokens / theme.json — [reference/design-tokens.md](reference/design-tokens.md) (token inventory, protected slugs) → routes to [skills/setup-theme-json.md](skills/setup-theme-json.md) or [skills/theme-json.md](skills/theme-json.md).
- Twig component — [reference/twig-templating.md](reference/twig-templating.md) → routes to [skills/create-component.md](skills/create-component.md).
- Skills without a reference owner (`adapt-base-styles`, `adapt-header-footer`, `create-acf-options`) — open the skill directly.

Full skill directory is in [CLAUDE.md](../CLAUDE.md).

## Content insertion (HARD RULE)

- **Requires the custom `xfive-mcp` WordPress plugin** (in `wp-content/plugins/xfive-mcp/`) to be active AND its MCP server registered in the agent client. If the `xfive-mcp-chisel` tools aren't exposed in your tool list, stop and ask the user to install/activate the plugin and register the MCP server — do not fall back to PHP seeds, WP-CLI, or manual paste.
- Any Gutenberg content insertion goes through `xfive-mcp-chisel` MCP tools. No PHP seeds, no WP-CLI, no manual paste. See [reference/mcp-workflow.md](reference/mcp-workflow.md).
- **ALWAYS call `xfive-blocks-block-schema` before any block markup** — every type, every time. Confirms registration (failed call = block missing/build not run/wrong slug — stop, don't guess) and exact attribute shape (wrong attrs silently ignored). Chisel ACF blocks use `chisel/{name}`, NOT `acf/{name}`.
- **Static vs dynamic seed shape (check `renderMode` in schema response).** Static blocks (`renderMode: "static"` — client `save()` returns JSX, e.g. `core/paragraph`, custom WP blocks) MUST be seeded as paired tags with the rendered inner HTML between them: `<!-- wp:name {attrs} -->INNER<!-- /wp:name -->`. Dynamic blocks (`renderMode: "dynamic"` — server `render_callback`, including ACF blocks) may be self-closing: `<!-- wp:name {attrs} /-->`. Self-closing a static block stores empty inner HTML; the editor re-runs `save()` on load, sees a diff, and shows "Block validation failed".
- **`seedAs` is necessary but not sufficient.** Even when schema says "Self-closing OK", the block may still have a client `save()` whose HTML output is compared byte-for-byte to `post_content` when the editor loads. Self-closing is only truly safe when the block has no `save()` HTML OR when its attributes don't affect markup. Any block with attributes that change `save()` output (image `width`/`height`/`sizeSlug`, button URL/className, heading `level`, group `tagName`/`layout`, etc.) MUST be seeded as paired tags with the fully-rendered inner HTML — even if `seedAs` says self-closing is OK. When in doubt, use paired tags.
- **`useBlockProps.save({className})` auto-prefixes `wp-block-{namespace}-{name}`** onto the wrapper element. A block registered as `{namespace}/{name}` with `useBlockProps.save({className: 'b-thing'})` actually renders `<tag class="wp-block-{namespace}-{name} b-thing ...">`. Easy to forget when hand-writing seed markup — read the block's `save.js` and include the auto-prefix.
- **Whitespace inside containers that wrap `<InnerBlocks.Content />` must match `save()` exactly.** Pretty-printing seeded HTML — adding newlines/tabs inside `<div class="...content">` wrappers that should be empty placeholders — triggers "Block validation failed". Inline the inner-block comments tightly: `<div class="b-accordion__item-content"><!-- wp:paragraph --><p>...</p><!-- /wp:paragraph --></div>` with no whitespace between `<div>` and the comment.
- **Verify by round-trip when unsure.** If you're seeding an unfamiliar block and not confident about `save()` output, the safe path is: insert one instance in the editor manually, save, then read it back via `xfive-posts-post-get-content` — copy that exact markup into your pattern. The block's own `save()` is ground truth; reconstructing from `save.js` is error-prone.
- Pages created with `post_status: "publish"` (not draft) so the user can preview immediately.
- New ACF block → pause, user builds, then schema-check → seed. Schema fails until `npm run build-scripts` runs.

## Patterns

- Every pattern has a single root `core/group` (or `core/cover`) with class `p-{slug}`. Required for scoped SCSS.
- Pattern SCSS at `src/styles/components/_p-{slug}.scss`, scoped under `.p-{slug}`. The build auto-generates `_index.scss` — drop the file and run `npm run build-scripts`; never hand-edit `_index.scss`.
- Default to patterns over custom blocks. See [reference/blocks.md](reference/blocks.md) block decision ladder.
- One pattern per section, not per page. Never build a single pattern that contains the whole homepage — each section gets its own `p-{slug}` pattern, assembled into the page via `xfive-posts-post-update-content` (write the full Gutenberg markup of the page).
- **Before building a repeater-based block for entity-like content** (case studies, team, portfolio, services-as-entities, events, locations, testimonials worth single-viewing), run the CPT ladder in [reference/section-mapping-decisions.md](reference/section-mapping-decisions.md#cpt-decision). If all 3 apply → CPT + CPT-driven block with `latest`/`selected` variants (Timber query in `chisel_timber_acf_blocks_data_{slug}` filter), NOT an ACF repeater. See [skills/create-cpt.md](skills/create-cpt.md) "Pair with a custom block".

## Page title display (ACF `page_title_display`)

Set per page based on Figma, in the same MCP batch as other ACF fields:

- Hero contains its own H1 → `"hide"`
- Custom visual but H1 needed for SEO → `"hide-visually"`
- Design shows a page title heading → `"show"` (default)

## Spacing between blocks

- **Default to `core/spacer` between every two sibling inner blocks** (even when Figma uses a uniform `gap`) — gives editors draggable handles. CSS `gap` doesn't.
- Figma px → style: 8→`tiny`, 16→`small`, 24→`normal`, 32→`medium`, 48→`large`, 64→`xlarge`, 80→`big`, 96→`huge`.
- NEVER `blockGap` (inconsistent) or CSS `gap` in pattern SCSS for sibling spacing (invisible to editors).
- One-off margins between two elements → pattern SCSS OK. Section padding (top/bottom of wrapper) → pattern SCSS OK.

**Walk the markup BEFORE serializing:** for every adjacent sibling pair, if Figma shows space, insert a `core/spacer` with the matching style. Do this while writing, not after.

**Spacers + `layout: flex` = double spacing.** Flex groups add `gap: var(--wp--style--block-gap)` on top of spacers. If using spacers → `layout: constrained` (default). Use flex only when you need flex features AND rely on its gap instead of spacers. Never mix.

**Chisel auto-adds bottom margins via `.c-block--{name}`** (in [src/styles/blocks/\_core.scss](../src/styles/blocks/_core.scss)) — stacks against spacers, causes double-gap.

At project start, sync those defaults to Figma's `text/*/paragraph-spacing` and `heading/*/paragraph-spacing` (don't keep Chisel starter values — see [CLAUDE.md](../CLAUDE.md)):

- Text blocks → `margin: 0 0 get-margin('{alias}')`
- Media/container blocks → `margin: 0 auto get-margin('{alias}')`

In pattern markup, set `"disableBottomMargin":true` + `"className":"u-no-margin-bottom"` on:

- Root pattern wrapper
- Every block immediately followed by a spacer
- Every last child of any container

In practice: nearly every non-spacer block inside patterns.

## Base styles vs pattern SCSS

- **Documented token values, mixin defaults, and block-style lists in these rules reflect Chisel starter state.** The project may have already adapted them — always read the actual file (`theme.json`, the mixin SCSS, `src/scripts/editor/blocks-styles.js`) for current state before comparing to the spec. Only protected slug _names_ (palette + spacing aliases) are stable; values are project-specific.
- If a Figma value should be the global default (all buttons, all headings) → update the base mixin in `src/design/tools/` or theme.json.
- If section-specific → override in pattern SCSS under `.p-{slug}`.
- Map Figma component variants to existing Chisel slots (primary/secondary/tertiary). Do not create new variants (white, etc.) while leaving old slots unused.

## Images

- Upload images as part of each section's build, not deferred to a final step. Each section should be fully reviewable (pattern + SCSS + images wired in) before moving on.
- Use `xfive-images-image-upload` without asking. Capture returned IDs, wire into block markup same step.
- SVG `<img>` tags MUST have explicit `width` and `height` attributes — SVGs have no intrinsic size.

## Breadcrumbs

- Render via `{{ breadcrumbs() }}` Twig fn (backed by `Chisel\Helpers\YoastHelpers::breadcrumbs()`). Style in `src/styles/vendor/_breadcrumbs.scss` (`.c-breadcrumbs`). Returns `''` without Yoast — safe pre-install. Never hand-roll.

## Reusing existing components (slider, icons, base styles)

- **Diff-only overrides.** Before styling `.swiper-button`, `.c-breadcrumbs`, `.wp-block-button` etc., read the global partial first (`src/styles/components/_slider.scss`, `_buttons.scss`, etc.). Only write rules that DIFFER from the default. Never re-declare base props (`display`, `cursor`, `transition`, default sizes).
- **Match the global selector chain to win specificity.** Globals like `.swiper-navigation-wrapper .swiper-button` (0,2,0) tie with `.b-block .swiper-button` (0,2,0) → source order wins, global beats your override. Mirror the global's chain, then prepend your block scope: `.b-block .swiper-navigation-wrapper .swiper-button` (0,3,0). Otherwise use `!important` with a one-line `// beats _slider.scss:NN` comment.
- **Swiper:** use the framework via `data-*` attrs on `.swiper.js-slider` (`data-slides-per-view`, `data-space-between`, `data-arrows`, `data-dots`, `data-breakpoints`, `data-args`). Don't hand-init Swiper. Reset positioning/colors via overrides; hide default text arrows with `&::after { content: none }` then render custom via `&::before` + `icon-svg()`. `slidesPerView: "auto"` requires fixed slide widths; otherwise use numeric values + `data-breakpoints`. **Pagination bullets need `flex-shrink: 0`** when the pagination container is a flex row, otherwise they collapse to 0 width and disappear.
- **Icons:** if a Figma icon differs from existing one, add a new variant — don't redefine. Place clean SVG (no Figma `var(--fill-0,…)`, no `width="100%"`) in `assets/icons-source/{name}.svg`, add slug to `$static-icons` in `src/design/settings/_index.scss`, then `@include icon-svg('{name}')`.
- **`!important` is OK** when a vendor library (Swiper, Gravity Forms) sets inline/runtime styles you can't outrank with specificity. Prefer specificity first; reach for `!important` only when proven necessary, with a one-line `// Swiper resets X` comment.

## SCSS

- Always `@use '~design' as *;` at top of every SCSS file. Use helpers (`get-color`, `get-gap`, `get-font-size`, etc.) — never raw `var(--wp--*)` custom properties.
- BEM naming: `.c-` components, `.o-` objects, `.u-` utilities, `.b-` blocks, `.p-` patterns, `is-/has-` state.
- Drop new partials into the relevant folder under `src/styles/`. The `_index.scss` barrels in every `src/styles/{folder}/` are **auto-generated by the build** (`chisel-scripts`) — never hand-edit them; the build picks up new partials automatically.

**Anti-patterns (NEVER do these — use the Chisel helper instead):**

- ❌ `mask: url('.../icons-source/{name}.svg')` → ✅ `@include icon-svg('{name}')`
- ❌ `var(--wp--preset--color--primary)` → ✅ `get-color('primary')`
- ❌ `var(--wp--preset--font-size--large)` → ✅ `get-font-size('large')`
- ❌ `var(--wp--preset--spacing--50)` → ✅ `get-margin('small')` / `get-gap('small')` / `get-padding('small')`
- ❌ raw hex values for brand colors → ✅ `get-color('{slug}')`
- ❌ raw px for spacing → ✅ `px-rem(n)` for one-offs, `get-margin('{alias}')` for tokens

## Design tokens (theme.json)

- Build incrementally — add tokens as Figma screens introduce them. Do not bulk-import upfront.
- NEVER rename protected slugs: `primary`, `primary-300`, `foreground`, `background`, `white`, `black`, `grey-100/300/900`, `secondary`, `info`, `error`, `success`, `warning`. SCSS references them by name.
- Extend existing custom aliases (`margin.medium`, `gap.normal`) rather than renaming — they're used inside theme.json itself.
- Map Figma spacing to nearest existing step first; only extend if Figma has a denser scale.

## Design import (Figma mode)

**Skip this section if the task isn't a Figma import** (static-asset or prompt mode — see [CLAUDE.md](../CLAUDE.md) "Modes").

- **For any Figma import task, load [skills/figma-to-chisel.md](skills/figma-to-chisel.md) FIRST.** It orchestrates the rest (theme setup → header/footer → sections → verification) and enforces Phase 0.5: create/update `FIGMA_IMPORT_PROGRESS.md` at project root from [reference/figma-import-template.md](reference/figma-import-template.md). Skip the orchestrator and you'll miss the tracking file, phase order, and skill handoffs.
- `FIGMA_IMPORT_PROGRESS.md` is the source of truth across sessions. Update it after every section built — patterns added, blocks added, CPTs, phase checkboxes, session log with absolute date. Survives `/compact` and new sessions; your memory does not.
- Load `figma:figma-use` skill before any `mcp__plugin_figma_figma__*` call. Pass `skillNames: "figma-use"`.
- URL: `figma.com/design/:fileKey/:fileName?node-id=:nodeId` → convert `-` to `:` in nodeId. Never call `get_metadata` on `/make/` files.
- Sections: top-to-bottom, one at a time, end-to-end (CPT + block + SCSS + images + page wiring + FIGMA_IMPORT_PROGRESS.md). No cross-section batching. Stop for review after each. Don't batch `get_design_context`.
- Extract every design property the Figma context provides — padding, gap, margin, widths, image dimensions, radius, typography — and map each to a theme.json token.

## ACF

- Populate ACF fields via `xfive-acf-acf-field-update` after creating options pages or field groups. Don't leave them empty for the user to fill manually.
- For options pages: `post_id: "option"`. For posts: numeric ID.

## Theme mods & options

- Use `xfive-options-options-update` to set theme mods (`custom_logo`, nav locations) and WP options (`show_on_front`, `page_on_front`).
- Always set the homepage after creating the Home page (`show_on_front: page`, `page_on_front: {id}`).

## Before completing any task

- Run `npm run build-scripts` to verify SCSS compiles.
- Every `get-color('...')` reference must exist in theme.json palette.
- Every `has-*-color` / `has-*-font-size` class in patterns must exist in theme.json presets.
- Pattern root wrapper has `p-{slug}` class; SCSS file matches.
