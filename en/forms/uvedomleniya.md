---
title: Notifications
slug: uvedomleniya
section: forms
order: 180
description: "Form notifications — admin alerts and auto-replies."
---

# 📬 Form Notifications

When a user submits a form, the system can notify the admin and send an auto-reply.

---

## Configuring in form_config

```json
{
  "notifications": {
    "admin_notify": true,
    "admin_emails": ["admin@site.ru"],
    "admin_subject": "New submission from the site",
    "auto_reply": true,
    "auto_reply_field": "email",
    "auto_reply_subject": "We received your submission",
    "auto_reply_template": "Thank you, {name}! We will contact you shortly."
  }
}
```

| Field | Description |
|-------|-------------|
| `admin_notify` | `true` — send notifications to admin |
| `admin_emails` | List of email addresses for notifications |
| `admin_subject` | Admin email subject |
| `auto_reply` | `true` — send auto-reply to user |
| `auto_reply_field` | Which field to use for the user's email |
| `auto_reply_subject` | Auto-reply subject |
| `auto_reply_template` | Auto-reply template, `{name}` is substituted |

---

## How It Works

1. User fills out and submits the form
2. Data is saved to `source_table`
3. If `admin_notify: true` — admin receives an email
4. If `auto_reply: true` — user receives an auto-reply

---

## Email Configuration

Email sending is configured in **Settings → System**:

- `email_driver` — `api` or `smtp`
- `email_api_key` — API key (for `smtp.bz` and similar)
- SMTP parameters — host, port, login, password

Without configured email, notifications will not be sent (but data will still be saved).