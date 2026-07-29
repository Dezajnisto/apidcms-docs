# Multilingual Sites

apidcms supports optional frontend multilingual mode. It's **disabled by default** — your site works as usual. Additional languages are enabled only when needed.

## How It Works

The base language always has no URL prefix. Additional languages use a prefix:

- `https://mysite.ru/` — base language (usually Russian)
- `https://mysite.ru/en/` — additional language (English)

Visiting `/en/...` sets a `site_lang` cookie (1 year). Returning to the base language (`/...`) clears the cookie. The cookie does **not** trigger automatic redirects — language is controlled solely by the URL.

## Settings

Two keys in `system_settings`:

| Setting | Default | Description |
|---|---|---|
| `site_language` | `ru` | Base frontend language |
| `site_languages` | `null` | JSON array of extra languages. `null` = multilingual disabled |

## Enabling an Additional Language

Via admin panel: Settings → System, or directly in the database:

```sql
UPDATE system_settings SET setting_value = '["en"]'
WHERE setting_key = 'site_languages';
```

After that:

- `/en/page` — English version of any page
- RU/EN language switcher in the header (shown only when `site_languages` is not empty)
- `site_lang` cookie remembers the choice

### Language Switcher Behavior

The switcher preserves the current page when changing languages:

- On `/en/docs` → RU links to `/docs`, EN links to `/en/docs`
- On `/` → RU links to `/`, EN links to `/en/`

This works via the `current_path` variable (URL path without the language prefix), automatically passed to all templates.

## Template Translations

Static strings (navigation, buttons, headings) use the `_t` object in `base.html.twig`:

```twig
{% set _t = {
  ru: {
    home: "Главная",
    docs: "Документация",
    search: "Поиск"
  },
  en: {
    home: "Home",
    docs: "Documentation",
    search: "Search"
  }
} %}

<html lang="{{ site_lang }}">
...
<a href="{{ url('/') }}">{{ _t[site_lang].home }}</a>
<input placeholder="{{ _t[site_lang].search }}">
```

### Important: Use url() for All Internal Links

The `url()` function automatically adds the language prefix for pages (but not for assets):

```twig
{# Correct — prefix added automatically #}
<a href="{{ url('/docs') }}">{{ _t[site_lang].docs }}</a>

{# Wrong — link never gets the prefix #}
<a href="/docs">{{ _t[site_lang].docs }}</a>
```

On an EN page, `url('/docs')` returns `/en/docs`; on RU, `/docs`. Asset paths (`/storage/...`) never receive a prefix.

### Language Switcher Template

```twig
{% if get_setting('site_languages')|default('') is not empty %}
<div class="lang-switch">
    <a href="/{{ current_path }}"
       class="lang-link{% if site_lang == 'ru' %} active{% endif %}">RU</a>
    <a href="/en/{{ current_path }}"
       class="lang-link{% if site_lang == 'en' %} active{% endif %}">EN</a>
</div>
{% endif %}
```

Template variables:
- `site_lang` — current language (`ru`/`en`)
- `current_path` — page path without prefix (on `/en/docs` → `docs`)

### Adding Translations for a New Page

1. Add keys to the `_t` object in `base.html.twig` (in both `ru` and `en` sections)
2. In the template: `{{ _t[site_lang].your_key }}`
3. All internal links: `{{ url('/path') }}`
4. Done

## Core\I18n — Programmatic Access

```php
use Core\I18n;

// Resolve a single value
$title = I18n::resolve($row['title'], 'en');
// "Привет" → "Привет" (plain text)
// '{"ru":"Привет","en":"Hello"}' → "Hello"

// Resolve an array of items
$items = I18n::resolveArray($items,
    ['title', 'description', 'content'], 'en');

// Build i18n JSON for storage
$json = I18n::encode(
    ['ru' => 'Привет', 'en' => 'Hello'], 'ru');
// → '{"ru":"Привет","en":"Hello"}'
// If only base language → plain text "Привет"

// SQL expression for locale-aware search
$expr = I18n::searchExpr('title', ['ru', 'en']);
// → COALESCE(json_extract(title, '$.ru'),
//            json_extract(title, '$.en'), title)
```

## JSON Format for Content

Content (pages, posts) can be stored as JSON:

```sql
-- Single language
title = "Home page"

-- Multiple languages
title = '{"ru":"Главная","en":"Home page"}'
```

`FrontController::render()` automatically resolves `title`, `description`, `content` and other fields via `resolveI18nData()`. Templates receive ready-to-use strings.

## Search

Search works via `LIKE` on text columns. For JSON fields, it finds keys along with values (minor noise). `json_extract()` support for clean locale-aware search is planned.

## Limitations

- **Content** (pages, posts) is currently single-language. JSON format for content is the next step.
- **Accept-Language** browser detection is not implemented.
- **Admin panel** always uses `admin_language`, independent of the frontend.

## See Also

- [Internationalization](/docs/i18n) — admin panel translation
- [apidcms.dezajno.ru](https://apidcms.dezajno.ru/en/) — live multilingual site example
