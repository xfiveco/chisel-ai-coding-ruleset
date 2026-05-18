# Coding Conventions

## PHP

- Core namespace: `Chisel\`. Custom: `Chisel\WP\Custom\`.
- Use `HooksSingleton` trait for classes with hooks
- Implement `action_hooks()` and `filter_hooks()` methods
- Factory classes for registration (CPTs, taxonomies, blocks)
- WordPress Coding Standards (PHPCS)

### Boot order (`functions.php` → custom singletons)

1. Composer autoload.
2. Chisel autoloader registers both `core/` and `custom/app/`.
3. Timber initializes.
4. Core singletons boot (AJAX controller, blocks, ACF, ACF blocks, assets, comments, site, sidebars, theme, CPTs, taxonomies, Twig, plugin integrations, Timber cache).
5. `custom/functions.php` boots project-specific singletons from `custom/app/WP/` via `get_instance()` calls.

### Namespace ↔ path mapping

The autoloader strips the `Custom` segment when resolving classes inside `custom/app/`. `Custom` is a namespace marker, **not** a directory name:

- `Chisel\WP\Custom\Assets` → `custom/app/WP/Assets.php`
- `Chisel\WP\Custom\Site` → `custom/app/WP/Site.php`
- `Chisel\Timber\Custom\ChiselPost` → `custom/app/Timber/ChiselPost.php`

When creating a new feature class, mirror an existing file like `custom/app/WP/Assets.php` for the namespace + `HooksSingleton` boilerplate, then add a `get_instance()` line to `custom/functions.php`.

### Existing helpers (check before writing utilities)

Helper classes in `core/Helpers/` — most have static methods. Read them before reinventing a utility:

| Helper                         | Purpose                                                              |
| ------------------------------ | -------------------------------------------------------------------- |
| `ThemeHelpers`                 | Theme.json color palette access, post thumbnails registration        |
| `AssetsHelpers`                | Asset registration / enqueueing / dependency resolution              |
| `ImageHelpers`                 | Responsive images, srcset, `ChiselImage` helpers                     |
| `BlocksHelpers`                | ACF block render callback, block inline CSS                          |
| `AcfHelpers`                   | ACF field group helpers                                              |
| `DataHelpers`                  | Array/string sanitization, structured-data utilities                 |
| `CacheHelpers`                 | Timber cache expiry resolution                                       |
| `AjaxHelpers`                  | REST/AJAX response shaping                                           |
| `CommentsHelpers`              | Comment list rendering                                               |
| `YoastHelpers`                 | `breadcrumbs()` and Yoast availability checks                        |
| `WoocommerceHelpers`           | WC product / category helpers                                        |
| `GravityFormsHelpers`          | GF availability checks, form rendering helpers                       |

JS helpers: `src/scripts/modules/utils.js` for shared frontend utilities (DOM, throttling, etc.). Import from there before writing new ones.

## JavaScript

| Context                                      | Style                                  |
| -------------------------------------------- | -------------------------------------- |
| Frontend (`view.js`, `src/scripts/modules/`) | Class-based vanilla ES6, no frameworks |
| Editor (`edit.js`, `src/scripts/editor/`)    | React/JSX with `@wordpress/*` packages |
| App entry (`src/scripts/app.js`)             | Import and instantiate modules         |

- Frontend: `js-*` prefixed selectors (not CSS classes), `DOMContentLoaded` bootstrap
- Editor: `useBlockProps()`, `InspectorControls` for sidebar, `InnerBlocks` for nested content
- No TypeScript — pure ES6+

## SCSS / CSS

- **ITCSS**: generic → elements → objects → components → blocks → widgets → utilities
- **BEM**: `.c-component`, `.c-component--modifier`, `.c-component__element`
- **Prefixes**: `c-` components, `o-` objects, `u-` utilities, `b-` blocks, `p-` patterns, `is-`/`has-` state
- Modern Sass: `@use` and `@forward` (not `@import`)
- **Always** `@use '~design' as *;` at top of every SCSS file — use helper functions, never raw CSS vars

### Design tool helpers

```scss
get-color('primary')        // var(--wp--preset--color--primary)
get-font-family('headings') // var(--wp--preset--font-family--headings)
get-font-size('large')      // var(--wp--preset--font-size--large)
get-gap('normal')           // var(--wp--custom--gap--normal)
get-margin('large')
get-padding('small')
get-border-radius('small')
get-line-height('medium')
get-box-shadow('3')
get-letter-spacing('loose')
get-transition()            // default 'normal'
rgba-color('primary', 50%)  // semi-transparent

@include bp('large') {
} // min-width breakpoint
@include bp-down('medium'); // max-width breakpoint
```

Full tool list: `src/design/tools/`.

- Block styles: `src/styles/blocks/_core-{name}.scss`
- Drop new partials into the relevant folder (`src/styles/{blocks,components,elements,objects,utilities,generic,vendor,...}`) — the `_index.scss` barrels are **auto-generated by the build**; never edit them by hand

## Twig / Timber

- Base layout: `views/base.twig` with `{% block body %}`, `{% block header %}`, `{% block main %}`, `{% block footer %}`
- Components: `views/components/` — include via `{% include 'components/name.twig' %}`
- ACF block templates: `src/blocks-acf/{name}/{name}.twig`
- Context: `{{ fields }}`, `{{ block }}`, `{{ wrapper_attributes }}`, `{{ is_preview }}`
- Use `Timber::context()` for global context
- **All Twig templates in `views/`**. `custom/views/` is legacy/unused — edit `views/` directly.

### Twig rules

Never use raw PHP in Twig. Use one of:

1. Timber built-in: `{{ theme.link }}`, `{{ site.url }}`, `{{ post.link }}`
2. `function()` bridge: `{{ function('wp_head') }}`, `{{ function('get_stylesheet_directory_uri') }}`
3. Registered Twig function: `{{ get_responsive_image() }}`, `{{ get_icon() }}`, `{{ bem() }}`
4. Custom Twig function via `chisel_twig_register_functions`
