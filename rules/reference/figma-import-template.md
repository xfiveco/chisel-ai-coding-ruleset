# FIGMA_IMPORT_PROGRESS.md Template

Use this template when creating `FIGMA_IMPORT_PROGRESS.md` in the project root. This file is the source of truth for multi-screen imports — survives across sessions, `/compact`, and crashes.

User edits this file by hand. Always treat contents as authoritative over your own recollection.

Keep entries concise — read at the start of every session.

```markdown
# Figma Import Progress

<!-- Tracks state of Figma -> Chisel imports across sessions.
     Both Claude and user edit it. When in doubt, this file wins. -->

## Source

- File key: {fileKey}
- File name: {Figma file name}
- URL: {full URL}

## Theme setup (setup-theme-json)

- [ ] theme.json bootstrapped from Figma variables
- [ ] Colors palette mapped
- [ ] Typography mapped (fonts, sizes)
- [ ] Spacing, radii, shadows mapped
- [ ] WOFF2 font files placed in assets/fonts/
- [ ] SCSS references verified

## Screens

### {Screen name} -- node {nodeId}

- Status: {not-started | in-progress | done}
- Target: {page ID N | CPT single | archive}
- Phases:
  - [ ] Phase 1 -- scope confirmed
  - [ ] Phase 2 -- Figma inspected
  - [ ] Phase 3 -- mapping decided
  - [ ] Phase 4 -- built (sections + images wired in)
  - [ ] Phase 5 -- verification passed
- Patterns: {list of p-\* slugs}
- Custom blocks: {list}
- ACF blocks: {list}
- CPTs: {list}
- Notes: {anything worth remembering}

## Shared components (build once, reuse)

- {p-slug-or-block-name} -- used by: {screen1}, {screen2} -- {one-line purpose}

## Deferred / follow-ups

- {item} -- {why deferred, what unblocks it}

## Session log

- {YYYY-MM-DD} -- {what was accomplished}
```
