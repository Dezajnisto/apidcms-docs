---
title: Caching
slug: keshirovanie
section: settings
order: 330
description: "Caching"
---

# ⚡ Caching

apidcms caches compiled Twig templates to speed up page loads.

---

## When to Clear

After any template changes:

1. Admin panel → **Cache** → **"Clear Cache"**
2. Refresh the site page

---

## What Is Cached

- Compiled `.php` Twig template files (`admin/app/views/cache/`)
- CSS (browser cache, controlled via `custom_css_version`)

---

## Where It Is

```
admin/app/views/cache/    # Twig cache
storage/cache/            # general cache
```

---

## Auto-reload

In development mode (`auto_reload: true` in config), Twig checks for changes automatically.

In production — clear the cache manually after edits.

---

## CSS Versioning

To make browsers pick up new CSS after changes to `custom.css`:

1. Increment `custom_css_version` in `system_settings`
2. Clear the cache

The `base.html.twig` template uses `?v={{ get_setting('custom_css_version') }}` — the browser sees a new version and reloads the file.
