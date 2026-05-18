---
description: Create a block pattern — a reusable section layout composed of existing blocks. Default choice for section layouts. Use before reaching for custom blocks or ACF blocks.
---

# Create Block Pattern

**Load [reference/blocks.md](../reference/blocks.md) first** — pattern file structure, root-wrapper rule (`p-{slug}` class), existing block styles, pattern categories. This skill is the _how_; reference is the _what_. Then load RULES.md if you haven't.

Patterns are the **default** for section layouts — cheaper than blocks, more editable. See [section-mapping-decisions.md](../reference/section-mapping-decisions.md) if unsure whether a pattern is the right choice.

## When to use

Hero sections, CTAs, feature grids, pricing tables, testimonial rows, FAQ lists, logo strips. Any layout combining headings, paragraphs, images, buttons, columns, groups, covers, spacers.

## Procedure

1. **List every core block the section will use** (group, heading, paragraph, image, button, columns, spacer, cover, etc.).
2. **Audit base SCSS for each block BEFORE writing markup.** For every block in the list, open the corresponding `src/styles/blocks/_core-{name}.scss` (if it exists) and `src/styles/blocks/_core.scss` globals. Also open `src/styles/elements/*.scss` for any HTML primitives (`<hr>`, `<a>`, `<input>`, etc.). Check each property (margin, padding, color, font, gap, border) against Figma values for this block:
   - **Matches Figma** → leave it alone.
   - **Diverges and should be the GLOBAL default** → update the base file now (before writing pattern markup). See [adapt-base-styles](adapt-base-styles.md).
   - **Diverges but only for this section** → note it; handle in pattern SCSS (step 5).

   Skipping this audit causes silent conflicts: base margins stack with spacers, base button styles override Figma variant, base group padding fights pattern layout, etc.

3. **Create pattern PHP** at `patterns/{slug}.php` — see [templates/pattern-markup.md](../templates/pattern-markup.md) for block grammar examples.
4. **Root wrapper rule**: single `core/group` (or `core/cover`) with class `p-{slug}`. Required for scoped SCSS.
5. **Create pattern SCSS** at `src/styles/components/_p-{slug}.scss`, scoped under `.p-{slug}`. Only include properties that are section-specific; global divergences should already be fixed in step 2.
6. **SCSS index is auto-generated** — do NOT edit `src/styles/components/_index.scss` by hand. The build picks up new `_p-{slug}.scss` files automatically.
7. **Upload images** via `xfive-images-image-upload` (capture IDs).
8. **Push to page** via MCP (see [reference/mcp-workflow.md](../reference/mcp-workflow.md)):
   - `xfive-blocks-block-schema` on every block type used
   - `xfive-posts-post-update-content` with the full serialized Gutenberg markup of the page (read current via `post-get-content`/`block-tree` first if you need to edit only one section).
9. **Verify** with `xfive-posts-post-get-content` or `block-tree`, then visually in browser.

## Pattern PHP template

```php
<?php
/**
 * Title: {Pattern Title}
 * Slug: chisel/{slug}
 * Categories: chisel-patterns/{category}
 * Description: {Description}
 * Keywords: {keyword1}, {keyword2}
 *
 * @package Chisel
 */

?>

<!-- block markup here — see templates/pattern-markup.md -->
```

## Pattern SCSS template

```scss
@use '~design' as *;

.p-{slug} {
  // scoped styles only
}
```

Breakpoints: `@include bp('large') { ... }`, `@include bp-down('medium') { ... }`.

## Spacing between children

See [RULES.md "Spacing between blocks"](../RULES.md#spacing-between-blocks) — load-bearing rules (spacer over `blockGap`, the px→style mapping, flex+spacer double-gap trap, `disableBottomMargin` placement) all live there. Available `is-style-*` spacer sizes come from `registerSpacerStyles()` in `src/scripts/editor/blocks-styles.js`.

## Guidelines

1. Prefix pattern slug with nothing (`home-hero`), but class is `p-home-hero`.
2. Use theme.json preset classes: `has-primary-color`, `has-large-font-size`, `is-style-primary`.
3. Use realistic copy, not lorem ipsum.
4. Flag unknowns — don't fabricate content.
5. If a pattern needs interactivity → stop and use [create-block](create-block.md) instead.
