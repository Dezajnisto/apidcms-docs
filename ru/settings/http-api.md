---
title: HTTP API
slug: http-api
section: settings
order: 350
description: "REST API /api/v1/: схема проекта, навигация, связи many-to-many, проверка записи, dry_run, журнал, статьи и документация; токены со скоупами"
---

# 🔌 HTTP API

Ядро отдаёт REST API по адресу `/api/v1/`. Он нужен, когда данные проекта должны
читаться или писаться не из браузера: мобильный клиент, скрипт, интеграция,
внешний сервис или AI-агент.

API работает поверх тех же таблиц, что и админка. Отдельной базы и отдельной
модели данных нет — схема таблицы и есть контракт.

> ⚠️ **Статус: preview.** Контракт `/api/v1` ещё меняется между версиями, а
> ответы не гарантируют полноты: запись проверяется не по всем правилам, часть
> путей запланирована и отвечает `501 not_implemented`. Все ответы несут заголовок
> `X-apidcms-api: v1-preview`. Прежде чем давать внешнему агенту право записи,
> прочитайте раздел «Чего пока нет».

---

## Авторизация

Запросы без токена отклоняются. Токен передаётся заголовком:

```
Authorization: Bearer <token>
```

Токен — это ключ доступа с именем и правами. Создаётся и отзывается в админке:
**Настройки → API**. Там же видно, когда токен использовался последний раз.

Права (скоуп) выбираются при создании:

| Скоуп | Что разрешено |
|---|---|
| `read` | только `GET`: схема, навигация, связи, таблицы, документация |
| `content` | `read` плюс запись в пользовательские таблицы и статьи |
| `full` | всё, включая шаблоны и служебные ручки |

Журнал `/api/v1/audit` — исключение из «только GET»: там IP клиентов и дифы
чужих записей, поэтому его читают только токены `content` и `full`.

Обращайтесь с токеном как с паролем админки — не в git, не в JS на фронтенде.
Сам токен показывается один раз при создании, в базе лежит только его хеш.
Скоуп, который применился к запросу, возвращается в заголовке `X-apidcms-scope`.

Старый ключ из системной настройки `api_key` продолжает работать (скоуп `full`),
но отозвать его из админки нельзя: заведите именованный токен и очистите
`api_key`, когда интеграция переедет.

Анонимно можно узнать только, что это за сайт: `GET /api/v1` и
`GET /api/v1/system/info`.

Системные таблицы (`system_settings`, `users`, `user_tokens`, `entity_relations`,
`visit_stats`, `table_meta`, `field_meta`, `plugin_*`) через API недоступны — тот
же чёрный список, что у AI-помощника.

---

## Начать с одного запроса

```bash
curl -H "Authorization: Bearer $TOKEN" https://example.com/api/v1/schema
```

`GET /api/v1/schema` отдаёт весь проект одним ответом: таблицы, колонки (тип,
обязательность, значение по умолчанию, допустимые значения), описания, внешние
ключи, связи many-to-many и граф страниц. Это дешевле и точнее, чем обойти
`/tables`, затем каждую `/schema` по очереди.

Если токена ещё нет — начните с `GET /api/v1`, он анонимный и перечисляет
возможности API вместе со статусом каждой.

### Описание проекта

| Метод | Путь | Что отдаёт |
|---|---|---|
| `GET` | `/api/v1` | `{cms, version, status: "preview", capabilities:[...]}` |
| `GET` | `/api/v1/system/info` | версия ядра, PHP, название сайта |
| `GET` | `/api/v1/schema` | все таблицы + связи + граф страниц |
| `GET` | `/api/v1/navigation` | граф страниц: `slug`, `url`, `type`, `template`, `source_table`, `filters` |
| `GET` | `/api/v1/relations` | связи many-to-many из `entity_relations` |
| `GET` | `/api/v1/relations/{table}` | то же для одной таблицы |
| `GET` | `/api/v1/relations/{table}/{id}` | связи одной записи: `target_ids` по каждому `relation_name` |
| `POST` | `/api/v1/relations` | связать запись: `{source_table, source_id, relation_name, target_ids[]}` |
| `DELETE` | `/api/v1/relations` | снять связи (пустой `target_ids` — снять все) |
| `GET` | `/api/v1/audit` | что записали через API: `limit`, `table`, `id` |

Колонка в `/schema` выглядит так:

```json
{
  "name": "price",
  "type": "INTEGER",
  "pk": false,
  "notnull": true,
  "default": null,
  "enum": null,
  "label": "Цена, ₽",
  "description": "Цена в рублях без НДС",
  "format": "integer",
  "example": "1500"
}
```

Типы и ограничения берутся из самой базы (`PRAGMA table_info`, `CHECK`-констрейнты,
`PRAGMA foreign_key_list`). Подписи, назначение и допустимые значения — из
метаданных проекта (см. ниже). Если поле не описано, в ответе `null`: это значит
«не описано», а не «не важно» — значение не выдумывается.

---

## Семантика таблиц

Форму таблицы видно из базы, смысл — нет: из `INTEGER` не следует, что это рубли,
а из `TEXT` — что там `active`, а не `published`.

