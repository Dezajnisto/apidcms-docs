---
title: page_config — справочник
slug: page-config-spravochnik
section: content
order: 100
description: "page_config — справочник"
---

# ⚙️ page_config — справочник

`page_config` — это JSON в колонке `page_config` таблицы `navigation`. Управляет поведением динамических страниц.

---

## Все опции

```json
{
  "source_table": "posts",
  "template": "blog",
  "sort": {"field": "created_at", "order": "DESC"},
  "order_field": "sort_order",
  "order_dir": "DESC",
  "items_per_page": 12,
  "filters": {"category": "news"},
  "get_filters": {"cat": "category_id"}
}
```

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `source_table` | string | (пусто) | Имя таблицы с данными |
| `template` | string | `default` | Имя шаблона |
| `sort.field` | string | `id` | Поле для сортировки |
| `sort.order` | string | `ASC` | Направление: `ASC` / `DESC` |
| `order_field` | string | (из sort) | Переопределяет поле сортировки |
| `order_dir` | string | (из sort) | Переопределяет направление: `ASC` / `DESC` |
| `items_per_page` | int | 10 | Записей на странице |
| `filters` | object | `{}` | Жёсткие фильтры (всегда применяются) |
| `get_filters` | object | `{}` | Фильтры из GET-параметров |

---

## `sort` — сортировка

```json
"sort": {"field": "created_at", "order": "DESC"}
```

Если поле не указано — используется `id ASC`.

### `order_field` и `order_dir` — переопределение сортировки

Позволяют переопределить поле и направление сортировки для конкретной страницы, не меняя `sort`:

```json
"order_field": "sort_order",
"order_dir": "DESC"
```

- `order_field` проверяется по структуре таблицы — нельзя указать несуществующую колонку
- `order_dir` — только `ASC` или `DESC`
- Имеют приоритет над `sort.field` и `sort.order`

Полезно когда нужно вывести записи в порядке, отличном от глобальной сортировки таблицы.

---

## `filters` — жёсткие фильтры

Применяются всегда, пользователь не может их изменить:

```json
"filters": {"status": "active", "category": "news"}
```

→ `WHERE status = 'active' AND category = 'news'`

---

## `get_filters` — фильтры из URL

Ключ — GET-параметр, значение — поле в таблице:

```json
"get_filters": {"cat": "category_id", "tag": "tag_slug"}
```

URL `/blog?cat=5&tag=popular` → `WHERE category_id = '5' AND tag_slug = 'popular'`

---

## `items_per_page` — пагинация

```json
"items_per_page": 20
```

0 или не указано — 10. В шаблоне доступны `current_page`, `total_pages`, `total_count`.

---

## `relations` — связи между таблицами (new in v1.3.10)

Позволяет настроить выпадающие списки в формах редактирования админки для полей, которые ссылаются на другие таблицы (foreign keys).

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id"
  }
}
```

| Ключ | Тип | Описание |
|------|-----|----------|
| `table` | string | Таблица, из которой брать значения для выпадающего списка |
| `label` | string | Колонка для отображения в списке (что увидит пользователь) |
| `value` | string | Колонка для значения (что сохранится в БД), обычно `id` |
| `tree` | bool | `false` | Иерархическое дерево: подкатегории с отступами `&mdash;` (требует колонку `parent_id` в связанной таблице) |
| `search` | bool | `false` | Поле поиска: вместо нативного `<select>` — кастомный combobox с фильтрацией при вводе |
| `type` | string | `"one-to-many"` | Тип связи: `"one-to-many"` (колонка в таблице) или `"many-to-many"` (через pivot-таблицу `entity_relations`) |

### Как это работает

1. Ключ объекта `relations` — имя колонки в текущей таблице (для 1:M) или виртуальное имя связи (для M:M)
2. Для 1:M (по умолчанию): поле отображается как `<select>` с опциями из связанной таблицы
3. Для M:M (`"type": "many-to-many"`): поле отображается как multi-select чекбоксы, данные хранятся в `entity_relations`
4. Текущее значение записи автоматически выбрано в списке

### Пример: категории в каталоге

Таблица `catalog` имеет колонку `category_id`, которая ссылается на таблицу `categories`:

```json
{
  "source_table": "catalog",
  "template": "catalog",
  "relations": {
    "category_id": {
      "table": "categories",
      "label": "name",
      "value": "id"
    }
  }
}
```

В форме редактирования записи каталога поле `category_id` станет выпадающим списком с названиями категорий вместо текстового поля.

### Несколько связей

Можно указать несколько полей:

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id"
  },
  "author_id": {
    "table": "users",
    "label": "username",
    "value": "id"
  }
}
```

### Дерево категорий (tree)

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id",
    "tree": true
  }
}
```

Если таблица `categories` имеет колонку `parent_id`, опции строятся рекурсивно: родительские категории → подкатегории с отступами.

### Поиск по списку (search)

```json
"relations": {
  "category_id": {
    "table": "categories",
    "label": "name",
    "value": "id",
    "search": true
  }
}
```

Поле становится combobox'ом: вводите текст — список фильтруется. `tree` и `search` можно комбинировать.

### Many-to-many связи (type: "many-to-many")

Для связей «многие-ко-многим» (одна запись → несколько категорий):

```json
"relations": {
  "categories": {
    "table": "categories",
    "label": "name",
    "value": "id",
    "type": "many-to-many"
  }
}
```

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `type` | string | `"one-to-many"` | Тип связи: `"one-to-many"` (по умолчанию) или `"many-to-many"` |

**Отличия от 1:M:**

- **Ключ в relations** — виртуальное имя (например `categories`), а не реальная колонка в таблице. Данные хранятся в pivot-таблице `entity_relations`, а не в колонке сущности
- **Админка:** вместо `<select>` — multi-select чекбоксы с поиском
- **Сохранение:** при create/update/delete система автоматически управляет записями в `entity_relations`
- **Фильтрация:** `get_filters` для M:M-полей использует subquery через pivot: `?categories=1,2,3`
- **Шаблон:** для получения связанных записей используйте функцию `get_pivot()` (см. [Twig-функции](#))

**Пример для каталога с несколькими категориями:**

```json
{
  "source_table": "catalog",
  "relations": {
    "categories": {
      "table": "categories",
      "label": "name",
      "value": "id",
      "type": "many-to-many",
      "search": true
    }
  },
  "get_filters": {
    "categories": "categories"
  }
}
```

В шаблоне:

```
{% set cats = get_pivot('catalog', item.id, 'categories', 'categories') %}
{% for cat in cats %}
  <span class="tag">{{ cat.name }}</span>
{% endfor %}
```

Фильтрация: `/catalog?categories=1,2` — покажет записи, у которых есть категории 1 ИЛИ 2.

**Pivot-таблица `entity_relations`:**

Таблица доступна в админке как обычная таблица. Структура:
- `source_table` — таблица сущности (например `catalog`)
- `source_id` — ID сущности
- `relation_name` — имя связи из page_config (например `categories`)
- `target_id` — ID из таблицы-справочника

### Обратная совместимость

Страницы без `relations` работают как раньше — все поля отображаются как обычные текстовые/числовые поля ввода.

---

## Приоритеты

Если опция указана и в `page_config`, и в колонке `template` таблицы navigation — **`page_config` имеет приоритет**.
