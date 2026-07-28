---
version: "v1.3.12"
release_date: "2026-07-21"
title: "Many-to-many связи и множественный выбор категорий"
slug: "v1-3-12-many-to-many-relations"
badge_type: "feature"
tags: ""
---

Множественный выбор категорий через pivot-таблицу entity_relations: multi-select с деревом и поиском, фильтрация, get_pivot()

## ✨ Новое

<ul><li>Twig get_pivot() для M:M-связей</li><li>Multi-select чекбоксы с деревом и поиском</li><li>Pivot-aware фильтрация и счётчики категорий</li></ul>

## 🚀 Улучшено

<ul><li>Универсальная очистка pivot при удалении</li><li>M:M поля как виртуальные (не колонки БД)</li><li>Счётчики категорий через entity_relations JOIN</li></ul>
