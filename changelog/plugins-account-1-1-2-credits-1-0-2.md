# Account v1.1.2 + Credits v1.0.2 — Исправление потока подписки

**Дата:** 30 июля 2026
**Тип:** hotfix
**Теги:** plugin,account,credits,subscription,payments,fix

## Account v1.1.2

- **Профиль:** AccountController теперь читает GET-параметры (`error`, `subscribed`, `pending`) от платёжного потока
- **Сообщения:** понятные тексты ошибок — «Не удалось создать платёж», «Платёж отменён», «Платёжный шлюз не настроен»
- **Успех:** сообщения «Подписка успешно оформлена!» и «Платёж ожидает подтверждения»

## Credits v1.0.2

- **Возврат из оплаты:** обработчик `payments.return` больше не перехватывает возвраты, если покупка кредитов отключена (приоритет 15, подписочный обработчик на 10 получает первый шанс)
- **Опечатка:** `subscription_plans` → `plugin_subscription_plans` в бонусном обработчике (начисление кредитов за подписку)

---

# Account v1.1.2 + Credits v1.0.2 — Subscription Flow Fixes

**Date:** July 30, 2026
**Type:** hotfix
**Tags:** plugin,account,credits,subscription,payments,fix

## Account v1.1.2

- **Profile:** AccountController now reads GET parameters (`error`, `subscribed`, `pending`) from payment flow
- **Errors:** human-readable messages — "Payment creation failed", "Payment canceled", "Payment gateway not configured"
- **Success:** "Subscription activated!" and "Payment pending confirmation" messages

## Credits v1.0.2

- **Payment return:** `payments.return` handler no longer intercepts returns when credits purchase is disabled (priority 15, subscription handler at 10 gets first chance)
- **Typo fix:** `subscription_plans` → `plugin_subscription_plans` in bonus credits handler
