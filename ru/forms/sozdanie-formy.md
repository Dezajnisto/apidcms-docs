---
title: Создание формы
slug: sozdanie-formy
section: forms
order: 170
description: "Пошаговое руководство: создание таблицы, настройка form_config, поля, уведомления, встраивание через Twig."
---

# Создание формы

apidcms позволяет создавать формы обратной связи, заявок, заказов и любых других — без программирования. Всё настраивается через админ-панель.

---

## Как это работает

Форма состоит из трёх частей:

1. **Таблица в БД** — куда сохраняются заявки
2. **Запись в реестре форм** (`forms`) — конфигурация: поля, уведомления, дизайн
3. **Шаблон** — как форма выглядит (Twig)

После настройки форму можно встроить в любую страницу одной строкой в Twig-шаблоне.

---

## Шаг 1. Создайте таблицу для хранения заявок

1. В админке нажмите **«Создать таблицу»**
2. Укажите название (например, `form_callback`)
3. Добавьте поля под ваши данные:
   - `name` → `TEXT` — имя
   - `phone` → `TEXT` — телефон
   - `message` → `TEXT` — сообщение
4. **Обязательно добавьте служебные поля:**
   - `pd_consent` → `INTEGER`, по умолчанию `0` — для галочки согласия на ПД
   - `read_status` → `TEXT`, по умолчанию `unread` — для отметки «прочитано/не прочитано»
   - `form_name` → `TEXT` — заполняется автоматически, нужно для фильтрации
5. Нажмите **«Создать»**

---

## Шаг 2. Зарегистрируйте форму

1. В боковом меню админки нажмите **«Формы»**
2. Нажмите **«Добавить форму»**
3. Заполните поля:

### Основные

| Поле | Описание | Пример |
|------|----------|--------|
| **Имя формы (name)** | Техническое имя, латиницей | `callback` |
| **Отображаемое имя** | Как форма называется в админке | «Обратный звонок» |
| **Таблица для данных** | Имя таблицы из Шага 1 | `form_callback` |
| **Шаблон** | Файл `.html.twig` в `form/` | `default` (стандартный) или свой |

### Поля формы (fields) — JSON

Описывает, какие поля будут в форме. Формат:

```json
{
  "name": {
    "label": "Ваше имя",
    "type": "text",
    "placeholder": "Иван",
    "required": true
  },
  "phone": {
    "label": "Номер телефона",
    "type": "tel",
    "placeholder": "+7 (999) 123-45-67",
    "required": true
  },
  "message": {
    "label": "Комментарий",
    "type": "textarea",
    "placeholder": "Опишите ваш вопрос",
    "required": false
  }
}
```

**Доступные типы полей:** `text`, `email`, `tel`, `textarea`, `select`, `checkbox`, `number`, `date`.

**Важно:** имена полей в JSON должны совпадать с названиями колонок в таблице. Поля `id`, `created_at`, `updated_at`, `read_status` — системные, их добавлять в fields не нужно.

### Уведомления (notifications) — JSON

```json
{
  "admin_notify": true,
  "admin_emails": ["your@email.ru"],
  "admin_subject": "Новая заявка с сайта",
  "auto_reply": false
}
```

| Поле | Описание |
|------|----------|
| `admin_notify` | Отправлять уведомление админу (true/false) |
| `admin_emails` | Список email для уведомлений |
| `admin_subject` | Тема письма |
| `auto_reply` | Отправлять автоответ пользователю |
| `auto_reply_field` | Из какого поля брать email пользователя |
| `auto_reply_subject` | Тема автоответа |

### Дизайн (design) — JSON

```json
{
  "submit_text": "Отправить заявку",
  "submit_class": "btn btn--primary",
  "form_class": ""
}
```

| Поле | Описание |
|------|----------|
| `submit_text` | Текст на кнопке отправки |
| `submit_class` | CSS-класс для кнопки |
| `form_class` | Дополнительный CSS-класс для `<form>` |
| `field_class` | CSS-класс для полей ввода |
| `label_class` | CSS-класс для подписей к полям |

---

## Шаг 3. Встройте форму на страницу

В любом Twig-шаблоне используйте функцию `render_form()`:

```twig
{{ render_form('callback', {template: 'default'}) }}
```

Где `callback` — это значение поля **name** из реестра форм.

### Параметры render_form()

Все параметры опциональны — значения по умолчанию берутся из реестра:

```twig
{{ render_form('callback', {
    template: 'modal',          # свой шаблон формы
    submit_text: 'Позвоните мне',
    submit_class: 'btn btn-primary',
    form_class: 'my-form',
    action: '/form-handler',    # URL обработчика (по умолчанию)
    hidden_fields: ['utm_source'],
    field_overrides: {name: {label: 'Имя и фамилия'}},
    field_defaults: {utm_source: 'organic'}
}) }}
```

---

## Галочка «Политика конфиденциальности»

Если в таблице формы есть колонка `pd_consent` (тип INTEGER, default 0) — чекбокс согласия на обработку ПД добавляется **автоматически**. Ничего дополнительно настраивать не нужно.

Чекбокс будет обязательным — пользователь не сможет отправить форму, не отметив его.

Ссылка в тексте ведёт на `/privacy` — убедитесь, что такая страница существует на вашем сайте.

### Как это выглядит в шаблоне

Если вы используете стандартный шаблон `default`, чекбокс уже встроен. Если пишете свой шаблон — добавьте:

```twig
{% if show_consent %}
    {% include 'form/fields/consent.html.twig' %}
{% endif %}
```

---

## Шаблоны форм

В комплекте три готовых шаблона:

| Шаблон | Описание |
|--------|----------|
| `default` | Стандартный — с лейблами над полями, Tailwind-стили |
| `minimal` | Минимальный — меньше отступов |
| `hero` | Для лендингов — крупный, с заголовком |

### Свой шаблон

Создайте файл `front/app/views/form/мой_шаблон.html.twig` и укажите его имя в поле **template** при регистрации формы (без `.html.twig`).

Пример минимального кастомного шаблона (без лейблов, только placeholder'ы):

```twig
{% extends "form/_base.html.twig" %}

{% block form_content %}
<form method="post" action="{{ action|default('/form-handler') }}"
      class="my-form {{ form_class }}"
      data-table="{{ source_table }}"
      data-form="{{ form_name }}"
      novalidate>

    <input type="hidden" name="form_name" value="{{ form_name }}">

    {% for field_name, field in fields %}
        {% if field_name not in hidden_fields|default([]) %}
            <input type="{{ field.type }}"
                   name="{{ field_name }}"
                   placeholder="{{ field.placeholder|default(field.label) }}"
                   {% if field.required %}required{% endif %} />
        {% endif %}
    {% endfor %}

    {% if show_consent %}
        {% include 'form/fields/consent.html.twig' %}
    {% endif %}

    <button type="submit">{{ submit_text|default('Отправить') }}</button>
</form>
{% endblock %}
```

### Доступные переменные в шаблоне

| Переменная | Описание |
|---|---|
| `form_name` | Техническое имя формы |
| `fields` | Массив полей из конфига |
| `show_consent` | `true` если в таблице есть колонка `pd_consent` |
| `csrf_token` | Токен CSRF-защиты |
| `action` | URL обработчика |
| `submit_text` | Текст кнопки |
| `submit_class` | CSS-класс кнопки |
| `form_class` | CSS-класс формы |
| `source_table` | Имя таблицы с данными |
| `success_message` | Текст при успешной отправке |

---

## Просмотр заявок

Все заявки собираются в разделе **«Уведомления»** админ-панели:

- **Общий список:** количество заявок по каждой форме, непрочитанные выделены
- **Детально:** список заявок конкретной формы, фильтрация, отметка «прочитано»
- **Просмотр:** отдельная заявка с возможностью удаления
- **Бейдж:** красный кружок в боковом меню показывает общее число непрочитанных

---

## AJAX-отправка (без перезагрузки страницы)

По умолчанию форма отправляется обычным POST с перезагрузкой страницы. Для AJAX-отправки добавьте на страницу скрипт, который перехватывает отправку формы и шлёт запрос с заголовком `X-Requested-With: XMLHttpRequest`.

Сервер сам определит, что это AJAX, и вернёт JSON вместо редиректа:

```json
{"success": true, "message": "Спасибо! Форма успешно отправлена.", "id": 42}
```

Пример скрипта (добавьте в конец Twig-шаблона страницы):

```html
<script>
document.querySelector('form[data-table="form_callback"]')
  .addEventListener('submit', async function(e) {
    e.preventDefault();
    var btn = this.querySelector('button[type="submit"]');
    var orig = btn.textContent;
    btn.disabled = true;
    btn.textContent = 'Отправка...';

    var resp = await fetch('/form-handler', {
      method: 'POST',
      body: new FormData(this),
      headers: {'X-Requested-With': 'XMLHttpRequest'}
    });
    var data = await resp.json();

    if (data.success) {
      this.innerHTML = '<div style="color:green">' + data.message + '</div>';
    } else {
      btn.disabled = false;
      btn.textContent = orig;
      alert(data.message);
    }
  });
</script>
```

---

## Из чего состоит форма (внутреннее устройство)

```
Пользователь заполняет форму
        │
        ▼
POST /form-handler
        │
        ▼
FormRenderer::processSubmission()
  ├─ Проверка honeypot (скрытое поле _hp_email)
  ├─ Проверка времени (форма открыта >3 сек)
  ├─ Валидация CSRF-токена
  ├─ Валидация полей (required, email-формат)
  ├─ Сохранение в таблицу (source_table)
  ├─ Отправка email-уведомлений
  └─ Ответ: JSON (AJAX) или редирект (обычный POST)
```


---
*Последнее обновление: 22 июля 2026 (v1.3.14)*
