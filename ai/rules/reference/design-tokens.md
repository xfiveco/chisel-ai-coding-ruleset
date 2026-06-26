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

Match every token by its **resolved value** (px / hex / weight), never by its name. Read the value from Figma (`get_variable_defs` — the only ground truth) and from the current `theme.json`, then pick the slug whose value matches. If none matches, map to the nearest step; extend `theme.json` only if Figma has a genuinely denser scale. **Never rename protected slugs** (palette + spacing aliases) to mirror Figma — SCSS, helpers, and theme.json reference them by name. Use a Figma name as a slug **only** for a brand-new token Chisel lacks.

**False friends — same name, different value (never map name-to-name):**

- `large` — Chisel's is mid-scale, not the biggest; Figma `large` / `xl` map _above_ it (`xlarge` / `big`).
- `medium` — Chisel's spacing value ≠ Figma's; check the px.

Don't trust the Figma token's **name**, the rendered `get_design_context` CSS, or the `weight:` / `size:` in a `Font(...)` summary — each can disagree with the bound variable. On conflict, the design's visual intent wins — surface it to the user. Always re-read `theme.json`; never assume a slug's value from memory or this doc.

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

This is for spacing _between_ siblings only. A section's own outer top/bottom band padding is NOT a spacer — set it as `style.spacing.padding` on the section's outer block (see [blocks.md "Root wrapper rule"](blocks.md#root-wrapper-rule)).

### Picking the spacer style

For each spacer, derive the slug from `theme.json` instead of hardcoding. This is the "match VALUES, never names" rule above, applied to spacers — the Figma gap's **resolved px** drives the choice, never the Figma token's name:

1. Get the Figma spacing's **resolved value** via `get_variable_defs` (e.g. `48`), not the token name (Figma `large` may be 48px while Chisel `large` is 32px — see the HARD RULE above).
2. Read the **rendered height of each spacer style** from `src/styles/blocks/_core-spacer.scss` — each `is-style-{alias}` sets `padding: get-margin('{alias}') 0`, so the spacer's rendered height = **2× that margin token** (e.g. `small` margin 12px → 24px tall; `medium` 24px → 48px; `large` 32px → 64px). The base `.wp-block-spacer` (no style class) is `padding: get-margin('normal') 0` → 48px.
3. Pick the `is-style-{alias}` whose **rendered height** matches the Figma px (closest if no exact). Seed `"className": "is-style-{alias}"`.

**The `height` attribute is IGNORED — always seed `height:"auto"`.** `src/scripts/editor/mods/core-spacer.js` runs an `editor.BlockEdit` filter that force-sets every `core/spacer` to `height: "auto"` on load. So an inline `height:"32px"` is silently overwritten and the rendered size comes **only** from the `is-style-*` padding. Seeding a px height is dead markup that misleads — write `{"height":"auto","className":"is-style-{alias}"}` with `style="height:auto"` (matches [templates/pattern-markup.md](../templates/pattern-markup.md)).

If no alias's rendered height is close enough, extend the scale in `theme.json` (add a margin alias + matching spacer `is-style` in `_core-spacer.scss` + `registerSpacerStyles()`) rather than picking a misfit — never rename an existing alias to match Figma, and never rely on the `height` attr.

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
