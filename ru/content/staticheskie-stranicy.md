---
title: Статические страницы (тип page)
slug: staticheskie-stranicy
section: content
order: 70
description: "Статические страницы (тип page)"
---

# 📄 Статические страницы (тип `page`)

Самый простой тип. Одна страница = одна запись в таблице `pages`.

---

## Как создать

1. Админка → Таблицы → **pages** → Добавить запись
2. Заполните:
   - `title` — заголовок
   - `slug` — адрес (`about`, `contacts`)
   - `content` — текст (HTML)
   - `status` — `active`
3. В **navigation** создайте запись:
   - `url` = `about` (совпадает со slug)
   - `page_type` = `page`
4. Готово. Страница доступна по `/about`.

---

## Шаблон

По умолчанию: `page.html.twig`

Можно указать свой через `template` в navigation:

```
template = "my-custom"
```

→ CMS ищет `my-custom.html.twig`

### Переменные в шаблоне

| Переменная | Содержит |
|-----------|----------|
| `page.title` | Заголовок |
| `page.content` | HTML-контент |
| `page.meta_title` | Meta title |
| `page.meta_description` | Meta description |
| `title` | = `page.title` |

### Пример шаблона

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

## Поля таблицы pages

| Поле | Тип | Описание |
|------|-----|----------|
| `title` | TEXT | Заголовок |
| `slug` | TEXT UNIQUE | Адрес |
| `content` | TEXT | HTML-контент |
| `meta_title` | TEXT | SEO title |
| `meta_description` | TEXT | SEO description |
| `status` | TEXT | `active` / `draft` |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |
