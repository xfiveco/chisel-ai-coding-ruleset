# Section Mapping Decisions

For each distinct section/component in the spec (Figma node, mockup region, or described feature), apply this decision ladder. Stop at the first match — default to the simplest option that works. For file structures, existing styles/mods, and the build-pipeline rule, see [blocks.md](blocks.md).

## Quick pick

| Need                                | Approach               | Skill                                               | Example                         |
| ----------------------------------- | ---------------------- | --------------------------------------------------- | ------------------------------- |
| Simple text/image section           | Core blocks directly   | —                                                   | No custom code                  |
| Visual variant of a core block      | Block style            | [extend-core-block](../skills/extend-core-block.md) | Button colors, spacer sizes     |
| Extra toggle/setting on a block     | Block mod              | [extend-core-block](../skills/extend-core-block.md) | Disable bottom margin           |
| Layout section with standard blocks | Pattern                | [create-pattern](../skills/create-pattern.md)       | Hero, CTA, features grid        |
| Repeatable field-driven content     | ACF block              | [create-acf-block](../skills/create-acf-block.md)   | Slider, team grid, testimonials |
| Complex interactive component       | Custom WP block        | [create-block](../skills/create-block.md)           | Accordion, tabs, carousel       |
| Entity-like repeating content       | CPT + CPT-driven block | [create-cpt](../skills/create-cpt.md)               | Case studies, team, portfolio   |

## Block decision ladder

1. **Pure core blocks** — section is just headings + paragraphs + image + buttons → use core blocks directly inside a pattern. No custom code.
2. **Core block + block style** — section is a core block (button, spacer, quote) with a color/size variant → add `registerBlockStyle()` entry in `src/scripts/editor/blocks-styles.js` and style in `src/styles/blocks/_core-{name}.scss`. Use [extend-core-block](../skills/extend-core-block.md).
3. **Core block + mod (toggle)** — on/off behavior (hide margin, reverse columns, invert colors) → block mod filter in `src/scripts/editor/mods/`. Use [extend-core-block](../skills/extend-core-block.md).
4. **Pattern** — layout composed of several blocks (hero, features grid, CTA band, testimonials, logo strip, FAQ list, pricing table) → use [create-pattern](../skills/create-pattern.md). **Default for most section layouts.**
5. **ACF block** — field-driven repeating content (slider, team grid, testimonials carousel, logo grid with links, stats counters) → use [create-acf-block](../skills/create-acf-block.md).
6. **Custom WP block** — complex interactive UI (tabs, accordion, filterable portfolio, stepped form) → use [create-block](../skills/create-block.md). Reserve for truly unique, interactive components.

> **Editability note:** Patterns are fully editable in the WP editor — they're composed of native blocks. "Editable in admin" is not a reason to choose ACF block over pattern. Choose ACF block (rung 5) only when content is _repeating with a fixed shape_ — N items where each item has the same set of fields (testimonials, logo grid, team cards). Choose custom WP block (rung 6) only when the editor needs interactive React UI in the canvas (tabs switching state, accordion, InnerBlocks composition).

## Shared components rule

Before creating anything new, check existing patterns/blocks for reuse. **Figma mode**: check the "Shared components" section of `FIGMA_IMPORT_PROGRESS.md`. **Other modes**: skim `patterns/` and `src/blocks*/` for matching slugs. If the current section matches an existing pattern/block, reuse it — don't create `hero-2` when `hero` exists.

If the existing component needs a variation, prefer:

1. Adding a block style or modifier class
2. Adding a pattern variant flag (not a new pattern)
3. Creating a new pattern only if the structure genuinely differs

If you reuse, record the current screen under the component's "used by" list (Figma mode: in `FIGMA_IMPORT_PROGRESS.md` Phase 7).

## CPT decision

Create a CPT when ALL three apply:

1. Spec shows multiple instances of the same content shape (portfolio items, team, case studies, services, events, locations)
2. User wants editors to manage them individually
3. There is a single-view page or an archive/listing

**Do NOT** create a CPT for:

- One-off pages
- Homepage sections
- Content that only appears inside a pattern

When you do create one:

- Use [create-cpt](../skills/create-cpt.md)
- Default Gutenberg-enabled (`editor` in supports, `show_in_rest: true`)
- ACF metaboxes only for WooCommerce products — never regular CPTs
- Per-entry layout: block template (seed blocks) + custom/ACF blocks for unique sections

## Header / footer / global elements

- Site-wide header/footer are Twig templates, not patterns. Use [adapt-header-footer](../skills/adapt-header-footer.md).
- Do NOT build header/footer as a block pattern unless user explicitly wants block-based site editing.

## Forms

- Use Gravity Forms (already integrated — `src/styles/gravity-forms.scss`)
- Form designs → configure GF form → embed via `[gravityform]` shortcode or GF block → scope styles in `_p-{pattern-slug}.scss`
