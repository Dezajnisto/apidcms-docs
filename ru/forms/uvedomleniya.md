---
title: Уведомления
slug: uvedomleniya
section: forms
order: 180
description: "Уведомления"
---

# 📬 Уведомления форм

Когда пользователь отправляет форму, система может уведомить админа и отправить автоответ.

---

## Настройка в form_config

```json
{
  "notifications": {
    "admin_notify": true,
    "admin_emails": ["admin@site.ru"],
    "admin_subject": "Новая заявка с сайта",
    "auto_reply": true,
    "auto_reply_field": "email",
    "auto_reply_subject": "Мы получили вашу заявку",
    "auto_reply_template": "Спасибо, {name}! Мы свяжемся с вами в ближайшее время."
  }
}
```

| Поле | Описание |
|------|----------|
| `admin_notify` | `true` — слать уведомления админу |
| `admin_emails` | Список email для уведомлений |
| `admin_subject` | Тема письма админу |
| `auto_reply` | `true` — слать автоответ пользователю |
| `auto_reply_field` | Из какого поля брать email пользователя |
| `auto_reply_subject` | Тема автоответа |
| `auto_reply_template` | Шаблон автоответа, `{name}` — подстановка |

---

## Как работает

1. Пользователь заполняет и отправляет форму
2. Данные сохраняются в `source_table`
3. Если `admin_notify: true` — админ получает email
4. Если `auto_reply: true` — пользователь получает автоответ

---

## Настройка email

Email-отправка настраивается в разделе **Настройки → Системные**:

- `email_driver` — `api` или `smtp`
- `email_api_key` — ключ API (для `smtp.bz` и подобных)
- Параметры SMTP — хост, порт, логин, пароль

Без настроенного email уведомления не будут отправляться (но данные сохранятся).
