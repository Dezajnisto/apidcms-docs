---
title: Динамические страницы (тип dynamic)
slug: dinamicheskie-stranicy
section: content
order: 80
description: "Универсальный тип dynamic"
---

# 📄 Динамические страницы (тип `dynamic`)

Тип `dynamic` — это универсальный тип страницы для работы с любыми таблицами данных. Он заменил старые `blog` и `catalog` и работает с **любой** таблицей.

---

## Как это работает

Вы указываете:
- `source_table` — имя таблицы с данными
- `template` — имя шаблона (без `.html.twig`)

CMS автоматически:
- Показывает **список записей** по адресу страницы
- Показывает **отдельную запись** по адресу `/{url}/{slug}`

---

## Настройка в navigation

```json
{
  "source_table": "docs",
  "template": "docs",
  "sort": {"field": "sort_order", "order": "ASC"},
  "items_per_page": 50
}
```

| Поле | Описание |
|------|----------|
| `source_table` | Имя таблицы в БД |
| `template` | Базовое имя шаблона |
| `sort.field` | Поле для сортировки |
| `sort.order` | `ASC` или `DESC` |
| `items_per_page` | Записей на странице (для списка) |

---

## Шаблоны

Для каждого динамического типа нужно **два шаблона**:

| Режим | Шаблон | Переменные |
|-------|--------|-----------|
| Список (`/url`) | `{template}.html.twig` | `items`, `nav_item`, `current_page`, `total_pages`, `total_count` |
| Запись (`/url/{slug}`) | `{template}_single.html.twig` | `item`, `nav_item`, `prev_item`, `next_item`, `items` |

### Список (`docs.html.twig`)

```twig
{% for p in items %}
  <a href="{{ url(nav_item.url ~ '/' ~ p.slug) }}">{{ p.title }}</a>
{% endfor %}
```

### Отдельная запись (`docs_single.html.twig`)

```twig
<h1>{{ item.title }}</h1>
<div>{{ item.content|markdown_to_html|raw }}</div>
```

> 💡 **Новое:** `items` теперь доступен и в `_single` шаблоне — можно строить боковую навигацию по всем записям.

---

## Примеры из реальных проектов

- **apidcms.dezajno.ru** — `docs` → таблица `docs`, документация
- **prostostihi.ru** — `examples` → таблица `poetry`, примеры стихов


---

## 🔧 Авто-поддерживаемые колонки

Dynamic-страницы автоматически работают со следующими колонками в таблице. Если колонка есть — она используется, если нет — пропускается.

| Колонка | Тип | Назначение |
|----------|------|-------------|
| `slug` | TEXT | URL-идентификатор записи. По нему строится URL отдельной страницы: `/{url}/{slug}`. Если нет slug — используется `id` |
| `status` | TEXT | Фильтрация: в список и на отдельную страницу попадают только записи со `status = 'active'` |
| `sort_order` | INTEGER | Порядок сортировки в списке. Приоритетнее `created_at`. Также используется для prev/next-навигации |
| `created_at` | DATETIME | Резервная сортировка и prev/next-навигация, если нет `sort_order` |
| `views_count` | INTEGER | **Новое в v1.3.7:** авто-инкремент при каждом просмотре записи. Ничего настраивать не нужно — просто добавьте колонку в таблицу |

### Использование views_count

```sql
-- Добавить колонку в таблицу
ALTER TABLE my_table ADD COLUMN views_count INTEGER DEFAULT 0;
```

В шаблоне `_single.html.twig`:

```twig
<span>👁️ {{ item.views_count|default(0) }} просмотров</span>
```

Значение увеличивается автоматически при каждом открытии страницы записи.

---

## 🔍 GET-фильтры (get_filters)

Dynamic-страницы поддерживают фильтрацию через GET-параметры. Настраивается в `page_config` через ключ `get_filters`.

### Настройка

```json
{
  "source_table": "catalog",
  "template": "catalog",
  "sort": {"field": "sort_order", "order": "ASC"},
  "items_per_page": 10,
  "get_filters": {
    "category": "category",
    "tool": "ai_tool",
    "difficulty": "difficulty"
  }
}
```

