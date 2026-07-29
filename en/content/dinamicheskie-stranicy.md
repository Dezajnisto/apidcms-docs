---
title: Dynamic Pages (type dynamic)
slug: dinamicheskie-stranicy
section: content
order: 80
description: "Universal dynamic type"
---

# 📄 Dynamic Pages (type `dynamic`)

The `dynamic` type is a universal page type for working with any data table. It replaced the old `blog` and `catalog` types and works with **any** table.

---

## How It Works

You specify:
- `source_table` — the name of the table with data
- `template` — the template name (without `.html.twig`)

CMS automatically:
- Shows a **list of records** at the page URL
- Shows an **individual record** at `/{url}/{slug}`

---

## Configuration in navigation

```json
{
  "source_table": "docs",
  "template": "docs",
  "sort": {"field": "sort_order", "order": "ASC"},
  "items_per_page": 50
}
```

| Field | Description |
|------|----------|
| `source_table` | Database table name |
| `template` | Base template name |
| `sort.field` | Sort field |
| `sort.order` | `ASC` or `DESC` |
| `items_per_page` | Records per page (for list view) |

---

## Templates

Each dynamic type requires **two templates**:

| Mode | Template | Variables |
|-------|--------|-----------|
| List (`/url`) | `{template}.html.twig` | `items`, `nav_item`, `current_page`, `total_pages`, `total_count` |
| Record (`/url/{slug}`) | `{template}_single.html.twig` | `item`, `nav_item`, `prev_item`, `next_item`, `items` |

### List (`docs.html.twig`)

```twig
{% for p in items %}
  <a href="{{ url(nav_item.url ~ '/' ~ p.slug) }}">{{ p.title }}</a>
{% endfor %}
```

### Single Record (`docs_single.html.twig`)

```twig
<h1>{{ item.title }}</h1>
<div>{{ item.content|markdown_to_html|raw }}</div>
```

> 💡 **New:** `items` is now also available in the `_single` template — you can build sidebar navigation across all records.

---

## Examples from Real Projects

- **apidcms.dezajno.ru** — `docs` → `docs` table, documentation
- **prostostihi.ru** — `examples` → `poetry` table, poetry examples

---

## 🔧 Auto-Supported Columns

Dynamic pages automatically work with the following columns in the table. If a column exists — it is used; if not — it is skipped.

| Column | Type | Purpose |
|----------|------|-------------|
| `slug` | TEXT | URL identifier of the record. Used to build the individual page URL: `/{url}/{slug}`. If no slug — `id` is used |
| `status` | TEXT | Filtering: only records with `status = 'active'` appear in the list and on individual pages |
| `sort_order` | INTEGER | Sort order in the list. Takes priority over `created_at`. Also used for prev/next navigation |
| `created_at` | DATETIME | Fallback sort and prev/next navigation if `sort_order` is absent |
| `views_count` | INTEGER | **New in v1.3.7:** auto-increment on each record view. No configuration needed — just add the column to the table |

### Using views_count

```sql
-- Add column to the table
ALTER TABLE my_table ADD COLUMN views_count INTEGER DEFAULT 0;
```

In the `_single.html.twig` template:

```twig
<span>👁️ {{ item.views_count|default(0) }} views</span>
```

The value increments automatically each time the record page is opened.

---

## 🔍 GET Filters (get_filters)

Dynamic pages support filtering via GET parameters. Configured in `page_config` with the `get_filters` key.

### Configuration

```json
{
  "source_table": "catalog",
  "template": "catalog",
  "sort": {"field": "sort_order", "order": "ASC"},
  "items_per_page": 10,
  "get_filters": {
    "category": "category",
    "tool": "ai_tool",
    "difficulty": "difficulty"
  }
}
```

| Key | Value |
|-----|---------|
| `get_filters` | Object: `"GET parameter" => "table column"` |

### How It Works

1. User navigates to `?category=text&tool=GPT`
2. CMS adds to SQL: `WHERE category = 'text' AND ai_tool = 'GPT'`
3. `total_count` and pagination respect filters
4. Filters combine with each other and with `filters` (hard filters from config)

### Example: filter links in Twig

```twig
{% set qs_tool = '' %}
{% if _GET.tool %}{% set qs_tool = '&tool=' ~ _GET.tool %}{% endif %}

<a href="?category=text{{ qs_tool }}"
   class="{% if _GET.category == 'text' %}active{% endif %}">
  Text
</a>
```

> The `_GET` variable is available in all templates since v1.3.7.

---

## 📊 Sorting (sort_options)

Dynamic pages support switching sort via the `?sort=` GET parameter. Configured in `page_config` with the `sort_options` key.

### Configuration

```json
{
  "sort": {"field": "created_at", "order": "DESC"},
  "sort_options": {
    "newest":  {"field": "created_at",   "order": "DESC"},
    "popular": {"field": "views_count",  "order": "DESC"},
    "rating":  {"field": "copies_count", "order": "DESC"}
  }
}
```

| Key | Value |
|-----|---------|
| `sort` | Default sort (`field`, `order`) |
| `sort_options` | Available sort options: `"key" => {"field": "...", "order": "ASC/DESC"}` |

### How It Works

1. User navigates to `?sort=popular`
2. CMS looks up the key `popular` in `sort_options`
3. Validates the column exists in the table (injection protection)
4. Applies `ORDER BY views_count DESC`
5. The `sort` parameter is preserved in pagination and combines with filters

### Example: sort links in Twig

```twig
{% set qs = '' %}
{% if _GET.category %}{% set qs = qs ~ '&category=' ~ _GET.category %}{% endif %}

<a href="?sort=newest{% if qs %}&{{ qs|trim('&') }}{% endif %}"
   class="{% if _GET.sort == 'newest' or _GET.sort is empty %}active{% endif %}">
  Newest first
</a>
<a href="?sort=popular{% if qs %}&{{ qs|trim('&') }}{% endif %}"
   class="{% if _GET.sort == 'popular' %}active{% endif %}">
  Most popular
</a>
```

### Security

- `sort_options` must be explicitly defined in `page_config` — without it, `?sort=` is ignored
- Column names are validated via `PRAGMA table_info` — arbitrary SQL is impossible
- Existing projects without `sort_options` work as before

---

## 🧩 Twig Functions for Data Access

Frontend templates have access to functions for retrieving data from other tables.

| Function | Description |
|---------|----------|
| `get_record(table, id)` | Single record by ID |
| `get_records(table, ids)` | Multiple records: `'1,3,5'` |
| `get_all(table, orderBy, orderDir)` | All records, with sorting |
| `get_setting(key)` | System setting value |

### Example: top N records from a table

```twig
{% set count = 0 %}
{% for item in get_all('catalog', 'copies_count', 'DESC') %}
  {% if item.status == 'active' and count < 4 %}
    {% set count = count + 1 %}
    <a href="/catalog/{{ item.slug }}">{{ item.title }}</a>
  {% endif %}
{% endfor %}
```

### json_decode filter

For columns with JSON data, use the `|json_decode` filter:

```twig
{% set features = plan.features|json_decode %}
{% for f in features %}<li>{{ f }}</li>{% endfor %}
```

> The `get_*` functions and `json_decode` filter are available since v1.3.7.
