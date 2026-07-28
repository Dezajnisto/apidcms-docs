---
title: Привязка шаблона к странице
slug: privyazka-shablona
section: templates
order: 250
description: "Привязка шаблона к странице"
---

# 🔗 Привязка шаблона к странице

---

## Способ 1: через колонку `template`

В таблице **navigation**, в записи страницы, укажите имя шаблона в поле `template`:

| `template` | Шаблон списка | Шаблон записи |
|-----------|--------------|---------------|
| `default` | `blog/list.html.twig` | `blog/single.html.twig` |
| `docs` | `docs.html.twig` | `docs_single.html.twig` |
| `my-theme` | `my-theme.html.twig` | `my-theme_single.html.twig` |

Не нужно указывать `.html.twig` — CMS добавит автоматически.

---

## Способ 2: через `page_config`

В поле `page_config` укажите `template` в JSON:

```json
{
  "source_table": "posts",
  "template": "blog-custom",
  "items_per_page": 12
}
```

Приоритет: `page_config.template` > поле `template` в navigation.

---

## Способ 3: кастомный путь

Если шаблон лежит в подпапке:

```
template = "blog/custom"
```

→ CMS ищет: `blog/custom.html.twig` (список) и `blog/custom_single.html.twig` (запись).
