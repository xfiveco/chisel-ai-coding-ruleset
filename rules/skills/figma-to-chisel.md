---
description: End-to-end Figma -> Chisel import. Orchestrates theme setup, base style adaptation, pattern creation, image upload, and page assembly. Use when user provides a Figma URL to import.
---

# Figma -> Chisel Import (Orchestrator)

**Load RULES.md first.** This skill calls other skills — don't reinvent their work.

## Required prerequisites

1. Load `figma:figma-use` skill before any `mcp__plugin_figma_figma__*` call.
2. Read [reference/screen-build-order.md](../reference/screen-build-order.md) for the phase checklist.

## Skill map

| Need                           | Skill                                         |
| ------------------------------ | --------------------------------------------- |
| First-time token bootstrap     | [setup-theme-json](setup-theme-json.md)       |
| Update theme.json              | [theme-json](theme-json.md)                   |
| Adapt buttons/typography/links | [adapt-base-styles](adapt-base-styles.md)     |
| Header/footer                  | [adapt-header-footer](adapt-header-footer.md) |
| Create a section pattern       | [create-pattern](create-pattern.md)           |
| Field-driven block             | [create-acf-block](create-acf-block.md)       |
| Interactive custom block       | [create-block](create-block.md)               |
| Core block variant/toggle      | [extend-core-block](extend-core-block.md)     |
| CPT or taxonomy                | [create-cpt](create-cpt.md)                   |
| ACF options page               | [create-acf-options](create-acf-options.md)   |
| Twig component                 | [create-component](create-component.md)       |

## Procedure

### Phase 0 — Project setup (first time)

If `theme.json` still has example palette (`#dd2424` primary, `#22dbdb` secondary), run `setup-theme-json` first.

### Phase 0.5 — FIGMA_IMPORT_PROGRESS.md

Read `FIGMA_IMPORT_PROGRESS.md` in project root. If missing, create from [reference/figma-import-template.md](../reference/figma-import-template.md).

### Phase 1 — Scope

Ask user (unless answered):

1. Figma URL + node (parse fileKey + nodeId)
2. What is this screen? (static page / CPT / archive / global)
3. Editor-driven or hardcoded content?
4. Interactive behavior? (accordion, tabs, slider → forces custom block)

### Phase 2 — Inspect

1. `get_metadata(fileKey, nodeId)` — cheap skeleton, extract section node IDs
2. `get_screenshot(fileKey, nodeId)` — visual reference (full page)
3. `get_variable_defs(fileKey, dsNodeId)` — confirm tokens match theme.json

Do NOT call `get_design_context` on all sections at once — overflow risk.

### Phase 3-4 — Section loop (one at a time, end-to-end)

For each section from top to bottom:

1. `get_design_context(fileKey, sectionNodeId)` — extract every property
2. Decide mapping via [reference/section-mapping-decisions.md](../reference/section-mapping-decisions.md) ladder
3. If first section or new elements appear → adapt base styles via `adapt-base-styles`
4. Build pattern via `create-pattern` (or appropriate skill)
5. Upload section images via `xfive-images-image-upload`
6. Push section to page via `xfive-posts-post-update-content` (full page markup; for partial updates fetch with `post-get-content`/`block-tree`, modify the markup string, write the whole thing back)
7. Update FIGMA_IMPORT_PROGRESS.md with what was built
8. Verify in browser before moving to next section

Header/footer: use `adapt-header-footer` skill, not patterns.

### Phase 5 — Verification

Run checklist in [reference/screen-build-order.md](../reference/screen-build-order.md) > "Phase 5".

### Phase 6 — Iteration

When refining: fresh screenshot → compare rendered → adjust tokens first, then pattern SCSS, then block code.

### Phase 7 — Update FIGMA_IMPORT_PROGRESS.md

Flip phase checkboxes, add patterns/blocks/CPTs, append session log line with absolute date.

## Output report

- Screen built: Figma node → WP page ID
- New artifacts: CPTs, blocks, patterns, theme.json changes
- Compile status: build-scripts pass/fail
- Follow-ups: missing assets, unclear copy, deferred interactivity
