---
title: Twig Functions & Filters
slug: twig-funkcii-i-filtry
section: templates
order: 260
description: "Complete reference of Twig functions and filters available in apidcms templates"
---

# Twig Functions & Filters

apidcms templates include all standard [Twig](https://twig.symfony.com/) features plus custom functions.

---

## Functions

### `url(path)`

Generates a site page URL.

```twig
<a href="{{ url('docs') }}">Documentation</a>
<a href="{{ url(nav_item.url ~ '/' ~ item.slug) }}">{{ item.title }}</a>
```

### `asset(path)`

Generates a URL for a static file.

```twig
<img src="{{ asset('images/logo.png') }}" alt="Logo">
<link rel="stylesheet" href="{{ asset('css/custom.css') }}">
```

### `get_setting(key)`

Gets a system setting value.

```twig
<title>{{ get_setting('site_title') }}</title>
<meta name="description" content="{{ get_setting('site_description') }}">
```

### `get_navigation()`

Returns an array of all active navigation items.

```twig
{% for item in get_navigation() %}
  <a href="{{ url(item.url) }}">{{ item.title }}</a>
{% endfor %}
```

### `render_form(name, options)`

Embeds a form into any page.

```twig
{{ render_form('contacts', {
    submit_text: 'Send',
    form_class: 'my-form'
}) }}
```

See: [Embedding Forms](forms-embed.md)

### `get_record(table, id)`

Gets a single record by ID.

```twig
{% set author = get_record('users', post.author_id) %}
<span>{{ author.name }}</span>
```

### `get_records(table, ids)`

Gets multiple records by a list of IDs.

```twig
{% set related = get_records('posts', [1, 3, 7]) %}
{% for p in related %}
  <a href="{{ url('blog/' ~ p.slug) }}">{{ p.title }}</a>
{% endfor %}
```

### `get_all(table, orderBy, orderDir)`

Gets all active records from a table.

```twig
{% set reviews = get_all('reviews', 'created_at', 'DESC') %}
```

### `get_pivot(source_table, source_id, relation_name, target_table)`

Gets related records via a many-to-many pivot table.

```twig
{% set cats = get_pivot('catalog', item.id, 'categories', 'categories') %}
{% for cat in cats %}
  <span class="tag">{{ cat.name }}</span>
{% endfor %}
```

**Parameters:**
- `source_table` — entity table (e.g. `catalog`)
- `source_id` — entity ID
- `relation_name` — relation name from `page_config.relations` (e.g. `categories`)
- `target_table` — lookup table (e.g. `categories`)

Returns an array of records from the target table. For single relations (1:M), use `get_record()`.

### `session_id()`

Returns the current session ID (for CSRF tokens, etc.).

```twig
<input type="hidden" name="token" value="{{ session_id() }}">
```

---

## Filters

### `|markdown_to_html`

Parses Markdown to HTML.

```twig
<div class="content">
  {{ item.content|markdown_to_html|raw }}
</div>
```

Safe mode is enabled — raw HTML from Markdown is escaped.

---

## Standard Twig Filters

All built-in Twig filters are available:

| Filter | Example |
|--------|---------|
| `|raw` | Output without escaping |
| `|date('d.m.Y')` | Date formatting |
| `|slice(0, 100)` | String truncation |
| `|striptags` | Strip HTML tags |
| `|length` | String/array length |
| `|default('text')` | Default value |
| `|json_encode` | To JSON |
