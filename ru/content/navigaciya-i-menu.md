---
title: Навигация и меню
slug: navigaciya-i-menu
section: content
order: 90
description: "Навигация и меню"
---

# 🧭 Навигация и меню

Таблица `navigation` — это сердце маршрутизации apidcms. Каждая запись — одна страница сайта.

---

## Поля таблицы

| Поле | Тип | Описание |
|------|-----|----------|
| `title` | TEXT | Название страницы (в меню и `<title>`) |
| `url` | TEXT | Адрес: `docs`, `/about`, `blog` |
| `page_type` | TEXT | Тип страницы: `page`, `dynamic`, `form`, `ai`, `landing`, `category`, `raw` |
| `source_table` | TEXT | Имя таблицы с данными (для `dynamic`, `form`, `landing`) |
| `page_config` | TEXT | JSON с настройками страницы |
| `template` | TEXT | Имя шаблона: `default` или своё (напр. `docs`) |
| `location` | TEXT | `header` / `footer` / `none` |
| `menu_order` | INTEGER | Порядок в меню |
| `parent_id` | INTEGER | ID родительского пункта (для вложенных меню) |
| `status` | TEXT | `active` / `inactive` |

---

## Типы страниц

| `page_type` | Назначение | `source_table` | Пример URL |
|-------------|-----------|----------------|------------|
| `page` | Статическая страница | — | `/about` |
| `dynamic` | Динамический список + записи по slug | `docs` | `/docs`, `/docs/install` |
| `form` | Форма обратной связи | `contacts` | `/contact` |
| `ai` | AI-чат | — | `/chat` |
| `landing` | Посадочная из секций | `sections` | `/` |
| `category` | Фильтрованный список | `posts` | `/blog/category/news` |
| `raw` | Файл как есть (`robots.txt`, `sitemap.xml`) | — | `/robots.txt` |

---

## `raw` — отдать файл как есть

Тип `raw` рендерит шаблон темы **без layout сайта** и отдаёт его с нужным
`Content-Type`. Для машинных файлов: `robots.txt`, `sitemap.xml`, файлов
подтверждения прав на сайт в поисковых системах.

| Поле | Значение |
|------|----------|
| `url` | `robots.txt` — адрес без слеша |
| `page_type` | `raw` |
| `template` | имя шаблона: `robots` → `robots.html.twig` |
| `location` | `none` — чтобы файл не попал в меню |
| `status` | `active` |
| `page_config` | `{"content_type": "text/plain; charset=utf-8"}` |

Шаблон лежит в теме: `themes/<тема>/front/robots.html.twig`. Если файла в теме
нет, используется шаблон ядра `core/views/front/robots.html.twig` — он отдаёт
`User-agent: *` и `Disallow:`.

`content_type` по умолчанию — `text/plain; charset=utf-8`. Для `sitemap.xml`:

```json
{"content_type": "application/xml; charset=utf-8"}
```

а `template` — `sitemap.xml.html.twig`. Можно указать имя и с расширением
(`sitemap.xml.twig`) — тогда `.html.twig` не добавляется.

В шаблоне доступны `nav_item`, `_GET`, `site_lang`, `site_title`,
`site_description`, `current_path` и обычные функции `get_setting()` и `url()`.

Экранирование Twig применяется только к выводу переменных (`{{ }}`), сам текст
шаблона выводится как есть. Если подставляете в `sitemap.xml` значение из
настроек, добавьте `|raw` — иначе `<`, `>` и `&` превратятся в HTML-сущности.

Если `template` не задан или файла нет — страница отдаёт 404.

⚠️ `page_type` — **не свободное поле**. Значение, которого нет в таблице выше,
обрабатывается как динамический список (`dynamic`) и без `source_table` отдаёт
404. Поэтому для robots.txt и sitemap.xml нужен именно `raw`.

⚠️ Если в `www/` лежит настоящий файл `robots.txt`, nginx отдаст его сам и до
CMS запрос не дойдёт. Управляемый из админки файл должен существовать только в
виде записи `navigation`.

---

## Как создать новую страницу

1. Зайдите в админку → Таблицы → **navigation**
2. Нажмите **«Добавить запись»**
3. Заполните поля:
   - `title` — как будет называться
   - `url` — адрес (без слеша в начале: `docs`)
   - `page_type` — выберите тип
   - `source_table` — таблица с данными
   - `template` — `default` или имя своего шаблона
   - `page_config` — JSON с настройками (опционально)
4. `status` — `active`
5. Сохраните

Готово. Страница сразу доступна по адресу.

---

## `page_config` — JSON настроек

Подробный справочник: [page_config](page_config.md)

```json
{
  "source_table": "docs",
  "template": "docs",
  "sort": {"field": "sort_order", "order": "ASC"},
  "items_per_page": 20
}
```

---

## `template` — как CMS выбирает шаблон

| Значение | Шаблон списка | Шаблон записи |
|----------|--------------|---------------|
| `default` | `blog/list.html.twig` | `blog/single.html.twig` |
| `docs` | `docs.html.twig` | `docs_single.html.twig` |
| `my/page` | `my/page.html.twig` | `my/page_single.html.twig` |

Если в `page_config` указан и `template` — приоритет у `page_config`.
