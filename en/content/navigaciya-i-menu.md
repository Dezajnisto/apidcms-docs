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
| `page_type` | TEXT | Page type: `page`, `dynamic`, `form`, `ai`, `landing`, `category`, `raw` |
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
| `raw` | File as-is (`robots.txt`, `sitemap.xml`) | — | `/robots.txt` |

---

## `raw` — serve a file as-is

The `raw` type renders a theme template **without the site layout** and sends it
with the requested `Content-Type`. Meant for machine-readable files:
`robots.txt`, `sitemap.xml`, search engine verification files.

| Field | Value |
|------|----------|
| `url` | `robots.txt` — address without a leading slash |
| `page_type` | `raw` |
| `template` | template name: `robots` → `robots.html.twig` |
| `location` | `none` — keeps the file out of the menu |
| `status` | `active` |
| `page_config` | `{"content_type": "text/plain; charset=utf-8"}` |

The template lives in the theme: `themes/<theme>/front/robots.html.twig`. If the
theme has no such file, the core template `core/views/front/robots.html.twig` is
used — it serves `User-agent: *` and `Disallow:`.

`content_type` defaults to `text/plain; charset=utf-8`. For `sitemap.xml`:

```json
{"content_type": "application/xml; charset=utf-8"}
```

and `template` — `sitemap.xml.html.twig`. A name with the extension
(`sitemap.xml.twig`) is used as-is, `.html.twig` is not appended.

Inside the template you get `nav_item`, `_GET`, `site_lang`, `site_title`,
`site_description`, `current_path` and the usual `get_setting()` and `url()`
functions.

Twig escaping applies to variable output (`{{ }}`) only; literal template text is
emitted as-is. When injecting a setting into `sitemap.xml`, add `|raw` —
otherwise `<`, `>` and `&` become HTML entities.

If `template` is empty or the file is missing, the page returns 404.

⚠️ `page_type` is **not a free-form field**. A value that is not in the table above
is handled as a dynamic list (`dynamic`) and returns 404 without `source_table`.
That is why robots.txt and sitemap.xml need `raw`.

⚠️ A real `robots.txt` file inside `www/` is served by nginx directly, the request
never reaches the CMS. A CMS-managed file must exist only as a `navigation` record.

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
