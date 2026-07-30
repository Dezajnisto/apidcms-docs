# apidcms — Documentation / Документация

[Русская версия ниже](#русский)

---

Centralized documentation for [apidcms](https://github.com/Dezajnisto/apidcms) — a lightweight PHP + SQLite CMS.

## Structure

```
apidcms-docs/
├── ru/               # Russian docs
├── en/               # English docs
├── changelog/        # Version history
├── images/           # Screenshots and images
└── manifest.json     # Route map
```

## How to contribute

1. Fork the repository
2. Make changes to .md files
3. Open a Pull Request

## Format

Each .md file contains YAML front-matter:

```yaml
---
title: Page title
slug: url-slug
section: quickstart
order: 10
description: Brief description
---
```

## Usage

Documentation is used on [apidcms.dezajno.ru](https://apidcms.dezajno.ru/docs) and in the admin panel as contextual help.

---

## Русский {#русский}

Централизованная документация для [apidcms](https://github.com/Dezajnisto/apidcms) — лёгкой CMS на PHP + SQLite.

### Структура

```
apidcms-docs/
├── ru/               # Русская документация
├── en/               # Английская документация
├── changelog/        # История версий
├── images/           # Скриншоты и изображения
└── manifest.json     # Карта роутов
```

### Как помочь

1. Форкните репозиторий
2. Внесите изменения в .md файлы
3. Откройте Pull Request

### Формат

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

### Использование

Документация используется на [apidcms.dezajno.ru](https://apidcms.dezajno.ru/docs) и в панели администрирования как контекстная справка.
