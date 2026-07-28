# apidcms — Документация

Централизованная документация для [apidcms](https://github.com/Dezajnisto/apidcms) — лёгкой CMS на PHP + SQLite.

## Структура

```
apidcms-docs/
├── quickstart/       # Быстрый старт
├── content/          # Контент и данные
├── forms/            # Формы и файлы
├── templates/        # Шаблоны и Twig
├── settings/         # Настройки
├── ai/               # AI Ассистент
├── plugins-dev/      # Разработка плагинов
├── changelog/        # История версий
└── manifest.json     # Карта роутов
```

## Как контрибьютить

1. Форкните репозиторий
2. Внесите изменения в .md файлы
3. Откройте Pull Request

## Формат

Каждый .md файл содержит YAML front-matter:

```yaml
---
title: Название страницы
slug: url-slug
section: quickstart
order: 10
description: Краткое описание
---
```

## Использование

Документация используется на [apidcms.dezajno.ru](https://apidcms.dezajno.ru/docs) и в панели администрирования как контекстная справка.
