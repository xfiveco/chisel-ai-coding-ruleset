# Design Tokens (theme.json)

Chisel starter inventory and naming conventions. **Treat values below as the starter state — read `theme.json` for the project's current values before relying on any specific size/hex.** The protected slug names (palette + spacing aliases) are the only stable facts; everything else may have been adapted to the project's spec.

See [skills/theme-json.md](../skills/theme-json.md) to modify.

## Colors

Palette slugs (protected — never rename; change hex values only):

- `foreground` (default text), `background` (default page bg)
- `primary` (+100, 200, 300, 600, 800, 900) — brand primary / CTA
- `secondary` (+100, 200, 300, 600, 800, 900) — brand secondary
- `grey-100/200/300/400/600/800/900` — neutral scale
- `black`, `white` — literal
- `success`, `info`, `warning`, `error` — status

Usage: `var(--wp--preset--color--{slug})` or SCSS `get-color('slug')`.

## Typography

- **body**: Roboto 300/700 — `var(--wp--preset--font-family--body)`
- **headings**: Manrope 700/800 — `var(--wp--preset--font-family--headings)`

Font sizes (9-step, fluid) — slugs: `tiny`, `small`, `normal`, `medium`, `large`, `extra-large`, `very-large`, `big`, `huge`. Read `theme.json` for current rem values + fluid ranges.

Heading defaults live in `theme.json` `styles.elements.h1..h6` — read the file for current size/line-height bindings.

Line height slugs: `small`, `medium`, `normal`, `large`. Read `theme.json` `settings.custom.line-height` for current numeric values.

## Spacing

Numeric scale lives in `settings.spacing.spacingSizes` (slugs are numeric strings — `20`, `30`, …). Named aliases live in `settings.custom.{margin|padding|gap}.{alias}` and point at the numeric slugs.

Named alias slugs (protected — never rename): `tiny`, `little`, `small`, `normal`, `medium`, `large`, `xlarge`, `big`, `huge`. The project may extend the scale further (`gigantic`, `section`, etc.) — read `theme.json` for the current set of slugs and their rem values.

SCSS: `get-margin('medium')`, `get-padding('large')`, `get-gap('normal')`.

## Border radius

Slugs: `tiny`, `little`, `small`, `normal`, `medium`, `large`, `full`. Read `theme.json` `settings.custom.border-radius` for current values — the project may have flattened the scale to `0` for a sharp-corner design.

SCSS: `get-border-radius('small')`.

## Box shadows

Numbered presets (read `theme.json` `settings.custom.box-shadow` for the current list — typically `1`, `2`, `3`, `3r`, `4`). SCSS: `get-box-shadow('3')`.

## Transitions

Slugs: `slow`, `normal`, `fast`. Read `theme.json` `settings.custom.transition` for current durations. SCSS: `get-transition('normal')` — default is `normal`.

## Letter spacing

Slugs: `none`, `tight`, `loose`, `looser`. Read `theme.json` `settings.custom.letter-spacing` for current values.

## Layout

- Content width: `settings.layout.contentSize` (read `theme.json`)
- Wide width: `settings.layout.wideSize` (read `theme.json`)

## Gradients

Slugs: `primary-secondary`, `secondary-primary` (135deg linear).

## Duotones

Slug: `primary-secondary`.

## Token extraction (procedure)

For the per-mode procedure (Figma variable defs vs static asset / prompt parsing) and the apply-changes step order, see [skills/setup-theme-json.md](../skills/setup-theme-json.md). Reference only owns the _what_ (the inventory above and the constraints below).

## Spacing between blocks

Default to a `core/spacer` between every two sibling inner blocks (even when Figma uses a uniform `gap`) — gives editors draggable handles. `blockGap` and CSS `gap` don't, and `blockGap` is inconsistent across layouts.

### Picking the spacer style

For each spacer, derive the slug from `theme.json` instead of hardcoding:

1. Read `settings.custom.gap` aliases from `theme.json` (each maps to a numeric `spacingSizes` slug, ultimately a rem value).
2. Convert each alias's rem value to px (×16).
3. Pick the alias whose px is closest to the Figma spacing. Use `"className": "is-style-{alias}"` on the spacer.

If no alias is close enough, extend the scale in `theme.json` rather than picking a misfit.

### Margin sync at project start

Chisel auto-adds bottom margins to text/media blocks via `.c-block--{name}` (in `src/styles/blocks/_core.scss`). These stack against spacers and cause double-gap unless synced:

1. Read Figma `text/*/paragraph-spacing` and `heading/*/paragraph-spacing` variables.
2. Compare to current mixin defaults in `src/design/tools/`.
3. Update each mixin's bottom margin to match Figma:
   - Text blocks → `margin: 0 0 get-margin('{alias}')`
   - Media/container blocks → `margin: 0 auto get-margin('{alias}')`

### Suppressing auto-margins in patterns

In pattern markup, set `"disableBottomMargin": true` + `"className": "u-no-margin-bottom"` on:

- Root pattern wrapper
- Every block immediately followed by a spacer
- Every last child of any container

In practice: nearly every non-spacer block inside patterns.

### Layout trap

**Spacers + `layout: flex` = double spacing.** Flex groups add `gap: var(--wp--style--block-gap)` on top of spacers. If using spacers → `layout: constrained` (default). Use `flex` only when you need flex features AND rely on its gap instead of spacers. Never mix.

## Special cases

- **Container widths** → map to `contentSize`/`wideSize`, don't hardcode column pixel values
- **SVG `<img>` tags** → always set explicit `width` and `height` (SVGs have no intrinsic size, render as 100x100 otherwise)

## Fonts

Google Fonts: download WOFF2 from `https://gwfh.mranftl.com/fonts`, save to `assets/fonts/`, register `fontFace` in theme.json. Never fabricate font files — flag as follow-up if missing.