Поэтому у таблиц и колонок есть описание, которое правится в админке:

> Таблицы → **Структура** → кнопка **«Семантика»**

Для таблицы: название, назначение, подробное описание. Для колонки: подпись,
пояснение, формат (`integer`, `money`, `date`, `url`, …), пример и список
допустимых значений. Хранится в таблицах `table_meta` и `field_meta` и отдаётся
через `/api/v1/schema` и `/api/v1/tables/{table}/schema` — это единственная часть
описания, которую нельзя восстановить из кода.

---

## Карта страниц: navigation.page_config

Связь «таблица → URL → шаблон» живёт в колонке `navigation.page_config` (JSON):

```json
{
  "source_table": "posts",
  "template": "blog",
  "items_per_page": 10,
  "filters": {"category": "news"},
  "get_filters": {"tag": "tags"},
  "sort": {"field": "created_at", "order": "DESC"}
}
```

Обычно она заполняется в админке при создании страницы, но именно это и делает
сайт сайтом: без `page_config` данные не превращаются в страницы. `GET
/api/v1/navigation` отдаёт её уже разобранной — без JSON-строк и без догадок:

```json
{
  "slug": "blog",
  "url": "/blog",
  "title": "Блог",
  "type": "dynamic",
  "template": "blog",
  "source_table": "posts",
  "filters": {"category": "news"},
  "items_per_page": 10,
  "page_config_error": null
}
```

Если JSON в `page_config` битый, страница всё равно попадает в ответ, а в
`page_config_error` будет причина.

---

## Связи many-to-many

Таблица `entity_relations` — внутренняя механика: через `/tables` её не видно, а
читать связи надо через `/relations`. Запись идёт тем же путём и одним запросом:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"source_table":"catalog","source_id":42,"relation_name":"categories","target_ids":[3,5]}' \
  https://example.com/api/v1/relations
```

`relation_name` — это имя связи, а не таблицы: соответствие «связь → таблица»
берётся из `navigation.page_config.relations`, а если проект его не объявлял —
из совпадения имени с таблицей. Проверяется всё, что можно проверить до записи:
существование таблицы, записи, связи и каждой цели. Уже существующая связь не
дублируется, она возвращается в `changes.already`. `?dry_run=1` показывает
`before` и `after`, `Idempotency-Key` защищает от повторов, `audit` пишет диф.

---

## Таблицы

| Метод | Путь | Что делает |
|---|---|---|
| `GET` | `/api/v1/tables` | список таблиц проекта |
| `GET` | `/api/v1/tables/{таблица}` | записи: `limit`, `offset`, `sort`, `order`, `search` |
| `GET` | `/api/v1/tables/{таблица}/{id}` | одна запись по `id` |
| `POST` | `/api/v1/tables/{таблица}` | создать запись (JSON в теле) |
| `PUT` | `/api/v1/tables/{таблица}/{id}` | обновить запись |
| `DELETE` | `/api/v1/tables/{таблица}/{id}` | удалить запись |
| `GET` | `/api/v1/tables/{таблица}/schema` | схема таблицы: колонки, связи, метаданные |
| `GET` | `/api/v1/tables/{таблица}/json-schema` | JSON Schema таблицы: правила записи для валидации на своей стороне |

Пример — забрать последние записи каталога:

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/tables/catalog?limit=20&offset=0&sort=id&order=desc"
```

---

## Запись: что проверяется

Запись в таблицу проверяется до обращения к базе. Правила берутся из самой
таблицы (`PRAGMA`) и из метаданных, поэтому «правила — это схема»:

| Правило | Откуда берётся |
|---|---|
| `type` | affinity колонки: `INTEGER`, `REAL`, `TEXT`, `BLOB`; `"1500"` для INTEGER примется, `"1500 руб"` — нет |
| `required` / `notnull` | `NOT NULL` без значения по умолчанию |
| `enum` | `field_meta.enum`, иначе `CHECK (col IN (...))` из DDL |
| `foreign_key` | `PRAGMA foreign_key_list`: связанная запись должна существовать |
| `format` | `field_meta.format`: `integer`, `money`, `number`, `boolean`, `date`, `datetime`, `url`, `email`, `slug` |
| `max_length` | `VARCHAR(n)` |
| `unknown_column` | поля, которых в таблице нет |

Отказ — `422` с машинночитаемым списком. Ничего не записано, ошибка называет
поле, правило, что пришло и что ожидалось:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"title":"Кроссовки","price":"1500 руб"}' \
  https://example.com/api/v1/tables/catalog
```

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Запись отклонена: проверьте поля",
    "table": "catalog",
    "action": "insert",
    "fields": [
      {"column": "price", "rule": "type", "got": "1500 руб", "expected": "integer (INTEGER)"}
    ],
    "hint": "Rules come from the table schema: GET /api/v1/tables/catalog/json-schema"
  }
}
```

Ограничения, которые база всё-таки отклонила сама, тоже приходят кодом, а не
текстом исключения: `conflict` (`409`, нарушен `UNIQUE`), `foreign_key_violation`
(`409`), `notnull_violation` (`422`), `database_error` (`422`).

