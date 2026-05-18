# Assets, Icons & Hooks

## Asset registration

Managed by `core/WP/Assets.php`. Add custom scripts/styles via filters in `custom/app/WP/Assets.php` (`filter_hooks()` method, `HooksSingleton` trait) — **not** in `custom/functions.php` (see [RULES.md "Architecture"](../RULES.md#architecture)). The filter hooks below all apply:

| Context         | Styles filter                   | Scripts filter            |
| --------------- | ------------------------------- | ------------------------- |
| Frontend        | `chisel_frontend_styles`        | `chisel_frontend_scripts` |
| Frontend footer | `chisel_frontend_footer_styles` | —                         |
| Admin           | `chisel_admin_styles`           | `chisel_admin_scripts`    |
| Editor          | `chisel_editor_styles`          | `chisel_editor_scripts`   |
| Login           | `chisel_login_styles`           | `chisel_login_scripts`    |

### Config keys

- **Style**: `src`, `deps`, `ver`, `media`, `condition`, `inline`
- **Script**: `src`, `deps`, `ver`, `strategy` (defer/async), `condition`, `localization`, `inline`

Scripts default to `{'in_footer': true, 'strategy': 'defer'}`.

### Pre-enqueue filters

Modify before enqueueing: `chisel_pre_enqueue_{context}_styles/scripts`.
Per-asset control: `chisel_enqueue_{context}_style/script` — return `false` to skip.

### Performance filters

- `chisel_async_scripts` — handles to load async
- `chisel_defer_scripts` — handles to load deferred
- `chisel_preload_style` — modify style tag for preloading

### Default assets

| Context  | Styles       | Scripts                               |
| -------- | ------------ | ------------------------------------- |
| Frontend | `main.css`   | `app.js` (with REST API localization) |
| Admin    | `admin.css`  | `admin.js` (with ACF color palette)   |
| Editor   | `editor.css` | `editor.js` (with icon labels)        |
| Login    | `login.css`  | `login.js` (with logo data)           |

Handles prefixed with `chisel-`. Build outputs to `build/scripts/` and `build/styles/`. Webpack generates `.asset.php` files for automatic deps + cache-busting.

### HMR / Fast Refresh

Active when `WP_DEBUG` + `SCRIPT_DEBUG` + `WP_ENVIRONMENT_TYPE='development'`. Enqueues `build/runtime.js`, creates companion JS files for CSS hot-reload.

## Icon system

Two modes via `CHISEL_USE_ICONS_MODULE` constant:

| Mode             | Source          | Path                                 |
| ---------------- | --------------- | ------------------------------------ |
| Source (default) | Individual SVGs | `assets/icons-source/{name}.svg`     |
| Module           | Compiled sprite | `assets/icons/icons.svg#{name}-view` |

Color variants in `assets/icons-source/color/{name}.svg`.

**Usage in Twig**: `{{ get_icon({ name: 'arrow', alt: 'Next' }) }}`

### Parameters

`name` (required), `inline`, `rectangle`, `force_mono`, `alt`, `is_css`, `color`.

### CSS classes

`.o-icon`, `.o-icon--{name}`, `.o-icon--inline`, `.o-icon--color`, `.o-icon--mono`.

Monochromatic icons use CSS `mask-image` (colorable via CSS). Color icons use `background-image`.

Static icons for buttons are listed in `src/design/settings/_index.scss` as `$static-icons`. Button system uses `has-icon-{name}` classes with `icon-svg()` mixin.

## Chisel hooks reference

The lists below cover the load-bearing hooks. **Not exhaustive** — for less common hooks (CPT/taxonomy defaults like `chisel_default_post_type_supports_{slug}`, per-asset enqueue gates like `chisel_enqueue_frontend_script`, loader-behavior hooks like `chisel_async_scripts` / `chisel_preload_styles_start_with`, block hooks like `chisel_styles_inline_size_limit`, etc.), grep `core/` for `apply_filters` / `do_action` to find the canonical set.

### Theme setup

| Hook                           | Type   | Purpose                            |
| ------------------------------ | ------ | ---------------------------------- |
| `chisel_nav_menus`             | filter | Modify registered navigation menus |
| `chisel_sidebars`              | filter | Modify sidebar registrations       |
| `chisel_sidebar_content`       | filter | Filter sidebar widget content      |
| `chisel_custom_post_types`     | filter | Register CPTs                      |
| `chisel_custom_taxonomies`     | filter | Register taxonomies                |
| `chisel_acf_options_pages`     | filter | Register ACF options pages         |
| `chisel_acf_options_sub_pages` | filter | Register ACF options sub-pages     |

### Twig

| Hook                             | Type   | Purpose                          |
| -------------------------------- | ------ | -------------------------------- |
| `timber/locations`               | filter | Add Twig template locations      |
| `timber/context`                 | filter | Add data to global Twig context  |
| `timber/post/classmap`           | filter | Map post types to Timber classes |
| `timber/term/classmap`           | filter | Map taxonomies to Timber classes |
| `chisel_twig_register_functions` | action | Register custom Twig functions   |
| `chisel_twig_register_filters`   | action | Register custom Twig filters     |
| `chisel_twig_register_tests`     | action | Register custom Twig tests       |

### Cache

| Hook                       | Purpose                          |
| -------------------------- | -------------------------------- |
| `chisel_cache_expiry`      | Cache duration                   |
| `chisel_cache_everything`  | Cache all contexts               |
| `chisel_environment_cache` | Environment-specific cache rules |
