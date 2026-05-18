---
description: Adapt Chisel's base styles (buttons, typography, links, forms, spacing) to match a target spec. Run BEFORE creating patterns so they inherit correct defaults. Spec source can be Figma, static assets, or user prompt.
---

# Adapt Base Styles

**Load RULES.md first.** Do this before writing pattern SCSS — patterns should inherit correct base styles, not override them.

Spec source: Figma `get_design_context`, mockup/screenshot, or a written description from the user. Procedure is identical — only step 1 (extraction) differs.

## Decision: global vs pattern-scoped

1. **Should look this way everywhere?** → Update the base mixin/element SCSS. Patterns inherit automatically.
2. **One-off variation for a specific section?** → Scope in pattern SCSS under `.p-{slug}`.
3. **Reusable variant but not default?** → Add a block style via [extend-core-block](extend-core-block.md).

**Default to global.** Don't duplicate pattern-scoped overrides for every section when the design wants different defaults.

## Procedure

1. Extract design properties from the spec — `get_design_context` output (Figma mode), or read the mockup / user description (other modes).
2. For each property, compare against the current base (file map below).
3. If it should be the global default → update the base file.
4. Map spec component variants to **existing Chisel slots** (primary/secondary/tertiary). Don't create new variant names (white, etc.) while leaving old ones unused — remap instead.

## File map

### Buttons

| What                                                    | File                                                 |
| ------------------------------------------------------- | ---------------------------------------------------- |
| Base mixin (padding, font, border, radius, transitions) | `src/design/tools/_buttons.scss` → `@mixin button()` |
| Variant mixins (primary, secondary, outline)            | Same file                                            |
| Size variants                                           | Same file → `button-small()`, `button-large()`       |
| Block styles registration                               | `src/scripts/editor/blocks-styles.js`                |
| Block button SCSS                                       | `src/styles/blocks/_core-button.scss`                |
| Component button SCSS                                   | `src/styles/components/_buttons.scss`                |

**Before building any pattern that uses a button, open `src/design/tools/_buttons.scss` and compare every property of `@mixin button()` to the spec's button:** padding (horizontal/vertical), font-family, font-size, font-weight, line-height, border-width, border-radius, transition. Read the mixin's current values — they may already have been adapted from Chisel starter values — and update only what diverges from the spec. Same procedure for `button-small`/`button-large` size variants and `button-primary`/`button-secondary`/`button-tertiary` color/border specifics.

### Per-block defaults (check BEFORE building a pattern that uses the block)

| Block                                                                                                   | File                                      | What lives here                                                       |
| ------------------------------------------------------------------------------------------------------- | ----------------------------------------- | --------------------------------------------------------------------- |
| Global block margins (all blocks)                                                                       | `src/styles/blocks/_core.scss`            | `.c-block--{name}` bottom margins for text and media/container blocks |
| `core/group`                                                                                            | `src/styles/blocks/_core-group.scss`      | Group wrapper styles                                                  |
| `core/button`                                                                                           | `src/styles/blocks/_core-button.scss`     | Button block defaults                                                 |
| `core/spacer`                                                                                           | `src/styles/blocks/_core-spacer.scss`     | Spacer `is-style-*` padding values                                    |
| `core/media-text`                                                                                       | `src/styles/blocks/_core-media-text.scss` | Media+text layout                                                     |
| `core/gallery`                                                                                          | `src/styles/blocks/_core-gallery.scss`    | Gallery grid defaults                                                 |
| `core/details`                                                                                          | `src/styles/blocks/_core-details.scss`    | Expand/collapse details                                               |
| `core/post-title` / `core/post-date` / `core/query` / `core/search` / `core/comments` / `core/latest-*` | `src/styles/blocks/_core-{name}.scss`     | Query and post-meta defaults                                          |

### Elements (HTML primitives under `styles/elements/`)

| What                            | File                               |
| ------------------------------- | ---------------------------------- |
| Horizontal rule (`<hr>`)        | `src/styles/elements/_hr.scss`     |
| Base `<a>`                      | `src/styles/elements/_link.scss`   |
| Form inputs, selects, textareas | `src/styles/elements/_form.scss`   |
| Shared element resets           | `src/styles/elements/_shared.scss` |

