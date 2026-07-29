---
title: Template Variables
slug: peremennye-v-shablonah
section: templates
order: 270
description: "List of variables available in different template contexts in apidcms"
---

# Template Variables

Which variables are available depending on the page type.

---

## List View (`dynamic`, list)

Template: `{template}.html.twig`

| Variable | Type | Description |
|----------|------|-------------|
| `items` | array | Records from source_table |
| `nav_item` | object | Current navigation item |
| `current_page` | int | Current pagination page |
| `total_pages` | int | Total pages |
| `total_count` | int | Total records |
| `config` | array | Parsed page_config |
| `query_string` | string | GET params for pagination links |
| `title` | string | title from navigation |

### `nav_item` Fields

| Field | Description |
|-------|-------------|
| `nav_item.title` | Name |
| `nav_item.url` | Address (`docs`, `blog`) |
| `nav_item.page_type` | Type (`dynamic`) |
| `nav_item.source_table` | Table name |

---

## Single Record (`dynamic`, single)

Template: `{template}_single.html.twig`

| Variable | Type | Description |
|----------|------|-------------|
| `item` | array | Current record (all table fields) |
| `nav_item` | object | Navigation item |
| `prev_item` | array/null | Previous record |
| `next_item` | array/null | Next record |
| `items` | array | **All** table records (for sidebar) |
| `title` | string | `item.title` |

---

## Static Page (`page`)

Template: `page.html.twig`

| Variable | Type | Description |
|----------|------|-------------|
| `page` | array | Record from pages table |
| `title` | string | `page.title` |
| `is_home` | bool | `true` for homepage |

---

## Landing (`landing`)

Template: `landing.html.twig` or custom

| Variable | Type | Description |
|----------|------|-------------|
| `items` | array | Sections from source_table |
| `title` | string | Page title |

---

## AI Chat (`ai`)

Template: `ai.html.twig`

| Variable | Type | Description |
|----------|------|-------------|
| `nav_item` | object | Navigation item |
| `title` | string | `nav_item.title` |

---

## Form (`form`)

Template: `form/{template}.html.twig`

Variables depend on FormRenderer and form_config. See: [Creating Forms](forms-create.md).