---

## «Что произойдёт?»: `dry_run`

Любая запись и удаление принимают `?dry_run=1`: проверка выполняется, изменения
не сохраняются, в ответе — что изменилось бы и какие страницы это показывают.

```bash
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/tables/catalog/42?dry_run=1"
```

```json
{
  "dry_run": true,
  "table": "catalog",
  "id": 42,
  "action": "delete",
  "would_change": true,
  "validation": [],
  "row": {"id": 42, "title": "Кроссовки"},
  "affected_urls": [
    {"slug": "catalog", "url": "/catalog", "title": "Каталог", "template": "catalog", "filters": {}}
  ]
}
```

`affected_urls` — ответ на «я записал, а на сайте ничего не изменилось»: это
страницы, которые собираются из этой таблицы (по `navigation.page_config`).

С `dry_run=1` неверные данные не дают `422`: в ответе `200` с
`"would_change": false` и списком `validation` — так один запрос отвечает и «можно
ли», и «что изменится».

---

## Повторы: `Idempotency-Key`

Агенты ретраят — на таймауте, на `502` от прокси, после падения клиента. Чтобы
ретрай не создавал вторую запись, передайте ключ:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -H "Idempotency-Key: 6f1c2a90-4b09-4b9a-a3d4-0c1f7f6b1e42" \
  -d '{"title":"Кроссовки"}' \
  https://example.com/api/v1/tables/catalog
```

Повтор с тем же ключом вернёт первый ответ (`201` и тот же `id`) и заголовок
`Idempotent-Replay: true`. Помним 24 часа. Хранятся только успешные ответы:
неудачная попытка остаётся повторяемой.

---

## Журнал: что сделал агент

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/audit?limit=20&table=catalog"
```

Каждая запись через API попадает в `system_audit`: метод, путь, таблица,
`row_id`, IP и диф (`before`, `after`, `changed`). Записи старше 90 дней
удаляются. Фильтры: `limit` (до 500), `table`, `id` записи.

---

## Статьи

| Метод | Путь | Что делает |
|---|---|---|
| `GET` | `/api/v1/articles` | список статей |
| `GET` | `/api/v1/articles/{slug}` | одна статья |
| `POST` | `/api/v1/articles` | создать статью |
| `PUT` | `/api/v1/articles/{slug}` | обновить статью |
| `DELETE` | `/api/v1/articles/{slug}` | удалить статью |

---

## Документация через API

Документация CMS отдаётся из локального кэша — без обращения к GitHub на каждый
запрос:

| Метод | Путь | Что отдаёт |
|---|---|---|
| `GET` | `/api/v1/docs` | оглавление: `slug`, заголовок, раздел, описание |
| `GET` | `/api/v1/docs?q=слова` | поиск по заголовкам, описаниям и текстам статей |
| `GET` | `/api/v1/docs/{slug}?lang=ru\|en` | статья в Markdown |
| `POST` | `/api/v1/docs/refresh` | обновить кэш из GitHub (`?force=1` — всё заново) |

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/docs/http-api?lang=ru"
```

Кэш лежит в `storage/cache/docs/`, чтение никогда не ходит в сеть, а обновление —
отдельный вызов: без сети остаётся прежняя копия, а `status.stale` говорит,
насколько она старая. Те же статьи отдаются как ресурсы MCP:
`apidcms://docs/{slug}`.

---

## Формат ответов

Успешный ответ — JSON-объект или массив объектов. Ошибка — всегда одинаково:

```json
{
  "error": {
    "code": "not_found",
    "message": "Table 'nope' does not exist"
  }
}
```

Коды: `unauthorized` (нет или неверный токен), `api_disabled` (API не настроен:
нет ни одного токена и не задан `api_key`), `insufficient_scope` (токену не
хватает прав — например, `read` пробует запись), `not_found` (таблица, запись или
статья не найдены), `bad_request` (некорректный путь или тело),
`validation_failed` (поле не прошло проверку — см. список `fields`),
`conflict` / `foreign_key_violation` / `notnull_violation` / `database_error`
(база отклонила запись), `forbidden` (системная таблица),
`not_implemented` (путь есть в планах, но ещё не реализован — отвечает `501`, а
не `404`, чтобы вы не искали опечатку), `internal_error` (ошибка кода).

---

## Чего пока нет

Планируется и помечено в `capabilities` как `planned`:

- **шаблоны** (`/api/v1/templates`) — без них задачу «добавь раздел» нельзя
  довести до конца.

MCP уже работает: `POST /mcp` отдаёт те же данные и действия как инструменты
агента — [MCP-сервер](../settings/mcp.md).

---

## Внешний контент: кэш

Если страница собирается из внешнего источника (тип страницы `external`), ядро
кэширует ответ по URL на час и при недоступности источника отдаёт **просроченный**
кэш. Поэтому недоступный внешний API не превращается в 15 секунд ожидания и не
ломает страницу — посетитель видит данные часовой давности.

Кэш лежит в `storage/cache/`, очищается кнопкой «Очистить кэш» в админке.
