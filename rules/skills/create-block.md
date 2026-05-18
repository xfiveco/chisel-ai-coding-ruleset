---
description: Create a custom Gutenberg block (React edit/save + frontend JS). Use ONLY for truly interactive or structurally unique components that can't be built from core blocks, block styles, block mods, or patterns.
---

# Create Custom Gutenberg Block

**Load [reference/blocks.md](../reference/blocks.md) first** — file list for `src/blocks/{name}/`, the build-pipeline rule (every SCSS file must be imported by a JS entry), and the decision ladder confirming you actually need a custom block. This skill is the _how_; reference is the _what_. Then load RULES.md if you haven't.

Custom blocks are the most expensive option — confirm via [section-mapping-decisions.md](../reference/section-mapping-decisions.md) that patterns, block styles, and block mods can't cover the need.

**Seed shape — static vs dynamic.** Custom WP blocks with `save()` returning JSX are _static_ — they must be seeded as paired tags with the rendered inner HTML between `<!-- wp:name -->...HTML...<!-- /wp:name -->`. Self-closing only for dynamic blocks (`save` returns `null`, server `render_callback`). Always check `renderMode` in `xfive-blocks-block-schema` response before seeding.

## Procedure

1. **Create all files** in `src/blocks/{block-name}/` — see [templates/custom-block-template.md](../templates/custom-block-template.md). File list: [reference/blocks.md "Custom WP Block"](../reference/blocks.md#custom-wp-block-srcblocksname).
2. **Run `npm run dev` or `npm run build-scripts`** to compile.
3. **Verify** in editor under "Chisel Blocks" category. `xfive-blocks-block-schema` to confirm registration + check `renderMode`.
4. **Test** in editor and on frontend.

## Optional files

- `view.js` — frontend interactivity (class-based vanilla ES6)
- `init.php` — server-side registration (required for child blocks to exist in registry for REST/MCP validation)

## Conventions

- CSS class: `b-{block-name}` + BEM (`__element`, `--modifier`)
- JS hook class: `js-{block-name}` (separate from CSS)
- State classes: `is-*`, `has-*`
- Translation: `__('text', 'chisel')`
- Block category: `chisel-blocks`
- Block name: `chisel/{block-name}`

## Guidelines

1. If it can be a pattern, make it a pattern.
2. If it can be a block style or mod, use [extend-core-block](extend-core-block.md) instead.
3. Child blocks (like accordion-item) need `"parent": ["chisel/parent-block"]` in block.json.
4. Parent-child communication via `"providesContext"` / `"usesContext"`.
5. Never use CSS classes as JS selectors — use `js-*` prefixed data attributes or classes.
