---
title: Creating a Form
slug: sozdanie-formy
section: forms
order: 170
description: "Step-by-step guide: creating a table, configuring form_config, fields, notifications, embedding via Twig."
---

# Creating a Form

apidcms lets you create feedback, application, order, and any other forms — without programming. Everything is configured through the admin panel.

---

## How It Works

A form consists of three parts:

1. **Database table** — where submissions are stored
2. **Entry in the forms registry** (`forms`) — configuration: fields, notifications, design
3. **Template** — how the form looks (Twig)

Once configured, a form can be embedded on any page with a single line in a Twig template.

---

## Step 1. Create a Table for Storing Submissions

1. In the admin panel, click **"Create Table"**
2. Enter a name (e.g., `form_callback`)
3. Add fields for your data:
   - `name` → `TEXT` — name
   - `phone` → `TEXT` — phone
   - `message` → `TEXT` — message
4. **Required: add service columns:**
   - `pd_consent` → `INTEGER`, default `0` — for the personal data consent checkbox
   - `read_status` → `TEXT`, default `unread` — for read/unread marking
   - `form_name` → `TEXT` — filled automatically, needed for filtering
5. Click **"Create"**

---

## Step 2. Register the Form

1. In the admin sidebar, click **"Forms"**
2. Click **"Add Form"**
3. Fill in the fields:

### Basic

| Field | Description | Example |
|-------|-------------|---------|
| **Form name (name)** | Technical name, Latin characters | `callback` |
| **Display name** | Name shown in the admin panel | "Callback" |
| **Data table** | Table name from Step 1 | `form_callback` |
| **Template** | `.html.twig` file in `form/` | `default` (standard) or custom |

### Form Fields (fields) — JSON

Describes which fields the form will have. Format:

```json
{
  "name": {
    "label": "Your Name",
    "type": "text",
    "placeholder": "John",
    "required": true
  },
  "phone": {
    "label": "Phone Number",
    "type": "tel",
    "placeholder": "+1 (555) 123-45-67",
    "required": true
  },
  "message": {
    "label": "Comment",
    "type": "textarea",
    "placeholder": "Describe your question",
    "required": false
  }
}
```

**Available field types:** `text`, `email`, `tel`, `textarea`, `select`, `checkbox`, `number`, `date`.

**Important:** field names in JSON must match the column names in the table. Fields `id`, `created_at`, `updated_at`, `read_status` are system fields — do not add them to `fields`.

### Notifications — JSON

```json
{
  "admin_notify": true,
  "admin_emails": ["your@email.ru"],
  "admin_subject": "New submission from the site",
  "auto_reply": false
}
```

| Field | Description |
|-------|-------------|
| `admin_notify` | Send notification to admin (true/false) |
| `admin_emails` | List of email addresses for notifications |
| `admin_subject` | Email subject |
| `auto_reply` | Send auto-reply to user |
| `auto_reply_field` | Which field to use for the user's email |
| `auto_reply_subject` | Auto-reply subject |

### Design — JSON

```json
{
  "submit_text": "Submit",
  "submit_class": "btn btn--primary",
  "form_class": ""
}
```

| Field | Description |
|-------|-------------|
| `submit_text` | Submit button text |
| `submit_class` | CSS class for the button |
| `form_class` | Additional CSS class for `<form>` |
| `field_class` | CSS class for input fields |
| `label_class` | CSS class for field labels |

---

## Step 3. Embed the Form on a Page

In any Twig template, use the `render_form()` function:

```twig
{{ render_form('callback', {template: 'default'}) }}
```

Where `callback` is the value of the **name** field from the form registry.

### render_form() Parameters

All parameters are optional — default values are taken from the registry:

```twig
{{ render_form('callback', {
    template: 'modal',          # custom form template
    submit_text: 'Call me',
    submit_class: 'btn btn-primary',
    form_class: 'my-form',
    action: '/form-handler',    # handler URL (default)
    hidden_fields: ['utm_source'],
    field_overrides: {name: {label: 'First and last name'}},
    field_defaults: {utm_source: 'organic'}
}) }}
```

---

## Privacy Policy Checkbox

If the form table has a `pd_consent` column (type INTEGER, default 0) — the personal data consent checkbox is added **automatically**. No additional configuration needed.

The checkbox will be mandatory — users cannot submit the form without checking it.

The link in the text points to `/privacy` — make sure such a page exists on your site.

### How It Looks in a Template

If you use the standard `default` template, the checkbox is already built in. If you are writing your own template, add:

```twig
{% if show_consent %}
    {% include 'form/fields/consent.html.twig' %}
{% endif %}
```

---

## Form Templates

Three ready-made templates are included:

| Template | Description |
|----------|-------------|
| `default` | Standard — labels above fields, Tailwind styles |
| `minimal` | Minimal — less spacing |
| `hero` | For landing pages — large, with a heading |

### Custom Template

Create a file `front/app/views/form/my_template.html.twig` and specify its name in the **template** field when registering the form (without `.html.twig`).

Example of a minimal custom template (no labels, placeholders only):

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

    <button type="submit">{{ submit_text|default('Submit') }}</button>
</form>
{% endblock %}
```

### Available Template Variables

| Variable | Description |
|----------|-------------|
| `form_name` | Technical form name |
| `fields` | Array of fields from config |
| `show_consent` | `true` if the table has a `pd_consent` column |
| `csrf_token` | CSRF protection token |
| `action` | Handler URL |
| `submit_text` | Button text |
| `submit_class` | Button CSS class |
| `form_class` | Form CSS class |
| `source_table` | Data table name |
| `success_message` | Text on successful submission |

---

## Viewing Submissions

All submissions are collected in the **"Notifications"** section of the admin panel:

- **Overview:** number of submissions per form, unread highlighted
- **Details:** list of submissions for a specific form, filtering, mark as read
- **View:** individual submission with delete option
- **Badge:** red circle in the sidebar shows total unread count

---

## AJAX Submission (Without Page Reload)

By default, the form submits via standard POST with page reload. For AJAX submission, add a script to the page that intercepts form submission and sends the request with the `X-Requested-With: XMLHttpRequest` header.

The server will detect it as AJAX and return JSON instead of a redirect:

```json
{"success": true, "message": "Thank you! The form was submitted successfully.", "id": 42}
```

Example script (add at the end of your page's Twig template):

```html
<script>
document.querySelector('form[data-table="form_callback"]')
  .addEventListener('submit', async function(e) {
    e.preventDefault();
    var btn = this.querySelector('button[type="submit"]');
    var orig = btn.textContent;
    btn.disabled = true;
    btn.textContent = 'Sending...';

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

## Form Internal Architecture

```
User fills out the form
        │
        ▼
POST /form-handler
        │
        ▼
FormRenderer::processSubmission()
  ├─ Honeypot check (hidden field _hp_email)
  ├─ Timing check (form open >3 sec)
  ├─ CSRF token validation
  ├─ Field validation (required, email format)
  ├─ Save to table (source_table)
  ├─ Send email notifications
  └─ Response: JSON (AJAX) or redirect (standard POST)
```


---
*Last updated: July 22, 2026 (v1.3.14)*