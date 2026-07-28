---
title: Встраивание формы
slug: vstraivanie-formy
section: forms
order: 200
description: "Встраивание формы"
---

# 🔌 Встраивание формы в любую страницу

Функция `render_form()` позволяет вставить форму куда угодно — не только на отдельную страницу.

---

## Базовый вызов

```twig
{{ render_form('contacts') }}
```

Где `contacts` — имя таблицы (из колонки `name` в таблице `forms`).

---

## С параметрами

```twig
{{ render_form('orders', {
    submit_text: 'Заказать',
    form_class: 'order-form',
    consent_enabled: true
}) }}
```

| Параметр | По умолчанию | Описание |
|----------|-------------|----------|
| `submit_text` | `Отправить` | Текст на кнопке |
| `form_class` | — | CSS-класс на `<form>` |
| `field_class` | — | CSS-класс на полях |
| `consent_enabled` | `false` | Показать чекбокс ПД |
| `template` | из form_config | Переопределить шаблон |

---

## Пример: форма в футере

```twig
{# в base.html.twig #}
<footer>
  <div class="footer-form">
    <h3>Остались вопросы?</h3>
    {{ render_form('contacts', {submit_text: 'Спросить'}) }}
  </div>
</footer>
```

---

## Отличие от страницы-формы

| | `page_type = 'form'` | `render_form()` |
|---|---|---|
| Отдельный URL | ✅ `/contact` | ❌ встраивается |
| Настройки | `form_config` в JSON | параметры в вызове |
| Валидация | ✅ | ✅ |
| Уведомления | ✅ | ✅ |

Оба способа используют одну и ту же таблицу и `FormRenderer`.
