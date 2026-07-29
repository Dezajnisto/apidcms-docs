---
title: Multi-Column Sort in Dynamic Lists
slug: multi-column-sort
section: content
order: 115
description: "How to specify a chain of fields for ORDER BY using sort.then in page_config"
---

## sort.then — Secondary Sorting

In `page_config` for dynamic pages, the `sort` key supports an optional `then` — an array of fields for secondary sorting.

This is useful when you need to sort records first by one column, and within identical values — by another.

### Basic Syntax

```json
{
  "source_table": "docs",
  "template": "docs",
  "sort": {
    "field": "category",
    "order": "ASC",
    "then": [
      {"field": "sort_order", "order": "ASC"}
    ]
  },
  "items_per_page": 50
}
```

### Full Example with Multiple then

```json
"sort": {
  "field": "section_title",
  "order": "ASC",
  "then": [
    {"field": "sort_order", "order": "ASC"},
    {"field": "created_at", "order": "DESC"}
  ]
}
```

This configuration will output records sorted by `section_title` (A-Z), within groups by `sort_order` (ascending), and with equal `sort_order` — by `created_at` (newest first).

### Which Fields Can Be Used

Any columns from the source table (`source_table`). Field names are filtered: only `a-z`, `A-Z` and `_` characters are allowed.

If a field is specified incorrectly or does not exist — the SQL query will return an error, and the page will show a 404 (debug mode will show details).

### Backward Compatibility

The old format without `then` still works:

```json
"sort": {"field": "sort_order", "order": "ASC"}
```

If `then` is absent — behavior is exactly the same as before.

### What Changed Technically

Fields in the SQL query are now wrapped in double quotes:

```sql
ORDER BY "section_title" ASC, "sort_order" ASC, "created_at" DESC
```

This prevents conflicts with SQL reserved words (e.g., `order`, `group`, `index`).

### When to Use

- **Catalog by sections** — group by `category`, within by `sort_order`
- **Blog with pinned posts** — first `is_pinned DESC`, then `created_at DESC`
- **Changelog** — group by release type, within by version
- **Table with categories** — first category, within position

### See Also

- [Dynamic Pages](/docs/dinamicheskie-stranicy)
- [page_config — Complete Reference](/docs/page-config-spravochnik)
