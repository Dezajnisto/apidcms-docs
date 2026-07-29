---
title: Static Pages (page type)
slug: staticheskie-stranicy
section: content
order: 70
description: "Static pages (page type)"
---

# 📄 Static Pages (type `page`)

The simplest type. One page = one record in the `pages` table.

---

## How to Create

1. Admin Panel → Tables → **pages** → Add record
2. Fill in:
   - `title` — heading
   - `slug` — address (`about`, `contacts`)
   - `content` — text (HTML)
   - `status` — `active`
3. In **navigation** create a record:
   - `url` = `about` (matches the slug)
   - `page_type` = `page`
4. Done. The page is accessible at `/about`.

---

## Template

Default: `page.html.twig`

You can specify your own via `template` in navigation:

```
template = "my-custom"
```

→ CMS looks for `my-custom.html.twig`

### Template Variables

| Variable | Contains |
|-----------|----------|
| `page.title` | Heading |
| `page.content` | HTML content |
| `page.meta_title` | Meta title |
| `page.meta_description` | Meta description |
| `title` | = `page.title` |

### Template Example

```twig
{% extends 'base.html.twig' %}

{% block title %}{{ page.meta_title ?? page.title }}{% endblock %}

{% block content %}
<article>
  <h1>{{ page.title }}</h1>
  {{ page.content|raw }}
</article>
{% endblock %}
```

---

## pages Table Fields

| Field | Type | Description |
|------|-----|----------|
| `title` | TEXT | Heading |
| `slug` | TEXT UNIQUE | Address |
| `content` | TEXT | HTML content |
| `meta_title` | TEXT | SEO title |
| `meta_description` | TEXT | SEO description |
| `status` | TEXT | `active` / `draft` |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |
