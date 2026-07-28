---
title: Внешние источники (тип external)
slug: external-page-type
section: content
order: 110
---

## external — данные из внешних JSON API

Тип страницы `external` загружает данные из внешнего JSON-источника (API, GitHub, NocoDB) вместо локальной таблицы SQLite.

### Как работает

1. При заходе на страницу `FrontController` создаёт `ExternalPageLoader`
2. Загрузчик проверяет кеш (если `cache_ttl > 0`)
3. Если кеша нет или он устарел — HTTP-запрос к `source_url`
4. Ответ парсится как JSON, `json_path` извлекает целевой массив
5. Данные передаются в Twig-шаблон как переменная `items`

### Конфигурация (navigation.page_config, JSON)

```json
{
  "source_url": "https://api.example.com/data.json",
  "json_path": "results",
  "cache_ttl": 3600,
  "method": "GET",
  "headers": {
    "Authorization": "Bearer {{ token }}"
  },
  "template": "catalog"
}
```

| Поле | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `source_url` | string | — | **Обязательное.** URL внешнего JSON |
| `json_path` | string | `""` | Путь через точку: `"data.items"` → `$data['data']['items']`. Пусто = весь ответ |
| `cache_ttl` | int | `0` | Время жизни кеша в секундах. `0` = живой режим. `3600` = час |
| `method` | string | `"GET"` | HTTP-метод: GET, POST, PUT, PATCH |
| `headers` | object | `{}` | Дополнительные заголовки. Поддерживает `{{ token }}` и `{{ setting.KEY }}` |
| `template` | string | `"default"` | Имя Twig-шаблона (без `.html.twig`). `"default"` → `external.html.twig` |

### Режимы работы

| Режим | cache_ttl | Поведение |
|-------|-----------|-----------|
| Кешируемый | > 0 | Первый запрос → API → кеш в файл. Повторные → из кеша пока не истечёт TTL |
| Живой | 0 | Каждый запрос → API. Query-параметры URL пробрасываются в source_url |

### Кеш

- Файлы: `/admin/views/cache/external_{md5(url+path)}.json`
- Формат: `{"ts": 1690000000, "url": "...", "data": {...}}`
- Авто-очистка по TTL; ручная — через Админка → Очистить кэш → External

### Плейсхолдеры в заголовках

- `{{ token }}` → системная настройка `external_default_token`
- `{{ setting.KEY }}` → любая системная настройка по ключу

### Переменные в шаблоне

| Переменная | Тип | Описание |
|------------|-----|----------|
| `items` | array | Массив данных (из `json_path` или весь ответ) |
| `raw` | array | Полный JSON-ответ |
| `from_cache` | bool | `true` если из кеша |
| `page_config` | object | Конфиг страницы |
| `source_url` | string | URL источника |
| `nav_item` | object | Элемент навигации |

### Примеры

**Каталог плагинов из GitHub (кешируемый):**
```json
{
  "source_url": "https://raw.githubusercontent.com/Dezajnisto/apidcms-plugins/main/plugins.json",
  "json_path": "plugins",
  "cache_ttl": 3600,
  "template": "plugins"
}
```

**Чейнджлог из GitHub Releases (кешируемый):**
```json
{
  "source_url": "https://api.github.com/repos/Dezajnisto/apidcms/releases",
  "cache_ttl": 3600,
  "headers": {"Accept": "application/vnd.github.v3+json"},
  "template": "changelog"
}
```

**Каталог из NocoDB (живой):**
```json
{
  "source_url": "https://app.nocodb.com/api/v2/tables/mxzy0/records",
  "json_path": "list",
  "cache_ttl": 0,
  "headers": {"xc-token": "{{ setting.nocodb_api_key }}"},
  "template": "catalog"
}
```

### Обработка ошибок

Если API недоступен или возвращает ошибку — показывается `external_error.html.twig` с понятным сообщением. Сайт не падает.


---

### Пример: Buildin.ai (реальный кейс)

Интеграция с [Buildin.ai API v2](https://buildin.ai/developer-api/v2/getting-started/overview). Реальный пример: wearefun.ru — свадебный гид с полным Markdown-контентом.

[/svadba](https://wearefun.ru/svadba) — список-оглавление, [/svadba/{id}](https://wearefun.ru/svadba/2db6d106-ad2a-44f4-8e7e-11cfdc1c933d) — детальная страница.

**Список страниц через Search API (POST):**

```json
{
  "source_url": "https://api.buildin.ai/v2/search",
  "json_path": "results",
  "cache_ttl": 3600,
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer {{ token }}",
    "_body": "{\"filter\":{\"property\":\"object\",\"value\":\"page\"},\"page_size\":50}"
  },
  "template": "wedding"
}
```

**Детальная страница (single-item):**

При URL `/page/{id}` apidcms автоматически: (1) находит страницу в items по id, (2) генерирует content_url заменой пути /v2/search на /v2/pages/{id}/content/markdown, (3) загружает Markdown через fetchContent() с Authorization, (4) передает detail_content в шаблон.

**Шаблон детальной (wedding_single.html.twig):**

```twig
{% set page_title = item.properties.title.title[0].plain_text %}
{% set md = detail_content|json_decode %}

<div class="body-text">
    {{ md.markdown|markdown_to_html|raw }}
</div>
```

**Важно:** ответ content/markdown приходит как JSON `{"markdown": "..."}`, поэтому в шаблоне сначала применяем |json_decode, затем берем .markdown.

**Пошаговое руководство:** [как мы интегрировали Buildin.ai](https://apidcms.dezajno.ru/docs/external-page-type#пример-buildinai-реальный-кейс)
