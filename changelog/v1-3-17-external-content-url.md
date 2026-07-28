---
version: "v1.3.17"
release_date: "2026-07-24"
title: "Авто-генерация content_url и авторизация в fetchContent для external-страниц"
slug: "v1-3-17-external-content-url"
badge_type: "release"
tags: ""
---

Single-item external страницы с авто-загрузкой Markdown. fetchContent() с авторизацией.

## ✨ Новое

Авто-генерация content_url из item.id. Single-item external с Markdown.

## 🚀 Улучшено

handleExternalPage создает content_url. fetchContent() передает Authorization.

## 🐛 Исправлено

fetchContent() включает Authorization header (исправлен 401 для защищенных API).
