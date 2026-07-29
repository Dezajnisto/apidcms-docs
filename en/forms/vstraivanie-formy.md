---
title: Embedding a Form
slug: vstraivanie-formy
section: forms
order: 200
description: "Embedding a form on any page using render_form()."
---

# 🔌 Embedding a Form on Any Page

The `render_form()` function allows you to insert a form anywhere — not just on a separate page.

---

## Basic Call

```twig
{{ render_form('contacts') }}
```

Where `contacts` is the table name (from the `name` column in the `forms` table).

---

## With Parameters

```twig
{{ render_form('orders', {
    submit_text: 'Order',
    form_class: 'order-form',
    consent_enabled: true
}) }}
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `submit_text` | `Submit` | Button text |
| `form_class` | — | CSS class on `<form>` |
| `field_class` | — | CSS class on fields |
| `consent_enabled` | `false` | Show PD consent checkbox |
| `template` | from form_config | Override template |

---

## Example: Form in Footer

```twig
{# in base.html.twig #}
<footer>
  <div class="footer-form">
    <h3>Still have questions?</h3>
    {{ render_form('contacts', {submit_text: 'Ask'}) }}
  </div>
</footer>
```

---

## Difference from a Form Page

| | `page_type = 'form'` | `render_form()` |
|---|---|---|
| Separate URL | ✅ `/contact` | ❌ embedded |
| Settings | `form_config` in JSON | parameters in the call |
| Validation | ✅ | ✅ |
| Notifications | ✅ | ✅ |

Both approaches use the same table and `FormRenderer`.