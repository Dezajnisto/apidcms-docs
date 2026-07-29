---
title: Navigation and Menu
slug: navigaciya-i-menu
section: content
order: 90
description: "Navigation and menu"
---

# 🧭 Navigation and Menu

The `navigation` table is the heart of apidcms routing. Each record is one site page.

---

## Table Fields

| Field | Type | Description |
|------|-----|----------|
| `title` | TEXT | Page name (in the menu and `<title>`) |
| `url` | TEXT | Address: `docs`, `/about`, `blog` |
| `page_type` | TEXT | Page type: `page`, `dynamic`, `form`, `ai`, `landing`, `category` |
| `source_table` | TEXT | Name of the data table (for `dynamic`, `form`, `landing`) |
| `page_config` | TEXT | JSON with page settings |
| `template` | TEXT | Template name: `default` or custom (e.g. `docs`) |
| `location` | TEXT | `header` / `footer` / `none` |
| `menu_order` | INTEGER | Order in the menu |
| `parent_id` | INTEGER | ID of the parent item (for nested menus) |
| `status` | TEXT | `active` / `inactive` |

---

## Page Types

| `page_type` | Purpose | `source_table` | Example URL |
|-------------|-----------|----------------|------------|
| `page` | Static page | — | `/about` |
| `dynamic` | Dynamic list + records by slug | `docs` | `/docs`, `/docs/install` |
| `form` | Contact form | `contacts` | `/contact` |
| `ai` | AI chat | — | `/chat` |
| `landing` | Landing page from sections | `sections` | `/` |
| `category` | Filtered list | `posts` | `/blog/category/news` |

---

## How to Create a New Page

1. Go to Admin Panel → Tables → **navigation**
2. Click **«Add Record»**
3. Fill in the fields:
   - `title` — what it will be called
   - `url` — address (without leading slash: `docs`)
   - `page_type` — select the type
   - `source_table` — table with data
   - `template` — `default` or your own template name
   - `page_config` — JSON with settings (optional)
4. `status` — `active`
5. Save

Done. The page is immediately accessible at the specified address.

---

## `page_config` — JSON Settings

Detailed reference: [page_config](page_config.md)

```json
{
  "source_table": "docs",
  "template": "docs",
  "sort": {"field": "sort_order", "order": "ASC"},
  "items_per_page": 20
}
```

---

## `template` — How CMS Chooses the Template

| Value | List Template | Record Template |
|----------|--------------|---------------|
| `default` | `blog/list.html.twig` | `blog/single.html.twig` |
| `docs` | `docs.html.twig` | `docs_single.html.twig` |
| `my/page` | `my/page.html.twig` | `my/page_single.html.twig` |

If `template` is specified both in `page_config` and the `template` column of the navigation table — `page_config` takes priority.
