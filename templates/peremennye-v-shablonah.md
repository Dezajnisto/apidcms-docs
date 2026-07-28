---
title: Переменные в шаблонах
slug: peremennye-v-shablonah
section: templates
order: 270
description: "Переменные в шаблонах"
---

# 📦 Переменные в шаблонах

Какие переменные доступны в зависимости от типа страницы.

---

## Список (`dynamic`, список)

Шаблон: `{template}.html.twig`

| Переменная | Тип | Описание |
|-----------|-----|----------|
| `items` | array | Массив записей из source_table |
| `nav_item` | object | Текущий пункт навигации |
| `current_page` | int | Текущая страница пагинации |
| `total_pages` | int | Всего страниц |
| `total_count` | int | Всего записей |
| `config` | array | Распарсенный page_config |
| `query_string` | string | GET-параметры для ссылок пагинации |
| `title` | string | title из navigation |

### `nav_item` — поля

| Поле | Описание |
|------|----------|
| `nav_item.title` | Название |
| `nav_item.url` | Адрес (`docs`, `blog`) |
| `nav_item.page_type` | Тип (`dynamic`) |
| `nav_item.source_table` | Имя таблицы |

---

## Отдельная запись (`dynamic`, single)

Шаблон: `{template}_single.html.twig`

| Переменная | Тип | Описание |
|-----------|-----|----------|
| `item` | array | Текущая запись (все поля таблицы) |
| `nav_item` | object | Пункт навигации |
| `prev_item` | array/null | Предыдущая запись |
| `next_item` | array/null | Следующая запись |
| `items` | array | **Все** записи таблицы (для сайдбара) |
| `title` | string | `item.title` |

---

## Статическая страница (`page`)

Шаблон: `page.html.twig`

| Переменная | Тип | Описание |
|-----------|-----|----------|
| `page` | array | Запись из таблицы pages |
| `title` | string | `page.title` |
| `is_home` | bool | `true` для главной |

---

## Лендинг (`landing`)

Шаблон: `landing.html.twig` или свой

| Переменная | Тип | Описание |
|-----------|-----|----------|
| `items` | array | Секции из source_table |
| `title` | string | Название страницы |

---

## AI-чат (`ai`)

Шаблон: `ai.html.twig`

| Переменная | Тип | Описание |
|-----------|-----|----------|
| `nav_item` | object | Пункт навигации |
| `title` | string | `nav_item.title` |

---

## Форма (`form`)

Шаблон: `form/{template}.html.twig`

Переменные зависят от FormRenderer и form_config. Подробнее: [Создание формы](forms-create.md).