### Typography

| What                                | File                                 |
| ----------------------------------- | ------------------------------------ |
| Families, sizes, fluid ranges       | `theme.json` → `settings.typography` |
| Heading h1-h6 styles                | `theme.json` → `styles.elements`     |
| Body defaults                       | `theme.json` → `styles.typography`   |
| Custom line-heights, letter-spacing | `theme.json` → `settings.custom`     |

### Links

| What                           | File                                  |
| ------------------------------ | ------------------------------------- |
| Link mixin (decoration, hover) | `src/design/tools/_link.scss`         |
| Base `<a>`                     | `src/styles/elements/_link.scss`      |
| Link colors                    | `theme.json` → `styles.elements.link` |

### Forms

| What                  | File                             |
| --------------------- | -------------------------------- |
| Input/textarea/select | `src/styles/elements/_form.scss` |
| Gravity Forms         | `src/styles/gravity-forms.scss`  |

### Spacing

| What               | File                                                |
| ------------------ | --------------------------------------------------- |
| Scale tokens       | `theme.json` → `settings.spacing.spacingSizes`      |
| Named aliases      | `theme.json` → `settings.custom.margin/padding/gap` |
| Content/wide width | `theme.json` → `settings.layout`                    |

### Icons (for button icons, link icons, etc.)

Chisel has a built-in icon system. Icons ship as SVG source files and are compiled into a spritesheet + class names.

| What                                          | File / action                                                                                                 |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Source SVGs                                   | `assets/icons-source/{name}.svg` — drop new ones here                                                         |
| Static icons list (used by `@mixin icon-svg`) | `src/design/settings/_index.scss` → `$static-icons: ('minus', 'plus', 'arrow-right', 'arrow-left', 'search')` |
| Editor-visible icon names                     | `core/WP/Assets.php` → look for the `'arrow-right' => __('Arrow Right', ...)` list                            |
| Button usage (right)                          | `"className":"has-icon has-icon-{name}","buttonIcon":"{name}"`                                                |
| Button usage (left)                           | add `has-icon-left` to className                                                                              |
| Button usage (with vertical separator)        | add `has-icon-separator` to className                                                                         |
| Direct SCSS use                               | `@include icon-svg('{name}')` inside `::before`/`::after`                                                     |

**When the spec uses an icon not in the list:**

1. Export the icon as a single-color SVG with `fill="currentColor"` (or no fill — gets masked).
2. Save as `assets/icons-source/{kebab-case-name}.svg`.
3. Add the name to `$static-icons` tuple in `src/design/settings/_index.scss`.
4. Add the name → label mapping in `core/WP/Assets.php` (editor UI).
5. Use via `has-icon has-icon-{name}` + `buttonIcon` attribute.

Don't inline SVG in Twig/patterns when a registered icon would do — single source of truth.

### Colors, radius, shadows

| What                           | File                                    |
| ------------------------------ | --------------------------------------- |
| Palette                        | `theme.json` → `settings.color.palette` |
| Radius / shadows / transitions | `theme.json` → `settings.custom.*`      |
| SCSS accessors                 | `src/design/tools/_theme.scss`          |

**Border radius check** — read the current `custom.border-radius` block in `theme.json` (the project may have already adapted from Chisel starter values). Specs often use `0` (sharp), `4px`, or very large pill values. Update the scale to match the spec. If the design has zero radius globally, set `border-radius: 0` in base mixins (button, input, card) — don't rely on aliases that still evaluate to non-zero.

### Global components

| What                | File                                                                   |
| ------------------- | ---------------------------------------------------------------------- |
| Header, footer, nav | `src/styles/components/_header.scss`, `_footer.scss`, `_main-nav.scss` |
| Nav toggle          | `src/styles/components/main-nav-toggle/`                               |

## Verification

After changes, run `npm run build-scripts` — every `get-color('slug')` and `has-{slug}-color` must resolve.
