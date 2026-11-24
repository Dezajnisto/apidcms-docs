# Account v1.2.0 — Подтверждение email и фикс сохранения профиля

**Дата:** 24 ноября 2026
**Тип:** feature
**Теги:** plugin,account,auth,email,fix,security

## Новое

- **Подтверждение email.** Настройка `require_email_confirm`: регистрация создаёт
  пользователя в статусе `pending` и отправляет письмо со ссылкой
  `/account/confirm?token=...` (живёт 24 часа). После перехода аккаунт становится
  `active`, пользователь сразу авторизован, срабатывает хук `account.confirmed`.
  Значение хранится в `plugin_settings`, поэтому обновление плагина не меняет
  поведение сайта — по умолчанию выключено.
- **Хук `account.registered`** — когда подтверждение выключено и вход происходит сразу.
- **`?return=/путь`** для `/login` и `/register`: после входа пользователь попадает
  туда, откуда пришёл. Принимается только локальный путь (защита от открытого
  редиректа).
- Шаблоны `registered_check_email`, `confirm_error`, `email_confirm`.
- Общий `mailer()`: конфиг почты читается из `system_settings` в одном месте.

## Исправлено

- **Не сохранялся профиль и не менялся пароль.** `$db->update()` вызывался с
  устаревшей сигнатурой `($table, $data, $where, $params)`, а ядро ждёт
  `($table, $id, $data)`. Собирался пустой `SET`, запрос падал с
  `near "WHERE": syntax error` — и ошибка уходила только в лог, поэтому страница
  показывала успех. Исправлено в трёх местах: подтверждение почты, сохранение
  профиля, смена пароля.
- `login` для неподтверждённого аккаунта говорит «аккаунт не подтверждён», а не
  «неверный email или пароль».

## Манифест

- Из `account/plugin.json` убран ключ `path` с внутренним путём одной из установок.
- `enabled: false` → `true`: это базовый плагин, от него зависят другие.
- В `plugins.json` обновлены версия, описание и теги.

---

# Account v1.2.0 — Email Confirmation and Profile Save Fix

**Date:** November 24, 2026
**Type:** feature
**Tags:** plugin,account,auth,email,fix,security

## Added

- **Email confirmation.** Setting `require_email_confirm`: registration creates a
  `pending` user and sends a letter with a `/account/confirm?token=...` link (valid
  24 hours). Following it activates the account, logs the user in, and fires the
  `account.confirmed` hook. The value lives in `plugin_settings`, so updating the
  plugin never changes a site's behaviour — off by default.
- **`account.registered` hook** — when confirmation is off and login happens immediately.
- **`?return=/path`** for `/login` and `/register`: the user lands where they came
  from. Only local paths are accepted (open-redirect protection).
- Templates `registered_check_email`, `confirm_error`, `email_confirm`.
- Shared `mailer()`: mail config is read from `system_settings` in one place.

## Fixed

- **Profile was not saved and password was not changed.** `$db->update()` was called
  with the outdated signature `($table, $data, $where, $params)` while the core expects
  `($table, $id, $data)`. An empty `SET` was built and the query failed with
  `near "WHERE": syntax error` — logged only, so the page looked successful. Fixed in
  three places: email confirmation, profile save, password change.
- `login` for an unconfirmed account now says "account is not confirmed" instead of
  "wrong email or password".

## Manifest

- `account/plugin.json`: the `path` key with an internal install path was removed.
- `enabled: false` → `true`: this is a base plugin other plugins depend on.
- `plugins.json`: version, description and tags updated.
