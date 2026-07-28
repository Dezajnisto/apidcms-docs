---
title: Системные настройки — справочник
slug: sistemnye-nastrojki-spravochnik
section: settings
order: 320
description: "Системные настройки — справочник"
---

# ⚙️ Системные настройки — справочник

Все настройки хранятся в таблице `system_settings`. Управляются через админку: Настройки → три вкладки.

---

## Основные настройки сайта

| Ключ | По умолчанию | Описание |
|------|-------------|----------|
| `site_title` | `Мой сайт` | Название сайта |
| `site_description` | `Описание сайта` | Meta description |
| `site_email` | — | Контактный email |
| `site_favicon` | — | Путь к favicon |
| `posts_per_page` | `10` | Записей на странице блога |
| `maintenance_mode` | `0` | `1` — сайт в режиме обслуживания |

---

## AI (DeepSeek)

| Ключ | Описание |
|------|----------|
| `ai_api_key` | API ключ DeepSeek |
| `ai_model` | Модель (по умолч. `deepseek-chat`) |
| `ai_prompt_template` | Системный промт для генерации шаблонов |
| `ai_prompt_table` | Промт для создания структуры таблиц |
| `ai_prompt_content` | Промт для генерации контента |
| `ai_prompt_fill_form` | Промт для заполнения форм |
| `ai_prompt_assistant` | Промт для универсального ассистента |

---

## AI Фронтенд (ai-страницы)

| Ключ | По умолчанию | Описание |
|------|-------------|----------|
| `ai_frontend_use_system` | `1` | Использовать системный ключ и модель |
| `ai_frontend_api_key` | — | Отдельный ключ для фронтенда |
| `ai_frontend_model` | — | Отдельная модель для фронтенда |
| `ai_frontend_prompt` | (см. ниже) | Системный промт для AI-чата |
| `ai_frontend_personality` | — | Имя/роль ассистента |
| `ai_public_tables` | — | JSON-массив: таблицы, доступные AI |
| `ai_sample_limit` | `50` | Макс. строк из таблицы для контекста |

---

## Статистика

| Ключ | По умолчанию | Описание |
|------|-------------|----------|
| `stats_enabled` | `0` | `1` — сбор включён |
| `stats_retention_days` | `90` | Сколько дней хранить данные |

---

## Email

| Ключ | По умолчанию | Описание |
|------|-------------|----------|
| `email_driver` | `api` | `api` или `smtp` |
| `email_api_provider` | — | Провайдер (напр. `smtp.bz`) |
| `email_api_key` | — | API ключ |
| `email_api_endpoint` | — | URL эндпоинта |
| `email_smtp_host` | — | SMTP сервер |
| `email_smtp_port` | `587` | Порт |
| `email_smtp_username` | — | Логин |
| `email_smtp_password` | — | Пароль |
| `email_smtp_encryption` | `tls` | `tls` / `ssl` |
| `email_from_email` | — | Email отправителя |
| `email_from_name` | `APIDCMS` | Имя отправителя |
