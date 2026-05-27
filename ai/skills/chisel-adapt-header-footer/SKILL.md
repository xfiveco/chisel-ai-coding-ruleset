---
name: chisel-adapt-header-footer
description: Adapt header, footer, navigation, and logo to match a target spec. Headers/footers are Twig templates (not patterns) in Chisel. Spec source can be Figma, static assets, or user prompt.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - AskUserQuestion
  - TodoWrite
  - mcp__xfive-mcp-chisel__*
  - mcp__plugin_figma_figma__*
---

# Adapt Header & Footer

Headers and footers are Twig templates — NOT patterns or blocks. Edit directly in `views/components/`. The `custom/` directory is for PHP overrides only; Twig templates live in `views/`.

## Files involved

| Component  | Twig                             | SCSS                                     | JS                                |
| ---------- | -------------------------------- | ---------------------------------------- | --------------------------------- |
| Header     | `views/components/header.twig`   | `src/styles/components/_header.scss`     | —                                 |
| Footer     | `views/components/footer.twig`   | `src/styles/components/_footer.scss`     | —                                 |
| Logo       | `views/components/logo.twig`     | inside `_header.scss`                    | —                                 |
| Main nav   | `views/components/main-nav.twig` | `src/styles/components/_main-nav.scss`   | `src/scripts/modules/main-nav.js` |
| Nav toggle | —                                | `src/styles/components/main-nav-toggle/` | inside `main-nav.js`              |

## Content source rule

**No hardcoded editable content in Twig.** Map each header/footer element to a source:

| Element               | Source                       | Twig access                          |
| --------------------- | ---------------------------- | ------------------------------------ |
| Logo                  | Customizer (`custom_logo`)   | `{{ logo }}`                         |
| Nav links             | Appearance > Menus           | `{{ get_nav_menu('main_nav') }}`     |
| CTA button (text/URL) | ACF Options > Theme Settings | `{{ options.header_cta_text }}` etc. |
| Footer columns        | ACF Options (repeater)       | `{{ options.footer_link_groups }}`   |
| Social links          | ACF Options (repeater)       | `{{ options.social_links }}`         |
| Copyright             | ACF Options                  | `{{ options.copyright_text }}`       |
| Newsletter            | Gravity Forms shortcode      | —                                    |

## Procedure

### 1. Inspect the spec

**Figma mode:** call `get_design_context` on the header and footer nodes.
**Other modes:** read the mockup or user description.

Extract in all modes: background, border, nav style, CTA button variant, icons, spacing.

### 2. Set up ACF Options (if not done)

Use [create-acf-options](../chisel-create-acf-options/SKILL.md) to register "Theme Settings" page with tabs (Header, Footer, Social, etc.). Add `options` to Timber context:

```php
// custom/app/WP/Site.php
public function add_to_context( array $context ): array {
    if ( function_exists( 'get_fields' ) ) {
        $context['options'] = get_fields( 'option' );
    }
    return $context;
}
```

Create the ACF field group JSON in `acf-json/group_{hash}.json`. Populate fields immediately via `xfive-acf-acf-field-update` with `post_id: "option"` — don't leave empty.

### 3. Edit Twig template

Edit `views/components/header.twig` directly. Use ACF Options for content:

```twig
<header id="header" class="c-header o-wrapper">
  <div class="c-header__inner o-wrapper__inner">
    {% include 'components/logo.twig' %}
    {% include 'components/main-nav.twig' %}
    <div class="c-header__actions">
      {% if options.header_cta_text and options.header_cta_url %}
        <a href="{{ options.header_cta_url }}" class="c-btn c-btn--primary">
          {{ options.header_cta_text }}
        </a>
      {% endif %}
    </div>
  </div>
</header>
```

### 4. Update SCSS

Edit `src/styles/components/_header.scss` (and `_main-nav.scss` as needed). Use `@use '~design' as *;` and design tool helpers.

### 5. Create nav menu

Use `xfive-menus-nav-menu-create` to create the menu and assign to location (`chisel_main_nav` or `chisel_footer_nav`). Warn: the tool may append items to an existing menu — check first.

### 6. Upload and set logo

```
xfive-images-image-upload { image_url or local_path }
xfive-options-options-update { type: "theme_mod", entries: { "custom_logo": <id> } }
```

## Mobile nav toggle

Hamburger color in `src/styles/components/main-nav-toggle/_mnt-settings.scss` (`$hamburger-layer-color`). Mobile overlay background in `_main-nav.scss` under `bp-down(large)`.

## What NOT to do

- Do NOT hardcode phone numbers, emails, CTAs, social URLs, or copyright text in Twig.
- Do NOT build header/footer as a block pattern unless user explicitly wants block-based site editing.
- `custom/views/` is legacy and unused for Twig — edit `views/` directly.
