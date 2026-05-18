---
description: Create an ACF block — field-driven repeating content (sliders, testimonials, team grids, stats counters). Editors fill in structured fields rather than composing free-form block content.
---

# Create ACF Block

**Load [reference/blocks.md](../reference/blocks.md) first** — required file list, the build-pipeline rule (`script.js` as webpack entry), `ignoreScripts` rules, and ACF field-data shape (`_{name}: "field_key"` pointers) all live there. This skill is the _how_; reference is the _what_. Then load RULES.md if you haven't.

Use ACF blocks for repeatable, field-driven content. For interactive UI or free-form composition, see [create-block](create-block.md) or [create-pattern](create-pattern.md).

## Procedure

1. **Check CPT ladder first.** If the block displays entity-like repeating content (case studies, team, portfolio, events), run [section-mapping-decisions.md CPT decision](../reference/section-mapping-decisions.md#cpt-decision). If all 3 apply → CPT + CPT-driven block with `latest`/`selected` variants ([create-cpt.md](create-cpt.md) "Pair with a custom block"), NOT a repeater here.
2. **Create the files** at `src/blocks-acf/{block-name}/`. File list and `block.json` script/style key requirements: [reference/blocks.md "ACF Block"](../reference/blocks.md#acf-block-srcblocks-acfname). Use the templates below for `block.json`, the Twig template, `style.scss`, `script.js`, and the ACF field group JSON.
3. **Populate any preset ACF option fields** immediately via `xfive-acf-acf-field-update`.
4. **Run `npm run dev` or `build-scripts` to compile** — Chisel registers blocks from `build/blocks-acf/`, NOT `src/blocks-acf/`. Until the build runs, the block won't appear and the editor will show "your site doesn't include {block-name} block" on existing posts referencing it.
5. **Verify** in editor under "Chisel Blocks", then `xfive-blocks-block-schema` to confirm registration.

## Templates

### block.json

Follow `src/blocks-acf/slider/block.json`:

```json
{
  "name": "chisel/{block-name}",
  "title": "{Block Title}",
  "description": "{description}",
  "category": "chisel-blocks",
  "icon": "{dashicon}",
  "apiVersion": 3,
  "keywords": ["{keyword1}", "{keyword2}"],
  "textdomain": "chisel",
  "acf": {
    "mode": "preview",
    "usePostMeta": false,
    "renderCallback": "\\Chisel\\Helpers\\BlocksHelpers::acf_block_render"
  },
  "supports": {
    "anchor": true,
    "align": ["wide", "full"],
    "alignWide": true,
    "className": true,
    "customClassName": true,
    "multiple": true,
    "reusable": true
  },
  "ignoreScripts": ["script"],
  "script": "file:./script.js",
  "style": ["file:./style-script.css"]
}
```

`script.js` is required as the webpack entry that imports `style.scss` (which produces `style-script.css`). `ignoreScripts: ["script"]` above is the default for SCSS-only blocks — it suppresses script execution while still letting webpack build the CSS.

**With real frontend JS** (carousel init, animation, etc.) — drop `ignoreScripts` so the script loads:

```json
{
  "script": "file:./script.js",
  "style": ["file:./style-script.css"]
}
```

See [reference/blocks.md](../reference/blocks.md#file-structures) "Build-pipeline rule" for the broader principle: every SCSS file must be imported by a JS entry listed in block.json or it will not compile.

### {block-name}.twig

```twig
<div {{ wrapper_attributes }}>
  {% if fields.{field_name} is not empty %}
    <div class="b-{block-name}__inner">
      {% for item in fields.{repeater_field} %}
        <div class="b-{block-name}__item">{{ item.{sub_field} }}</div>
      {% endfor %}
    </div>
  {% else %}
    {% include 'partials/block-edit-button.twig' with {'block_name': '{block-name}'} %}
  {% endif %}
</div>
```

Available context (set by `BlocksHelpers::acf_block_render`):

- `{{ wrapper_attributes }}` — block wrapper HTML attrs
- `{{ fields }}` — all ACF fields from `get_fields()`
- `{{ block }}` — block data
- `{{ is_preview }}` — boolean
- `{{ slug }}` — block slug prefixed with `b-`
- `{{ post_id }}` — current post ID

Image helpers:

- `{{ get_image(fields.image_field) }}` — Timber image object
- `{{ get_responsive_image(fields.image_field, 'large') }}` — responsive img tag

### style.scss

```scss
@use '~design' as *;

.b-{block-name} {
  // BEM naming
  // theme.json tokens via get-* helpers
}
```

### script.js (optional)

```js
import './style.scss';

class BlockName {
  constructor(element) {
    this.element = element;
  }
}

document.addEventListener('DOMContentLoaded', () => {
  const elements = document.querySelectorAll('.b-{block-name}');
  if (!elements.length) return;
  elements.forEach((el) => new BlockName(el));
});
```

### acf-json/group\_{hash}.json

```json
{
  "key": "group_{unique_hash}",
  "title": "{Block Title} Fields",
  "fields": [
    {
      "key": "field_{unique_hash}",
      "label": "{Field Label}",
      "name": "{field_name}",
      "type": "{text|image|repeater|select|etc}",
      "required": 0
    }
  ],
  "location": [[{ "param": "block", "operator": "==", "value": "chisel/{block-name}" }]]
}
```

Generate unique hex strings for hashes.

Common field types: `text`, `textarea`, `wysiwyg`, `image` (return: `id`), `repeater` with `sub_fields`, `select`, `radio`, `true_false`, `link`, `group` with `sub_fields`.

## Validation defaults

Default `"required": 0` on all fields and `"min": 0` on repeaters. Required fields and min-row constraints fire validation errors in the editor when the block is first inserted (before the editor reads serialized data) — confusing for editors and blocks the page from saving. If a field is genuinely required, enforce it in Twig (skip rendering the card) rather than at the ACF layer.

## Default alignment

Add via `chisel_editor_scripts` filter inside `custom/app/WP/Assets.php` (the class is already registered in `custom/functions.php` via `get_instance()`). Use the class's existing `filter_hooks()` method — don't add hooks directly in `custom/functions.php` (see [RULES.md "Architecture"](../RULES.md#architecture)):

```php
// custom/app/WP/Assets.php
public function filter_hooks(): void {
    add_filter( 'chisel_editor_scripts', array( $this, 'set_block_default_alignment' ) );
}

public function set_block_default_alignment( array $data ): array {
    $data['editor']['localize']['data']['blocksDefaultAlignment']['chisel/{block-name}'] = 'full';
    return $data;
}
```

## Guidelines

1. ACF field group JSON auto-loads from `src/blocks-acf/{block-name}/acf-json/`.
2. Always populate content via ACF fields in editor — never hardcode in Twig.
3. BEM class: `b-{block-name}`, `__element`, `--modifier`.
4. Use `partials/block-edit-button.twig` for empty state.
