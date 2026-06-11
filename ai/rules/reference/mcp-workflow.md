# MCP Workflow (xfive-mcp-chisel)

The `xfive-mcp-chisel` MCP server is the **only** supported path for creating or modifying Gutenberg content.

## Prerequisites

The MCP server is provided by the custom `xfive-mcp` WordPress plugin (Xfive-internal, not on wordpress.org). Before any tool call in this workflow:

1. Confirm the plugin is installed at `wp-content/plugins/xfive-mcp/` and activated in WP admin.
2. Confirm the `xfive-mcp-chisel` server is registered in your agent client's MCP config and the `xfive-mcp-chisel-*` tools appear in your available tool list.

If either is missing: **stop**. Ask the user to install/activate the plugin and configure the MCP server, then restart the agent client. Do not improvise a fallback (PHP seeds, WP-CLI, manual paste are all forbidden — see "Do NOT" below).

## Hard rule

Any Gutenberg content insertion goes through MCP:

- Pages out of patterns from a Figma import
- CPT entry layouts
- Inserting/replacing/moving blocks on existing posts
- "Build this page for me" requests

**Do NOT:**

- Write PHP seed scripts or WP-CLI commands
- Ask user to paste markup into the editor
- Edit `wp_posts.post_content` directly in the database
- Use these tools to modify `patterns/*.php` files (those are source templates, not posts)

## Available tools

| Category             | Tools                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| Posts                | `post-by-title`, `post-get-content`, `post-create`, `post-update`, `post-update-content`, `post-trash` |
| Blocks               | `block-tree` (read), `block-schema` (read)                                                             |
| Media                | `media-upload`, `media-migrate`                                                                        |
| Menus                | `nav-menu-create`                                                                                      |
| ACF                  | `acf-field-update`                                                                                     |
| Options & theme mods | `options-update`                                                                                       |

## Workflow for inserting content

1. **Find or create the post**: `post-by-title` to look up, `post-create` to make new.
2. **Check existing structure** (if editing): `post-get-content` or `block-tree`.
3. **Validate block attributes** before writing markup: `block-schema` on each block type. Catches typos/wrong types. Wrong attributes are silently ignored.
4. **Upload images** via `media-upload` as part of each section build (not deferred). Capture attachment IDs/URLs. If URL returns wrong content-type (e.g. Figma asset URLs), download locally to a temp folder outside the theme (e.g. system temp, or a project-level `_tmp/` directory) then upload via `local_path`.
5. **Generate block markup** as serialized WordPress block grammar. Use preset classes (`has-primary-color`, `is-style-primary`, `has-large-font-size`).
6. **Write content**: ALWAYS via `post-update-content` with `mode:"replace"` and the **full serialized markup of the entire post**. To add a section, fetch current with `post-get-content`, concatenate the new section onto the full existing markup, write the whole thing back. (No partial-block tools — they were removed because index-based mutation was fragile.)
   - **⚠️ NEVER use `mode:"append"` (or rely on it appending).** On this project's `xfive-mcp` plugin it **REPLACES the whole post** — using it to add one section silently wipes every prior section. Confirmed twice. Always do get → concat-onto-full → `mode:"replace"`.
7. **Verify**: after every write, run `block-tree` and **count ALL top-level sections** against what should be there. A `block-tree` taken after a destructive write returns only the surviving block(s) — which falsely reads as a clean parse. Then user reloads in browser.

## Post creation defaults

- **Always create pages as `publish`** (not draft) so user can preview immediately.
- **Set homepage** after creating Home page:
  ```
  xfive-options-options-update {
    type: "option",
    entries: { "show_on_front": "page", "page_on_front": 95 }
  }
  ```

## ACF fields

Populate ACF fields immediately after creating options pages or field groups — don't leave empty.

```
xfive-acf-acf-field-update {
  post_id: "option",  // or numeric post ID
  fields: { "header_cta_text": "Let's talk", "header_cta_url": "#contact" }
}
```

## Theme mods and options

```
xfive-options-options-update {
  type: "theme_mod",  // or "option"
  entries: { "custom_logo": 42 }
}
```

Common use cases:

- **Site logo**: upload via `media-upload` → set `custom_logo` theme mod to attachment ID
- **Front page**: `type: "option"`, `entries: { "show_on_front": "page", "page_on_front": {id} }`
- **Nav menu locations**: `type: "theme_mod"`, `entries: { "nav_menu_locations": { "chisel_main_nav": 3 } }`

## Images

- Upload **as part of each section's build**, not deferred to a final step. Each section must be fully reviewable before moving on.
- Upload without asking for permission — it's a required step.
- SVG `<img>` tags require explicit `width` and `height`.

## Nav menus

`nav-menu-create` may append items to an existing menu with the same name. Check first if the menu already exists to avoid duplicates.
