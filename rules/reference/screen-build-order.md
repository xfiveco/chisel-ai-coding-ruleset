# Screen Build Order

Per-screen pipeline — applies whether the spec comes from Figma, static assets, or a written prompt.

## Phase 4 — Build order (for one screen)

Each phase depends on previous. Execute in order:

1. **Confirm theme.json** matches the spec's design tokens (Phase 0).
2. **Adapt base styles** to match the spec via [adapt-base-styles](../skills/adapt-base-styles.md). Do this BEFORE creating patterns — patterns should inherit correct defaults.
3. **Register new CPTs / taxonomies** via [create-cpt](../skills/create-cpt.md).
4. **Create custom blocks** via [create-block](../skills/create-block.md) / [create-acf-block](../skills/create-acf-block.md). Compile after.
5. **Add block styles / mods** via [extend-core-block](../skills/extend-core-block.md). Compile after.
6. **Create patterns** via [create-pattern](../skills/create-pattern.md). Each pattern has root wrapper `p-{slug}` class + matching `_p-{slug}.scss`.
   - **Build sections in spec reading order — top to bottom.** The page assembles visually as you go, making review easier and matching the user's mental model. Do NOT order by complexity (simple-first), even though it feels safer — the cost of getting stuck on a hard section early is lower than the cost of an empty-looking page during review.
   - **Upload images for each section as part of the section build** (not deferred). Capture attachment IDs.
   - **Wire images into block markup** in the same step — no placeholder `src=""` left behind.
7. **Assemble section into page via MCP** — see [mcp-workflow.md](mcp-workflow.md). Each section fully reviewable before moving on.
8. **Review & adjust SCSS** — run build, open page, fix spacing/color drift against the spec (Figma screenshot, mockup, or written description).

## Phase 5 — Verification checklist

Before declaring a screen done:

- [ ] `npm run build-scripts` passes (no Sass errors)
- [ ] Every `get-color('...')` in new SCSS refers to a slug that exists in theme.json
- [ ] Every `has-*-color` / `has-*-background-color` class in patterns refers to an existing palette slug
- [ ] Every `has-*-font-size` refers to an existing fontSize slug
- [ ] Patterns have a root wrapper with `p-{slug}` class
- [ ] Pattern SCSS file `src/styles/components/_p-{slug}.scss` exists and is scoped under `.p-{slug}` (the build auto-regenerates `_index.scss` to forward it)
- [ ] Custom blocks compile and appear in "Chisel Blocks" inserter category
- [ ] CPTs show up in admin menu with correct icon
- [ ] Rendered page matches the spec at primary viewport
- [ ] Page is published (not draft)
- [ ] Homepage set via `options-update` (if this is the Home page)
