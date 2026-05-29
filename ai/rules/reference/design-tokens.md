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

## Mapping Figma tokens → theme.json (HARD RULE — match VALUES, never names)

**A Figma token name is NOT its Chisel slug. Match by resolved value, not by matching alias names.** Figma's spacing/size names are arbitrary and differ per file — e.g. Figma `large` = 48px but Chisel `get-gap('large')` = 32px (slug `70`/2rem); Figma `base` = 24px. Mapping Figma `large` → Chisel `large` by name silently ships the wrong value.

Procedure for **every** Figma-sourced value (spacing, gap, padding, font-size, radius, color, shadow, line-height):

1. Call `get_variable_defs` on the node to get the **resolved value** (`48`, `#24c1b1`, `700`, …). This is the only ground truth.
2. Read the current `theme.json` and find the slug whose **value matches** (48px → `gap`/`padding` alias that resolves to `3rem`, which is `xlarge` → spacing `80`). Use that slug's `get-*()` helper.
3. If **no** slug matches the value, map to the nearest existing step (per "Design tokens" rule in CLAUDE.md), and only extend `theme.json` if Figma genuinely has a denser scale. **Do NOT rename existing slugs to mirror Figma** — palette + spacing aliases are protected (SCSS, helpers, and theme.json reference them by name; renaming breaks call sites).

Do NOT trust: the Figma token's **name**, the rendered `get_design_context` CSS, or the `weight:`/`size:` fields inside a `Font(...)` style summary — each can disagree with the bound variable (e.g. a `nav item/font-weight` variable of `900` rendering as `weight: 700`). When a bound variable and the rendered value conflict, the design's visual intent wins — surface the conflict to the user rather than silently picking one.

The current alias→value map drifts as the project adapts — **always re-read `theme.json` to confirm**, never assume a slug's value from memory or from this doc.

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

- Content width: `settings.layout.contentSize` (read `theme.json`) — narrowest, used by `core/group` default and most text-heavy patterns. SCSS: `get-layout-size('content')`.
- Wide width: `settings.layout.wideSize` (read `theme.json`) — used by `alignwide` and broader patterns. SCSS: `get-layout-size('wide')`.
- Other named widths (frame, container, narrow…): `settings.custom.layout.{name}` in `theme.json`. SCSS: `get-layout('{name}')`. Example: `settings.custom.layout.frame-width: "67rem"` for the 1072px header/footer frame → `max-width: get-layout('frame-width')`.

### Width-token decision ladder

1. **Text content body** (narrowest) → `contentSize` via `get-layout-size('content')`.
2. **Wide patterns** (grids, image bands) → `wideSize` via `get-layout-size('wide')`. Apply via `alignwide` className.
3. **Anything else recurring** (header/footer frame, narrow CTA card, sidebar rail) → new `settings.custom.layout.{name}` slug + use `get-layout('{name}')`.
4. **Truly one-off, never repeated** → may stay as a raw `px-rem(…)` in pattern SCSS, but flag it: if a second pattern needs the same width, promote to a token.

**Never hardcode `max-width: px-rem(N)` for a value that recurs.** Two patterns with the same hardcoded width = the token should already exist.

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
