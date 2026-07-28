---
version: "v1.3.23"
release_date: "2026-07-28"
title: "Документация из GitHub и strip_frontmatter в ExternalPageLoader"
slug: "v1-3-23-strip-frontmatter"
badge_type: "feature"
tags: ""
---

Страницы документации переключены на external-тип: контент из GitHub-репозитория apidcms-docs. Новая опция strip_frontmatter убирает YAML-шапку из сырого markdown.

## ✨ Новое

Документация из GitHub:
- Страница /docs переключена с dynamic на external
- Контент из репозитория apidcms-docs (manifest.json + md-файлы)
- Кэширование 1 час, обновление через сброс кэша админки

ExternalPageLoader: новая опция strip_frontmatter
- Убирает YAML-шапку (---...---) из загружаемого markdown
- Настраивается в page_config: "strip_frontmatter": true

## 🚀 Улучшено

- 36 статей документации в 7 разделах на GitHub
- manifest.json с метаданными: content_url, section, order, description
- Единый источник документации для сайта, админки и AI
