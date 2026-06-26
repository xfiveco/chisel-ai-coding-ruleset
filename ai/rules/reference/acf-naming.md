# ACF Field Group Naming (canonical — all ACF field groups)

Applies to **every** ACF field group JSON you author, regardless of where it attaches (block, options page, WooCommerce product meta box, term meta). Two independent properties, two independent rules — plus a context-dependent prefix source.

This is the single source of truth. The block reference, the create-acf-block skill, and the create-acf-options skill all point here.

## Rule 1 — `key` (group + every field) = real ACF-style hex hash

`group_` / `field_` followed by 13 hex chars, generated fresh per field — e.g. `group_6a1f3d2c9e740`, `field_6a1f3d2c9e741`. **Never** human-readable keys (`group_contact_fields`, `field_heading`).

**Filename must equal the group key:** `group_{hash}.json`.

Why: the key is the field group's sync identity. ACF's admin UI writes edits back to a JSON file named after the group key. Human-named keys/files break the round-trip — import → edit-in-UI → save-back: the group imports, but UI edits don't persist to disk, and column/wrapper settings (e.g. a repeater sub-field `wrapper.width` of `25%`/`75%`) silently fail to populate. This is identical across all attachment points — it's about ACF's save mechanism, not about blocks.

## Rule 2 — `name` (the field slug) = namespaced, globally unique

Every field `name` carries a namespace prefix so names are globally unique across the project. WPML registers translatable strings by field `name`; duplicate names bleed strings across contexts.

**The prefix SOURCE depends on what the group attaches to:**

| Attaches to (`location` param) | Values stored in | Name prefix source | Example |
| --- | --- | --- | --- |
| `block` (`chisel/{block}`) | `post_content` (delimiter JSON) | block initials — multi-word → word initials, single-word → first 2 letters; collision → later block extends its prefix | `blueprint-process` → `bp_heading`; `slider` → `sl_…`; `blog-posts` after `blueprint-process` → `blp_…` |
| `options_page` | `wp_options` (`options_{name}`) | options page / section slug | `header_cta_text`, `footer_logo`, `social_links` |
| `post_type == product` (WooCommerce — the ONLY sanctioned non-block meta box; Chisel is Gutenberg-first elsewhere) | `wp_postmeta` (`{name}` + `_{name}`) | `product_` or the feature name | `product_spec_sheet`, `product_warranty_years` |
| `taxonomy` (term meta) | `wp_termmeta` | taxonomy slug | `genre_color`, `brand_logo` |

**Sub-fields** (repeater / group) use the **full parent name** as their base, in every context: repeater `bp_steps` → `bp_steps_label`, `bp_steps_title`. Options repeater `social_links` → `social_links_url`, `social_links_label`.

## Rule 3 — `label` stays human-readable

Only `name` is prefixed. `label` is what editors see: `Heading`, `Step label`, `CTA text`. Never prefix labels.

## Rule 4 — bump `modified` on EVERY edit to a field-group JSON

Any time you hand-edit a field-group JSON — adding/removing a field, renaming, changing a preference, fixing a key, anything — set the group's top-level `"modified"` to the current unix timestamp (seconds). This applies to **every** ACF JSON file, not just WPML/preference changes.

Why: ACF only flags a group as **"Sync available"** in *Custom Fields → Field Groups* when the JSON `modified` is **newer** than the copy in the database. If you edit the file but leave `modified` stale (or omit it), the admin shows no change and the edit never reaches the DB — a silent no-op. A brand-new group must include `modified` from creation, or it can never be synced.

- The value is unix **seconds** (not milliseconds), e.g. `"modified": 1782460414`.
- After you sync in the admin, ACFML re-serializes the file and bumps `modified` itself — that overwrite is expected; don't fight it.
- Editing JSON is only half the round-trip: the edit is inert until the group is synced (or re-imported). Always tell the user to sync after a JSON edit.

## Example — one field, all three properties distinct

Block `blueprint-process` → prefix `bp`:

```json
{
  "key": "field_7b3c4e1a9f852",   // hex hash, globally unique (Rule 1)
  "label": "Heading",             // human-readable, shown to editors (Rule 3)
  "name": "bp_heading",           // namespaced slug, WPML-safe (Rule 2)
  "type": "text"
}
```

## Skills

- Block field group → [create-acf-block skill](../../skills/chisel-create-acf-block/SKILL.md)
- Options page field group → [create-acf-options skill](../../skills/chisel-create-acf-options/SKILL.md)
- Block file structure + ACF seed-data shape (`_{name}: "field_key"` pointers) → [blocks.md](blocks.md)
- Per-field WPML translation preferences (`wpml_cf_preferences`, Expert mode) → [acf-wpml-translation.md](acf-wpml-translation.md)
