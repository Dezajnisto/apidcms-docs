---
version: "v1.3.15"
release_date: "23 Июля 2026"
title: "Мульти-колоночная сортировка в dynamic-списках через sort.then"
slug: "v1-3-15-multi-column-sort"
badge_type: "enhancement"
tags: "sort, dynamic, page-config, enhancement"
---

Мульти-колоночная сортировка dynamic-списков: ключ sort.then в page_config позволяет задать цепочку ORDER BY. Обратная совместимость.

В page_config dynamic-страниц добавлена поддержка вторичной сортировки: ключ `sort.then` позволяет указать цепочку полей для ORDER BY.

## ✨ Новое

- Ключ `sort.then` в page_config — цепочка вторичных полей сортировки
- Обратная совместимость: старые конфиги без `then` работают без изменений
- Поля теперь оборачиваются в двойные кавычки (защита от SQL-ключевых слов)

## 🚀 Улучшено

- `FrontController::showDynamicList` — сборка ORDER BY из цепочки полей
- Валидация имён полей через preg_replace (только a-zA-Z_)

## 🐛 Исправлено

- Нет в этом релизе

```
{
  "source_table": "docs",
  "template": "docs",
  "sort": {
    "field": "section_title",
    "order": "ASC",
    "then": [{"field": "sort_order", "order": "ASC"}]
  },
  "items_per_page": 50
}
```
