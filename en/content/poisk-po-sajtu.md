---
title: Site Search
slug: poisk-po-sajtu
section: content
order: 160
---

# Site Search

The `search` page type adds a search form with results to the site.

## How to Enable

Add a record to the `navigation` table:

- **page_type:** `search`
- **url:** `search` (or any other)
- **location:** `header` (or wherever needed)

The search will automatically work at `/search` (or your chosen url).

## What It Searches

The search runs across **all tables** specified as `source_table` in active `navigation` records, plus always the `pages` table.

From each table, text columns (TEXT, VARCHAR) are selected — excluding system columns: `status`, `slug`, `url`, `image`, `email`, `phone`, `password`, `token`, `api_key`.

## Results

- The title is taken from the `title` → `name` → `summary` column (first one found)
- URL is built automatically: for `pages` — `/{slug}`, for dynamic tables — `/{nav_url}/{item_slug}`
- Snippet: 150-200 characters around the first found word
- Highlight: `<mark>...</mark>` around the search term
- Sorting: by relevance (exact match in title > partial match)

## Template

`search.html.twig` — located in the core (`core_lib/front/app/views/`). A project can override it by creating its own `front/app/views/search.html.twig`.

## Security

- System tables (`system_settings`, `navigation`, `sqlite_sequence`, `visit_stats`) are excluded from search
- Passwords, tokens, API keys — columns excluded by name
- Only searches active records (status='active')
- Minimum query length: 2 characters