| Ключ | Значение |
|-----|---------|
| `get_filters` | Объект: `"GET-параметр" => "колонка в таблице"` |

### Как это работает

1. Пользователь переходит по URL `?category=text&tool=GPT`
2. CMS добавляет в SQL: `WHERE category = 'text' AND ai_tool = 'GPT'`
3. `total_count` и пагинация учитывают фильтры
4. Фильтры комбинируются друг с другом и с `filters` (жёсткие фильтры из конфига)

### Пример: ссылки-фильтры в Twig

```twig
{% set qs_tool = '' %}
{% if _GET.tool %}{% set qs_tool = '&tool=' ~ _GET.tool %}{% endif %}

<a href="?category=text{{ qs_tool }}"
   class="{% if _GET.category == 'text' %}active{% endif %}">
  Текст
</a>
```

> Переменная `_GET` доступна во всех шаблонах начиная с v1.3.7.
---

## 📊 Сортировка (sort_options)

Dynamic-страницы поддерживают переключение сортировки через GET-параметр `?sort=`. Настраивается в `page_config` через ключ `sort_options`.

### Настройка

```json
{
  "sort": {"field": "created_at", "order": "DESC"},
  "sort_options": {
    "newest":  {"field": "created_at",   "order": "DESC"},
    "popular": {"field": "views_count",  "order": "DESC"},
    "rating":  {"field": "copies_count", "order": "DESC"}
  }
}
```

| Ключ | Значение |
|-----|---------|
| `sort` | Сортировка по умолчанию (`field`, `order`) |
| `sort_options` | Доступные варианты сортировки: `"ключ" => {"field": "...", "order": "ASC/DESC"}` |

### Как это работает

1. Пользователь переходит по URL `?sort=popular`
2. CMS ищет ключ `popular` в `sort_options`
3. Проверяет, что колонка существует в таблице (защита от инъекций)
4. Применяет `ORDER BY views_count DESC`
5. Параметр `sort` сохраняется в пагинации и комбинируется с фильтрами

### Пример: ссылки сортировки в Twig

```twig
{% set qs = '' %}
{% if _GET.category %}{% set qs = qs ~ '&category=' ~ _GET.category %}{% endif %}

<a href="?sort=newest{% if qs %}&{{ qs|trim('&') }}{% endif %}"
   class="{% if _GET.sort == 'newest' or _GET.sort is empty %}active{% endif %}">
  Сначала новые
</a>
<a href="?sort=popular{% if qs %}&{{ qs|trim('&') }}{% endif %}"
   class="{% if _GET.sort == 'popular' %}active{% endif %}">
  Популярные
</a>
```

### Безопасность

- `sort_options` должен быть явно задан в `page_config` — без него `?sort=` игнорируется
- Имя колонки валидируется через `PRAGMA table_info` — произвольный SQL невозможен
- Существующие проекты без `sort_options` работают как раньше
---

## 🧩 Twig-функции для доступа к данным

В любом шаблоне фронтенда доступны функции для получения данных из других таблиц.

| Функция | Описание |
|---------|----------|
| `get_record(table, id)` | Одна запись по ID |
| `get_records(table, ids)` | Несколько записей: `'1,3,5'` |
| `get_all(table, orderBy, orderDir)` | Все записи, с сортировкой |
| `get_setting(key)` | Значение системной настройки |

### Пример: топ-N записей из таблицы

```twig
{% set count = 0 %}
{% for item in get_all('catalog', 'copies_count', 'DESC') %}
  {% if item.status == 'active' and count < 4 %}
    {% set count = count + 1 %}
    <a href="/catalog/{{ item.slug }}">{{ item.title }}</a>
  {% endif %}
{% endfor %}
```

### json_decode фильтр

Для колонок с JSON-данными используйте фильтр `|json_decode`:

```twig
{% set features = plan.features|json_decode %}
{% for f in features %}<li>{{ f }}</li>{% endfor %}
```

> Функции `get_*` и фильтр `json_decode` доступны начиная с v1.3.7.
