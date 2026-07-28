---
title: Twig-функции и фильтры
slug: twig-funkcii-i-filtry
section: templates
order: 260
description: "Twig-функции и фильтры"
---

# 🎨 Twig-функции и фильтры

В шаблонах apidcms доступны все стандартные возможности [Twig](https://twig.symfony.com/) плюс собственные функции.

---

## Функции

### `url(path)`

Генерирует URL страницы сайта.

```twig
<a href="{{ url('docs') }}">Документация</a>
<a href="{{ url(nav_item.url ~ '/' ~ item.slug) }}">{{ item.title }}</a>
```

### `asset(path)`

Генерирует URL для статического файла.

```twig
<img src="{{ asset('images/logo.png') }}" alt="Лого">
<link rel="stylesheet" href="{{ asset('css/custom.css') }}">
```

### `get_setting(key)`

Получает значение системной настройки.

```twig
<title>{{ get_setting('site_title') }}</title>
<meta name="description" content="{{ get_setting('site_description') }}">
```

### `get_navigation()`

Возвращает массив всех активных пунктов меню.

```twig
{% for item in get_navigation() %}
  <a href="{{ url(item.url) }}">{{ item.title }}</a>
{% endfor %}
```

### `render_form(name, options)`

Встраивает форму в любую страницу.

```twig
{{ render_form('contacts', {
    submit_text: 'Отправить',
    form_class: 'my-form'
}) }}
```

Подробнее: [Встраивание формы](forms-embed.md)

### `get_record(table, id)`

Получает одну запись по ID.

```twig
{% set author = get_record('users', post.author_id) %}
<span>{{ author.name }}</span>
```

### `get_records(table, ids)`

Получает несколько записей по списку ID.

```twig
{% set related = get_records('posts', [1, 3, 7]) %}
{% for p in related %}
  <a href="{{ url('blog/' ~ p.slug) }}">{{ p.title }}</a>
{% endfor %}
```

### `get_all(table, orderBy, orderDir)`

Получает все активные записи из таблицы.

```twig
{% set reviews = get_all('reviews', 'created_at', 'DESC') %}
```

### `get_pivot(source_table, source_id, relation_name, target_table)`

Получает связанные записи через many-to-many pivot-таблицу.

```twig
{% set cats = get_pivot('catalog', item.id, 'categories', 'categories') %}
{% for cat in cats %}
  <span class="tag">{{ cat.name }}</span>
{% endfor %}
```

**Параметры:**
- `source_table` — таблица сущности (например `catalog`)
- `source_id` — ID сущности
- `relation_name` — имя связи из `page_config.relations` (например `categories`)
- `target_table` — таблица-справочник (например `categories`)

Возвращает массив записей из целевой таблицы. Для одиночных связей (1:M) используйте `get_record()`.

### `session_id()`

Возвращает ID текущей сессии (для CSRF-токенов и т.д.).

```twig
<input type="hidden" name="token" value="{{ session_id() }}">
```

---

## Фильтры

### `|markdown_to_html`

Парсит Markdown в HTML.

```twig
<div class="content">
  {{ item.content|markdown_to_html|raw }}
</div>
```

Безопасный режим включён — сырой HTML из Markdown экранируется.

---

## Стандартные фильтры Twig

Доступны все встроенные фильтры Twig:

| Фильтр | Пример |
|--------|--------|
| `\|raw` | Вывод без экранирования |
| `\|date('d.m.Y')` | Формат даты |
| `\|slice(0, 100)` | Обрезка строки |
| `\|striptags` | Удаление HTML-тегов |
| `\|length` | Длина строки/массива |
| `\|default('текст')` | Значение по умолчанию |
| `\|json_encode` | В JSON |
