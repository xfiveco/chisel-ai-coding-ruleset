---
name: chisel-create-acf-options
description: Register an ACF Options page or sub-page for global site settings (theme options, social links, footer content, header settings) accessible from any template via `get_field('name', 'option')`.
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
---

# Create ACF Options Page

No paired reference doc — this skill is self-contained.

## Procedure

1. Register options page in `custom/app/WP/Acf.php`
2. Add `options` to Timber context in `custom/app/WP/Site.php` (if not already done)
3. Create ACF field group JSON in `acf-json/group_{hash}.json`
4. Populate fields immediately via `xfive-acf-acf-field-update` (don't leave empty)

## 1. Register options page

Edit `custom/app/WP/Acf.php`:

### Top-level page

```php
public function register_acf_options_pages( $options_pages ) {
    $options_pages[] = array(
        'menu_slug'  => '{menu-slug}',
        'page_title' => __( '{Page Title}', 'chisel' ),
    );
    return $options_pages;
}
```

### Sub-page

```php
public function register_acf_options_sub_pages( $options_sub_pages ) {
    $options_sub_pages[] = array(
        'menu_slug'   => '{sub-page-slug}',
        'page_title'  => __( '{Sub Page Title}', 'chisel' ),
        'menu_title'  => __( '{Menu Title}', 'chisel' ),
        'parent_slug' => '{parent-menu-slug}',
    );
    return $options_sub_pages;
}
```

## 2. Add options to Timber context

In `custom/app/WP/Site.php`:

```php
public function filter_hooks(): void {
    add_filter( 'timber/context', array( $this, 'add_to_context' ), 11 );
    // ... existing hooks
}

public function add_to_context( array $context ): array {
    if ( function_exists( 'get_fields' ) ) {
        $context['options'] = get_fields( 'option' );
    }
    return $context;
}
```

Access in Twig: `{{ options.field_name }}`.

## 3. ACF field group JSON

Create `acf-json/group_{unique-hash}.json`:

```json
{
  "key": "group_{unique_hash}",
  "title": "{Page Title} Fields",
  "fields": [
    {
      "key": "field_{unique_hash}",
      "label": "{Field Label}",
      "name": "{field_name}",
      "type": "{text|image|repeater|select|etc}",
      "required": 0
    }
  ],
  "location": [[{ "param": "options_page", "operator": "==", "value": "{menu-slug}" }]],
  "menu_order": 0,
  "position": "normal",
  "style": "default",
  "label_placement": "top",
  "instruction_placement": "label",
  "hide_on_screen": "",
  "active": true
}
```

Use tabs for large field groups:

```json
{ "key": "field_{hash}", "label": "Header", "name": "", "type": "tab", "placement": "top" }
```

## 4. Populate fields

```
xfive-acf-acf-field-update {
  post_id: "option",
  fields: { "header_cta_text": "Let's talk", "header_cta_url": "#contact" }
}
```

## Common field types

`text`, `textarea`, `wysiwyg`, `url`, `email`, `image` (return: `id`), `file`, `gallery`, `repeater` with `sub_fields`, `group` with `sub_fields`, `select`, `radio`, `checkbox`, `true_false`, `link`, `relationship`, `post_object`, `taxonomy`.

## Accessing in Twig

After adding to context:

```twig
{{ options.header_cta_text }}
{{ options.footer_logo.url }}
{% for item in options.social_links %}
  <a href="{{ item.url }}">{{ item.label }}</a>
{% endfor %}
```

For images, use Chisel helpers:

```twig
{{ get_responsive_image(options.footer_logo, 'medium') }}
```
