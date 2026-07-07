# Blocks Reference

Descriptive lookup for block types, file structures, and existing styles/mods. For the decision ladder (which type to pick), see [section-mapping-decisions.md](section-mapping-decisions.md). For step-by-step scaffolding, see the matching skill linked from each section below.

## File structures

**Build-pipeline rule (applies to ALL blocks).** Every `.scss` file must be `import`ed by a JS entry (`index.js`, `script.js`, `view.js`, `edit.js`, etc.) listed in `block.json` — otherwise webpack does not compile it and the block renders unstyled. The example reference is `assets/example-blocks/`. Each `block.json` script key produces a matching `style-{handle}.css` (e.g. `script` → `style-script.css`, `viewScript` → `style-view.css`, `editorScript` → `style-index.css`), which must be listed in `style` / `viewStyle` / `editorStyle`.

Set `"ignoreScripts": ["script"]` (or similar) only when that script is SCSS-only (no real frontend JS) — `ignoreScripts` suppresses script execution while still letting webpack build the CSS. If the script has real frontend JS, omit `ignoreScripts` so it loads.

### Block JS/CSS keys — what each file is for

Registration is type-agnostic: native (`src/blocks/`) and ACF (`src/blocks-acf/`) blocks loop the **same** key set (`core/Factories/RegisterBlocks.php`). Both use this model.

| `block.json` key   | File                                  | Context           | Purpose                                                                                                                                       |
| ------------------ | ------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `editorScript`     | `index.js`                            | Editor only       | Block registration + (native) the `edit.js` component. Imports editor SCSS.                                                                   |
| `editorStyle`      | `index.css`                           | Editor only       | Editor-only appearance.                                                                                                                       |
| `script`           | `script.js`                           | Editor + frontend | **CSS entry** (`import './style.scss';`). Carries real JS only for a **native** block that needs the same behavior live in the editor canvas. |
| `style`            | `style-script.css`, `style-index.css` | Editor + frontend | Shared CSS. **Array** when `style.scss` is imported by both `script.js` and `index.js` — webpack emits one file per entry, so list both.      |
| `viewScript`       | `view.js`                             | **Frontend only** | **The block's own frontend behavior lives here** — scoped to the block, never shipped to the editor.                                          |
| `viewStyle`        | `view.css`                            | Frontend only     | Frontend-only CSS (not loaded in editor).                                                                                                     |
| `viewScriptModule` | `view.js` (ESM)                       | Frontend only     | Same as `viewScript` but registered as an ES module — use only when the block genuinely needs native ESM.                                     |

**Rules (HARD):**

