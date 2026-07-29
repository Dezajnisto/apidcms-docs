---
title: page_config — Reference
slug: page-config-spravochnik
section: content
order: 100
description: "page_config reference"
---

# ⚙️ page_config — Reference

`page_config` is a JSON in the `page_config` column of the `navigation` table. It controls the behavior of dynamic pages.

---

## All Options

```json
{
  "source_table": "posts",
  "template": "blog",
  "sort": {"field": "created_at", "order": "DESC"},
  "order_field": "sort_order",
  "order_dir": "DESC",
  "items_per_page": 12,
  "filters": {"category": "news"},
  "get_filters": {"cat": "category_id"}
}
```

| Key | Type | Default | Description |
|------|-----|-------------|----------|
| `source_table` | string | (empty) | Name of the data table |
| `template` | string | `default` | Template name |
| `sort.field` | string | `id` | Sort field |
| `sort.order` | string | `ASC` | Direction: `ASC` / `DESC` |
| `order_field` | string | (from sort) | Overrides the sort field |
| `order_dir` | string | (from sort) | Overrides the sort direction: `ASC` / `DESC` |
| `items_per_page` | int | 10 | Records per page |
| `filters` | object | `{}` | Hard filters (always applied) |
| `get_filters` | object | `{}` | Filters from GET parameters |

---

## `sort` — Sorting

```json
"sort": {"field": "created_at", "order": "DESC"}
```

If the field is not specified — `id ASC` is used.

### `order_field` and `order_dir` — Sort Override

Allow overriding the sort field and direction for a specific page without changing `sort`:

```json
"order_field": "sort_order",
"order_dir": "DESC"
```

- `order_field` is validated against the table structure — non-existent columns cannot be specified
- `order_dir` — only `ASC` or `DESC`
- Takes priority over `sort.field` and `sort.order`

Useful when you need to display records in a different order than the table's global sort.

---

## `filters` — Hard Filters

Always applied; users cannot change them:

```json
"filters": {"status": "active", "category": "news"}
```

→ `WHERE status = 'active' AND category = 'news'`

---

## `get_filters` — Filters from URL

Key is a GET parameter, value is a table field:

```json
"get_filters": {"cat": "category_id", "tag": "tag_slug"}
```

URL `/blog?cat=5&tag=popular` → `WHERE category_id = '5' AND tag_slug = 'popular'`

---

## `items_per_page` — Pagination

```json
"items_per_page": 20
```

0 or not specified — 10. The template has access to `current_page`, `total_pages`, `total_count`.

---

## `relations` — Table Relations (new in v1.3.10)

Allows configuring dropdown lists in admin panel edit forms for fields that reference other tables (foreign keys).

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id"
  }
}
```

| Key | Type | Description |
|------|-----|----------|
| `table` | string | Table from which to fetch dropdown values |
| `label` | string | Column to display in the list (what the user sees) |
| `value` | string | Column for the value (what gets saved to DB), usually `id` |
| `tree` | bool | `false` | Hierarchical tree: subcategories with indentation `&mdash;` (requires a `parent_id` column in the related table) |
| `search` | bool | `false` | Search field: instead of a native `<select>` — a custom combobox with filtering on input |
| `type` | string | `"one-to-many"` | Relation type: `"one-to-many"` (column in table) or `"many-to-many"` (via pivot table `entity_relations`) |

### How It Works

1. The key of the `relations` object — the column name in the current table (for 1:M) or a virtual relation name (for M:M)
2. For 1:M (default): the field displays as a `<select>` with options from the related table
3. For M:M (`"type": "many-to-many"`): the field displays as multi-select checkboxes, data is stored in `entity_relations`
4. The current record value is automatically selected in the list

### Example: Categories in a Catalog

The `catalog` table has a `category_id` column referencing the `categories` table:

```json
{
  "source_table": "catalog",
  "template": "catalog",
  "relations": {
    "category_id": {
      "table": "categories",
      "label": "name",
      "value": "id"
    }
  }
}
```

In the catalog record edit form, the `category_id` field becomes a dropdown list with category names instead of a text field.

### Multiple Relations

Multiple fields can be specified:

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id"
  },
  "author_id": {
    "table": "users",
    "label": "username",
    "value": "id"
  }
}
```

### Category Tree (tree)

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id",
    "tree": true
  }
}
```

If the `categories` table has a `parent_id` column, options are built recursively: parent categories → subcategories with indentation.

### Search by List (search)

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id",
    "search": true
  }
}
```

The field becomes a combobox: type text — the list filters. `tree` and `search` can be combined.

### Many-to-Many Relations (type: "many-to-many")

For many-to-many relationships (one record → several categories):

```json
"relations": {
  "categories": {
    "table": "categories",
    "label": "name",
    "value": "id",
    "type": "many-to-many"
  }
}
```

| Key | Type | Default | Description |
|------|-----|-------------|----------|
| `type` | string | `"one-to-many"` | Relation type: `"one-to-many"` (default) or `"many-to-many"` |

**Differences from 1:M:**

- **Key in relations** — a virtual name (e.g. `categories`), not an actual column in the table. Data is stored in the pivot table `entity_relations`, not in the entity column
- **Admin panel:** instead of `<select>` — multi-select checkboxes with search
- **Saving:** on create/update/delete, the system automatically manages records in `entity_relations`
- **Filtering:** `get_filters` for M:M fields use a subquery via pivot: `?categories=1,2,3`
- **Template:** use the `get_pivot()` function to retrieve related records (see [Twig Functions](#))

**Example for a catalog with multiple categories:**

```json
{
  "source_table": "catalog",
  "relations": {
    "categories": {
      "table": "categories",
      "label": "name",
      "value": "id",
      "type": "many-to-many",
      "search": true
    }
  },
  "get_filters": {
    "categories": "categories"
  }
}
```

In the template:

```
{% set cats = get_pivot('catalog', item.id, 'categories', 'categories') %}
{% for cat in cats %}
  <span class="tag">{{ cat.name }}</span>
{% endfor %}
```

Filtering: `/catalog?categories=1,2` — shows records that have categories 1 OR 2.

**Pivot table `entity_relations`:**

The table is accessible in the admin panel as a regular table. Structure:
- `source_table` — entity table (e.g. `catalog`)
- `source_id` — entity ID
- `relation_name` — relation name from page_config (e.g. `categories`)
- `target_id` — ID from the reference table

### Backward Compatibility

Pages without `relations` work as before — all fields display as regular text/number input fields.

---

## Priority

If an option is specified both in `page_config` and in the `template` column of the navigation table — **`page_config` takes priority**.
