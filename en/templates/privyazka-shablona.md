---
title: Template Assignment
slug: privyazka-shablona
section: templates
order: 250
description: "How to assign custom templates to pages in apidcms"
---

# Template Assignment

---

## Method 1: Via the `template` Column

In the **navigation** table, specify the template name in the `template` field:

| `template` | List Template | Single Template |
|-----------|--------------|-----------------|
| `default` | `blog/list.html.twig` | `blog/single.html.twig` |
| `docs` | `docs.html.twig` | `docs_single.html.twig` |
| `my-theme` | `my-theme.html.twig` | `my-theme_single.html.twig` |

You don't need to include `.html.twig` — the CMS adds it automatically.

---

## Method 2: Via `page_config`

Specify `template` in the `page_config` JSON field:

```json
{
  "source_table": "posts",
  "template": "blog-custom",
  "items_per_page": 12
}
```

Priority: `page_config.template` > `template` field in navigation.

---

## Method 3: Custom Subdirectory

If the template lives in a subfolder:

```
template = "blog/custom"
```

→ CMS looks for: `blog/custom.html.twig` (list) and `blog/custom_single.html.twig` (single record).
