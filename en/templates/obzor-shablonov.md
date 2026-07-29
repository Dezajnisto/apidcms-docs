---
title: Templates Overview
slug: obzor-shablonov
section: templates
order: 230
description: "Overview of the Twig templating system in apidcms"
---

# Templates (Twig)

apidcms uses [Twig](https://twig.symfony.com/) — a fast and secure templating engine.

---

## Where They Live

```
front/app/views/
├── base.html.twig          # base layout
├── page.html.twig          # static pages
├── docs.html.twig          # list (example)
├── docs_single.html.twig   # single record (example)
├── form/                   # form templates
└── blog/                   # blog templates (default)
```

---

## How It Works

1. Each template extends `base.html.twig`
2. `base.html.twig` contains the common layout: `<head>`, header, footer
3. Child templates override blocks: `title`, `content`, `scripts`

### base.html.twig (example)

```twig
<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}Site{% endblock %}</title>
</head>
<body>
  {% include 'header.html.twig' %}
  <main>{% block content %}{% endblock %}</main>
  {% include 'footer.html.twig' %}
</body>
</html>
```

### Child Template

```twig
{% extends 'base.html.twig' %}

{% block title %}My Page{% endblock %}

{% block content %}
  <h1>Hello!</h1>
{% endblock %}
```

---

## Available Functions

See full reference: [Twig Functions & Filters](twig-reference.md)

Quick overview: `url()`, `asset()`, `get_setting()`, `get_navigation()`, `render_form()`, `get_record()`, `get_records()`, `get_all()`, `|markdown_to_html`.

---

## Available Variables

See full reference: [Template Variables](template-variables.md)

Variables depend on the page type. Core ones: `items`, `item`, `nav_item`, `page`, `config`.
