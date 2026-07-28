---
version: "v1.3.14"
release_date: "23 Июля 2026"
title: "Тип страницы external — данные из внешних JSON API/URL"
slug: "v1-3-14-external-page-type"
badge_type: "feature"
tags: "external, api, json, cache"
---

Новый тип страницы external загружает данные напрямую из внешних JSON-источников: GitHub, NocoDB, любое API. С кешированием и живым режимом.

Данные для страниц теперь можно брать не только из SQLite, но и напрямую из внешних JSON-источников.

## ✨ Новое

- Тип страницы `external` — загрузка данных из внешнего JSON API/URL
- Класс `ExternalPageLoader` с HTTP-запросами и файловым кешем
- Dot-notation `json_path` для извлечения нужного массива из ответа
- Два режима: кешируемый (cache_ttl > 0) и живой (cache_ttl = 0)
- Плейсхолдеры `{{ token }}` и `{{ setting.KEY }}` в заголовках
- Проброс query параметров в source_url для живого режима
- Очистка external кеша через админку (CacheController)
- Два дефолтных шаблона: `external.html.twig` и `external_error.html.twig`

## 🚀 Улучшено

- `init_system_tables.php` — добавлена настройка `external_default_token`

## 🐛 Исправлено

- Нет в этом релизе
