# Account Plugin v1.1.0 — Magic-link вход и смена пароля

Плагин Account (личный кабинет) получил восстановление доступа через magic-link и смену пароля.

## Что нового

- **Восстановление доступа:** `/account/forgot` — форма ввода email, генерация одноразового токена, отправка magic-link на почту
- **Magic-link вход:** `/account/reset?token=***` — проверка токена, мгновенная авторизация, удаление использованного токена
- **Смена пароля:** `/account/change-password` — форма смены пароля для авторизованных пользователей (текущий + новый + подтверждение)
- **Шаблон письма** `email_reset.html.twig` — редактируется через админку плагинов (вкладка «Шаблоны»)
- **Настройка TTL токена:** select 1 час / 24 часа / 7 дней в настройках плагина
- **Ссылки:** «Забыли пароль?» на странице входа, «Сменить пароль» в профиле

## Безопасность

- Rate-limit: один активный токен на email
- Токен одноразовый, удаляется после использования
- Универсальные сообщения (не раскрывают существование email)
- Токен — 64 символа (random_bytes), хеш пароля — bcrypt

## Технические детали

- Использует `Core\EmailSender` из ядра (API → SMTP → mail fallback)
- Токены хранятся в таблице `user_tokens` (тип `password_reset`)
- Без правок ядра — только плагин

---

# Account Plugin v1.1.0 — Magic-link Login & Password Change

The Account plugin now supports passwordless magic-link login and password change.

## What's New

- **Password recovery:** `/account/forgot` — email form, one-time token generation, magic-link email
- **Magic-link login:** `/account/reset?token=***` — token verification, instant login, token deletion
- **Password change:** `/account/change-password` — change password form for authenticated users (current + new + confirm)
- **Email template** `email_reset.html.twig` — editable via admin plugin panel (Templates tab)
- **Token TTL setting:** select 1 hour / 24 hours / 7 days in plugin settings
- **Links:** "Forgot password?" on login page, "Change password" in profile

## Security

- Rate limit: one active token per email
- Single-use token, deleted after use
- Generic messages (doesn't reveal email existence)
- Token: 64 chars (random_bytes), password hash: bcrypt

## Technical Details

- Uses `Core\EmailSender` from the core (API → SMTP → mail fallback)
- Tokens stored in `user_tokens` table (type `password_reset`)
- No core modifications — plugin only