1. **A block's frontend JS lives in the block as `view.js` (`viewScript`)** — not in `src/scripts/modules/`, not pushed into the editor via `script`. One block = one folder for its markup, styles, AND scripts.
2. **`src/scripts/modules/` is the global/site-wide frontend layer** (nav, scroll, fades, utils — bootstrapped via `app.js`), most of it not block-related. Put code there only when it's genuinely site-global. Block behavior never goes here; a util shared by 2+ blocks is the rare exception, and even then it's a shared helper, not block code.
3. **`script.js` is the SCSS entry** — it imports `style.scss` so webpack builds `style-script.css` (which feeds the critical-CSS inliner in `core/WP/AcfBlocks.php`). It stays even when the block has frontend JS; the behavior goes in `view.js`, not here.
4. **`ignoreScripts` = "this key has no JS to run"** (build the CSS, don't enqueue an empty handle). Set it for `script` on any block whose frontend JS is in `view.js` or which has none. It does NOT make `script.js` permanently CSS-only — it's about whether _that key_ carries runnable JS.

**ACF vs native — the only divergence.** ACF blocks render ACF's **field form** in the editor, not the Twig output — so frontend behavior _always_ goes in `view.js`/`viewScript` (`script`-as-JS would load in an editor with nothing to bind to). Native blocks render the real block via `edit.js`, so they may use `script` (drop `ignoreScripts`) **only** when the identical behavior must run live in the editor canvas. Default for both: behavior → `view.js`.

### Custom WP Block (`src/blocks/{name}/`)

Reference layout: `assets/example-blocks/blocks/example/`. Full file list:

```
block.json        # metadata, API v3, category: chisel-blocks. Lists every JS entry (editorScript, script, viewScript) and the matching style files.
index.js          # editor registration — must `import './style.scss'` (or `editor.scss`) to compile editor CSS
edit.js           # React editor component
save.js           # React save component (omit if server-rendered via `render.php`)
script.js         # editor + frontend webpack entry. Must `import './style.scss'` so webpack builds `style-script.css`. Add to `ignoreScripts: ["script"]` if SCSS-only.
view.js           # frontend-only webpack entry. Must `import './view.scss'` if those styles exist.
style.scss        # shared editor + frontend styles — imported by both index.js AND script.js, which produces TWO outputs (style-index.css + style-script.css); list both in "style" array
view.scss         # frontend-only styles — imported by view.js
editor.scss       # editor-only — imported by edit.js / index.js
render.php        # optional — server-side render (use with `"render": "file:./render.php"` in block.json; replaces save.js)
init.php          # optional — server-side registration (e.g. for child blocks needing REST/MCP validation)
```

`script.js` is included even though it often just contains `import './style.scss';` — without it, webpack has no entry for the shared frontend styles. See `assets/example-blocks/blocks/example/script.js` for the canonical one-line example.

### ACF Block (`src/blocks-acf/{name}/`)

```
block.json              # metadata with "acf" key + renderCallback; set "script": "file:./script.js", "style": ["file:./style-script.css"]. Add "ignoreScripts": ["script"] ONLY when script.js is SCSS-only.
{name}.twig             # Twig template
style.scss              # styles — must be `import`ed by script.js so webpack builds style-script.css
script.js               # REQUIRED — webpack entry. Even if only `import './style.scss';`. Omit and SCSS never compiles → block renders unstyled.
view.js                 # frontend-only JS (viewScript) — the block's own interactivity lives HERE, not src/scripts/modules/. Add only when the block is interactive.
view.scss               # frontend-only CSS (viewStyle) — imported by view.js. Optional.
acf-json/*.json         # ACF field group
```

See [create-acf-block](../../skills/chisel-create-acf-block/SKILL.md) for the full procedure and required block.json keys — always load it before scaffolding a new ACF block. For custom WP blocks see [create-block](../../skills/chisel-create-block/SKILL.md).

**ACF field group naming (HARD RULE).** Keys must be hex hashes, filename = group key, field `name`s must be namespace-prefixed (block initials → `bp_heading`), `label`s stay human. Full spec, prefix-derivation cases, per-context prefix sources, and example: **[acf-naming.md](acf-naming.md)** — the canonical, all-context rule. Read it before authoring any field group JSON.

**ACF field data shape (load-bearing — applies any time you seed an ACF block).** Markup is `wp:chisel/{name}`, NOT `wp:acf/{name}` — Chisel uses `register_block_type()`. Every `data` field needs a `_{name}: "field_key"` partner. ACF resolves values via these key-pointers; without them `get_fields()` returns empty. Repeaters need `items: N`, `_items: "field_B"` plus every sub-field per row with its key (field names below use the prefix rule above):

```json
"data": {
  "bp_heading": "Hello", "_bp_heading": "field_A",
  "bp_steps": 2, "_bp_steps": "field_B",
  "bp_steps_0_quote": "first",  "_bp_steps_0_quote": "field_C",
  "bp_steps_1_quote": "second", "_bp_steps_1_quote": "field_C"
}
```

### Pattern (`patterns/{slug}.php`)

```php
<?php
/**
 * Title: Pattern Name
 * Slug: chisel/{slug}
 * Categories: chisel-patterns/{category}
 * Description: What this pattern does
 * Keywords: keyword1, keyword2
 *
 * @package Chisel
 */
?>
<!-- block markup with p-{slug} root wrapper -->
```

## Pattern categories

Built-in (registered by core): `hero`, `features`, `cta`, `testimonials`, `team`, `pricing`, `text`, `gallery`, `faq`, `stats`, `logos`. A pattern's `Categories:` header uses the namespaced form `chisel-patterns/{slug}`.

**A category must be registered before use — an unregistered slug is silently dropped and the pattern falls into "Uncategorized".** Prefer a built-in; only add a new one when none fit.

### Registering a custom category (project layer)

Never edit core's list in `core/WP/Blocks.php` — core exposes the **`chisel_block_patterns_categories`** filter. Add categories via `block_patterns_categories()` in `custom/app/WP/Blocks.php` (the `Chisel\WP\Custom\Blocks` class — create it with `HooksSingleton` and `get_instance()` it in `custom/functions.php` if absent; register the filter in `filter_hooks()`). Key by **unprefixed** slug; core prepends the `chisel-patterns/` namespace and `[Theme Name]` label.

```php
// custom/app/WP/Blocks.php — inside block_patterns_categories()
$custom_categories = array(
    'process' => array(
        'label'       => __( 'Process', 'chisel' ),
        'description' => __( 'Process / steps sections.', 'chisel' ),
    ),
);

return array_merge( $categories, $custom_categories );
```

Then a pattern can use `Categories: chisel-patterns/process`.

## Pattern slug naming (HARD RULE)

Name patterns by **function, not page** — the slug is the section type, not where it first appears: `hero` not `home-hero`, `pill-list` not `industry-pills`. Variant-qualify (`hero-split`, `cta-banner`), never page-qualify. Patterns are reusable across pages/CPTs; a page-named slug couples one to a single context. Reuse an existing pattern (or add a variant) before minting a new slug — see [section-mapping-decisions.md "Shared components rule"](section-mapping-decisions.md#shared-components-rule). Page-prefix only a pattern that is genuinely one-off and page-bound.

## Root wrapper rule

Every pattern has a single root `core/group` (or `core/cover`) with class `p-{slug}`. Required for:

1. **Scoped styling** — custom class isolates pattern's inner blocks from same blocks elsewhere
2. **Structural integrity** — single root keeps pattern as one selectable unit in editor

The root wrapper also carries `"metadata":{"name":"{Pattern Title}"}` (per pattern — its human Title, matching the `Title:` header) so it shows a readable label in the editor List View instead of a generic "Group".

**Four-way sync (HARD RULE).** The pattern slug drives four names that must always match: the `Slug:` header suffix (`chisel/{slug}`), the pattern filename (`patterns/{slug}.php`), the root class (`p-{slug}`), and the SCSS file + scope (`src/styles/patterns/_{slug}.scss` scoped under `.p-{slug}`). Derive the slug from the section's **function** — never from the page the section was built for. (`p-home-hero` on a `hero-image-cta` pattern is the canonical failure: the class stops matching the slug, and when a second page reuses the section, two pattern files style the same page-named class and their SCSS collides.) Renaming a pattern renames all four in the same change. Never let two pattern files share one `p-*` class base.

**Section vertical padding lives on the outer block, not in pattern SCSS.** Set the section's top/bottom band padding as `style.spacing.padding` (`var:preset|spacing|NN` preset) on the root `core/group` — both `core/group` and `core/columns` support `spacing.padding`, so a columns-rooted section can carry it directly. Reserve pattern SCSS for inner/structural spacing the block can't express.

Pattern SCSS: `src/styles/patterns/_{slug}.scss`, scoped under `.p-{slug}`. Filenames never carry the `p-` prefix — the `patterns/` folder already provides the context (same as `patterns/{slug}.php`); the prefix belongs only to the CSS class, per BEM.

**Class only the root; target inner blocks by tag (HARD RULE).** Only the root group carries `p-{slug}`. **Never** add a BEM `__element` class to leaf/text blocks (paragraph, heading, list, image) — style them by tag from the root: `.p-{slug} h2`, `.p-{slug} p`, or by the block's own class `.p-{slug} .wp-block-media-text`. A `p-{slug}__heading` class lives only on the seeded instance, so a paragraph an editor adds later inherits nothing. Add a `p-{slug}__{name}` class to a **structural** block (inner group, columns, media-text) **only when** tag/descendant targeting can't single it out (e.g. two sibling inner groups needing different styles) — never to text elements.

## Existing block styles

Registered in `src/scripts/editor/blocks-styles.js` — **read that file for the current list** (the project may have added/removed variants since these docs were written). Each top-level `register*Styles()` method holds one block's variants:

- `registerButtonsStyles()` → `core/button` variants (typically primary / secondary / tertiary, each with an `-outline` companion)
- `registerSpacerStyles()` → `core/spacer` size variants used by `is-style-{name}` (e.g. `tiny`, `small`, `medium`, `large`, `xlarge`, `big`)

## Existing block mods

In `src/scripts/editor/mods/` (registered via `blocks-mods.js`). Several add custom attributes or force values — **when seeding, set the attr AND its companion class together**, or the editor desyncs / the style doesn't render:

- **core.js**: adds a `disableBottomMargin` attr (toggle) to every `core/*` + `chisel/*` block. **The attr alone renders nothing** — the bottom-margin removal is done by the `u-no-margin-bottom` utility class. When seeding, set BOTH `"disableBottomMargin":true` AND `"className":"… u-no-margin-bottom"` (and include `u-no-margin-bottom` in the rendered class list). Needed on any block immediately followed by a spacer or the last child of a container (else base margin + spacer = double gap).
- **core-button.js**: adds `buttonSize`, `buttonIcon`, `buttonIconPosition` attrs to `core/button`, kept in sync with classes `is-size-{size}`, `has-icon has-icon-{name}`, `has-icon-left`. **When seeding a button icon/size, set the attr AND the class:** e.g. `{"buttonIcon":"arrow-right","className":"… has-icon has-icon-arrow-right"}`. Class-only works visually but the editor control shows empty and a later edit can wipe it. Icon `{name}` must be in `$static-icons`.
- **blocks-alignment.js**: on select, force-sets a default `align` per block from the PHP-provided `chiselEditorScripts.blocksDefaultAlignment` map. A seeded `align` on those blocks may be overwritten when the user selects the block — check the map (or just rely on it) rather than fighting it.
- **core-spacer.js**: forces every `core/spacer` to `height:"auto"` in the editor — spacer size comes ONLY from the `is-style-{size}` padding, never the `height` attr. Always seed `{"height":"auto","className":"is-style-{size}"}` — the style class is mandatory on every spacer, even for the default size (no bare spacers). See [design-tokens.md "Picking the spacer style"](design-tokens.md#picking-the-spacer-style).

## Spacing between sibling blocks

Composition rule (the spacer _sizing_ math — px→style mapping, margin-sync, flex double-gap trap — lives in [design-tokens.md "Spacing between blocks"](design-tokens.md#spacing-between-blocks)):

- **Default a `core/spacer` between every two sibling inner blocks** — even when Figma uses a uniform `gap`. Editors need draggable handles; `blockGap` and CSS `gap` give none and are invisible to the editor. NEVER use `blockGap` or CSS `gap` in pattern SCSS for **vertical** sibling spacing. One-off margins or section padding in pattern SCSS are fine.
- **Horizontal gutters are the exception**: gaps between columns in `core/columns` (or a grid) can't be spacers — set them ON the block via `blockGap` with a preset value (`"style":{"spacing":{"blockGap":{"left":"var:preset|spacing|{N}"}}}`). That's the correct, token-backed tool for the horizontal axis.
- **Walk the markup BEFORE serializing.** For every adjacent sibling pair, if the design shows space, insert a spacer — do this while writing, not after.
- Pair with the `disableBottomMargin` + `u-no-margin-bottom` rule above (every block immediately followed by a spacer, and every last child of a container) or base margin + spacer = double gap.

Editor-only UI helpers (not seed-affecting): `components/BlockEditSelector.js` (an "Edit {block}" button), `components/RenderAppender.js` (custom InnerBlocks inserter), `blocks.js` (adds `e-block-sidebar--{block}` class to the inspector), `utils.js` (icon choices for the button mod).
