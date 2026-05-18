---
description: Bootstrap theme.json from a design spec (Figma variables, static mockups, or user prompt). Translates the design system into theme.json tokens so later work uses `var(--wp--preset--*)` instead of raw values. Run once per project.
---

# theme.json Bootstrap

**Load RULES.md first.** Run this BEFORE building any screens, patterns, or blocks.

## Spec source

The spec can come from any of three sources — procedure is the same, only step 2 (extraction) differs:

- **Figma mode** — user provides a Figma file URL with optional design system node ID. Use the Figma MCP tools to pull variables.
- **Static-asset mode** — user provides screenshots, PDFs, or a written token list. Extract values by reading.
- **Prompt mode** — user describes the system in chat ("primary is teal #0ABAB5, body is Inter 16/24, spacing scale 8/16/24/32..."). Use what they give; ask before inventing values.

In all modes: read theme.json first, never fabricate values, and always update SCSS references when you rename slugs.

## Prerequisites

1. Confirm theme.json still has the Chisel example palette (`#dd2424` primary, `#22dbdb` secondary). If already customized, ask before overwriting.
2. **Figma mode only**: load `figma:figma-use` skill before any `mcp__plugin_figma_figma__*` call. URL parsing: `figma.com/design/:fileKey/:fileName?node-id=:nodeId` → convert `-` to `:` in nodeId. `/branch/:branchKey/` → use `branchKey` as fileKey. `/make/` → use `makeFileKey`, never call `get_metadata`.

## Procedure

### 1. Read current theme.json

Read `theme.json` at the theme root. Note which tokens are still example data (safe to overwrite) vs custom (ask first).

### 2. Pull tokens from the spec

**Figma mode** — decide where tokens come from BEFORE inspecting sections:

1. **Design system node available** → `get_variable_defs(fileKey, dsNodeId)` on it first. Preferred path — returns structured variables.
2. **No centralized DS node** → per-section `get_design_context`, extract CSS variables from output. Use a representative frame.
3. **DS + section overrides** → do both. Apply overrides as section-scoped SCSS, not new theme.json tokens.
4. **No Variables, only Styles** → `get_metadata` + `get_screenshot` and infer visually — flag to the user that values are being inferred.

Never skip — jumping straight to section markup without knowing the token source leads to hardcoded values. Extract every property the Figma context provides: padding, gap, margin, widths, image dimensions, radius, typography (size/weight/line-height), colors. Map each to the nearest theme.json token; if no match, add the token first. Never hardcode raw px or hex when a token can represent them.

**Static-asset mode:** read the screenshots/PDF, ask user to confirm hex values, font names, and spacing steps that aren't visually unambiguous.

**Prompt mode:** use what the user provided. If anything is missing (e.g. user gives colors but no spacing scale), ask — don't invent.

### 3. Map spec → Chisel slugs

Chisel uses a fixed taxonomy. Reuse existing slug names whenever possible so SCSS keeps working. See [reference/design-tokens.md](../reference/design-tokens.md) for the full slug → meaning map.

**Rule:** if the spec has richer structure (e.g., `surface/muted`, `border/subtle`, `brand/tertiary`), add as new slugs alongside existing ones — don't force into existing slots.

**Do NOT keep Chisel starter values if they diverge from the spec.** The Chisel defaults are a starting point, not the final product ([CLAUDE.md](../../CLAUDE.md)). This applies to spacing especially — "close enough" is not acceptable. If the spec uses 16/24/32/48/64/96 and Chisel's default scale is 4/8/12/16/20/24/28/32/48, **replace the scale with the spec's values** (keeping the same slug numbers so SCSS keeps resolving). Same for colors and fonts: values must match the spec exactly.

**Search/replace for slug renames.** If you need to rename a slug (color, font-size, spacing), always follow with a project-wide SCSS search/replace for `get-color('old-slug')`, `get-font-size('old-slug')`, `get-margin('old-slug')`, etc. Never leave stale references.

### 4. Protected slugs (DO NOT RENAME)

Full list in [reference/design-tokens.md](../reference/design-tokens.md) (palette, named aliases). These are referenced by name in SCSS/JS — renaming silently breaks the build. Before renaming any slug, grep for references:

```
Grep: get-color\('{slug}'\)  in  src/
Grep: has-{slug}(-color|-background-color)?  in  .
Grep: "{slug}"  in  src/scripts/editor/
```

If there are matches, either keep the slug or update every match in the same change.

### 5. Apply changes (in order)

Edit `theme.json` in-place. Preserve keys/ordering; only change values and add new slugs.

1. `settings.color.palette` — hex values + new slugs
2. `settings.color.gradients` — from spec
3. `settings.color.duotone` — same
4. `settings.typography.fontFamilies` — every weight/style used
5. `settings.typography.fontSizes` — sizes + fluid ranges
6. `settings.spacing.spacingSizes` — scale. **Pull every unique spacing value from the spec** (`xsmall`, `small`, `base`, `medium`, `large`, `xlarge`, `section/padding-*`, `spacing/spacing-*`, `xxsmall`, etc.) and map to the 9 slug slots (20/30/40/50/60/70/80/90/100). Also update `settings.custom.margin`/`padding`/`gap` aliases if alias names change. Remember: WP `core/spacer` at `.is-style-X` maps 1:1 to these aliases — the visible spacer gap in the editor is what `get-margin('X')` evaluates to, so this step directly affects what spacer options editors can use.
7. `settings.custom.*` — radius, shadow, transition, letter-spacing, line-height
8. `settings.layout` — contentSize / wideSize
9. `styles.elements.h1..h6` — if heading scale shifted

### 6. Fonts

Place WOFF2 files at `assets/fonts/{filename}.woff2`. Download from `https://gwfh.mranftl.com/fonts` if Google Font — no need to ask. If missing, flag as follow-up and DO NOT fabricate files.

### 7. Fix SCSS fallout

After theme.json changes, verify:

- `Grep: get-color\('([^']+)'\)` in `src/` — every argument exists as a slug
- `Grep: has-([a-z0-9-]+)-color` in `patterns/` and `src/styles/blocks/` — each matches a palette slug

If mismatch: either add the missing slug or update the SCSS.

### 8. Hand off

Once theme.json is stable and SCSS builds:

- **Figma mode** → invoke [figma-to-chisel](figma-to-chisel.md) for each page.
- **Static-asset / prompt mode** → proceed to [adapt-base-styles](adapt-base-styles.md), then [create-pattern](create-pattern.md) / [create-acf-block](create-acf-block.md) / [create-cpt](create-cpt.md) as needed.

## Output report

- **Colors added/changed**: slug → old hex → new hex (or "new")
- **Fonts**: body + headings families, weights, missing WOFF2 files
- **Type scale**: slug → size changes
- **Spacing scale**: new steps
- **Layout widths**: content / wide
- **SCSS references touched**: files where `get-color()` args were updated (ideally zero)
- **Follow-ups**: missing font files, unmappable tokens

## Guidelines

1. Never rename protected slugs without updating every reference in the same change.
2. Never fabricate values — if the spec has no token, keep the Chisel default and flag it.
3. Prefer extending (new slugs) over renaming (existing slugs).
4. Always read theme.json first — don't rewrite from a template.
5. Fluid min/max ratio ≤ 2× — don't shrink below ~60% of max on mobile.
