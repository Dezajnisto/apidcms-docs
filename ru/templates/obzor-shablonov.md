---
title: Обзор шаблонов
slug: obzor-shablonov
section: templates
order: 230
description: "Обзор шаблонов"
---

# 📝 Шаблоны (Twig)

apidcms использует [Twig](https://twig.symfony.com/) — быстрый и безопасный шаблонизатор.

---

## Где находятся

```
front/app/views/
├── base.html.twig          # базовая разметка
├── page.html.twig          # статические страницы
├── docs.html.twig          # список (пример)
├── docs_single.html.twig   # запись (пример)
├── form/                   # шаблоны форм
└── blog/                   # шаблоны блога (default)
```

---

## Как работает

1. Каждый шаблон расширяет `base.html.twig`
2. В `base.html.twig` — общая разметка: `<head>`, шапка, подвал
3. Дочерние шаблоны переопределяют блоки: `title`, `content`, `scripts`

### base.html.twig (пример)

```twig
<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}Сайт{% endblock %}</title>
</head>
<body>
  {% include 'header.html.twig' %}
  <main>{% block content %}{% endblock %}</main>
  {% include 'footer.html.twig' %}
</body>
</html>
```

### Дочерний шаблон

```twig
{% extends 'base.html.twig' %}

{% block title %}Моя страница{% endblock %}

{% block content %}
  <h1>Привет!</h1>
{% endblock %}
```

---

## Доступные функции

См. полный справочник: [Twig-функции и фильтры](twig-reference.md)

Кратко: `url()`, `asset()`, `get_setting()`, `get_navigation()`, `render_form()`, `get_record()`, `get_records()`, `get_all()`, `|markdown_to_html`.

---

## Доступные переменные

См. полный справочник: [Переменные в шаблонах](template-variables.md)

Зависят от типа страницы. Основные: `items`, `item`, `nav_item`, `page`, `config`.
