---
title: Multilingual Sites
slug: multilingual-sites
section: features
order: 55
description: How to build a multilingual site: settings, URL prefixes, _t object, Core\I18n class.
---

# Multilingual Sites

apidcms supports building sites in multiple languages through a URL prefix system, cookies, and the `_t` translation object.

## How to Enable

In the admin panel, go to **Settings** (`/admin/settings`). Find these two options:

| Setting | Description | Example |
|---------|-------------|---------|
| `site_language` | Default site language | `ru` |
| `site_languages` | Additional languages (JSON array) | `["en", "de"]` |

After saving, a language switcher appears on the frontend, and all pages become available with a language prefix in the URL.

**Important:** the system is opt-in — multilingual support is inactive until `site_languages` is populated.

## How It Works

### URL Prefixes

Each additional language gets a prefix in the URL:

| Base language (ru) | English (en) |
|--------------------|--------------|
| `/changelog` | `/en/changelog` |
| `/docs/install` | `/en/docs/install` |
| `/plugins` | `/en/plugins` |

The base language (`site_language`) does **not** get a prefix — its URLs remain bare. This preserves backward compatibility with existing links.

### The site_lang Cookie

When visiting a page with a language prefix, the system sets a `site_lang` cookie for 1 year. If the user navigates to a bare URL (without prefix), the system checks the cookie and serves the stored language.

Switching to the base language is done via the `?lang=ru` query parameter — it clears the cookie and returns to the base locale.

### The _t Object in Templates

All translations are stored in the `base.html.twig` template as a `_t` object:

```twig
{% set _t = {
  ru: {
    home: "Главная",
    docs: "Документация",
    changelog: "CHANGELOG"
  },
  en: {
    home: "Home",
    docs: "Documentation",
    changelog: "Changelog"
  }
} %}
```

Usage in any template:

```twig
<h1>{{ _t[site_lang].changelog_heading }}</h1>
<a href="{{ url('/docs') }}">{{ _t[site_lang].docs }}</a>
```

The `site_lang` variable is automatically passed to all templates — it contains the current language (`ru` or `en`).

### The url() Function

All internal links must use `url()`, not hardcoded paths:

```twig
{# Correct — prefix is added automatically #}
<a href="{{ url('/changelog') }}">Changelog</a>

{# Wrong — link always goes to the base language #}
<a href="/changelog">Changelog</a>
```

`url()` checks the current locale and, if it differs from the base language, adds the prefix:

- When `site_lang = en`: `url('/changelog')` → `/en/changelog`
- When `site_lang = ru`: `url('/changelog')` → `/changelog`

System paths (`/admin`, `/storage`, `/static`) are never prefixed.

### Language Switcher

A minimal switcher in `base.html.twig`:

```twig
{% if get_setting('site_languages')|default('') is not empty %}
<div class="lang-switch">
    <a href="/{{ current_path }}?lang=ru"
       class="{% if site_lang == 'ru' %}active{% endif %}">RU</a>
    <a href="/en/{{ current_path }}"
       class="{% if site_lang == 'en' %}active{% endif %}">EN</a>
</div>
{% endif %}
```

- **Base language (RU):** `/{current_path}?lang=ru` — query parameter signals explicit switch
- **Additional language (EN):** `/en/{current_path}` — URL prefix

The `current_path` variable contains the current path **without** the locale prefix.

### current_path

The `current_path` variable is automatically stripped of the locale prefix:

| Page URL | `site_lang` | `current_path` |
|----------|-------------|----------------|
| `/changelog` | `ru` | `changelog` |
| `/en/changelog` | `en` | `changelog` |
| `/en/changelog/v2-1-1` | `en` | `changelog/v2-1-1` |

This allows the language switcher to always generate correct links.

## Core\I18n Class

For advanced translation handling in PHP, use the `Core\I18n` class.

### resolve()

Extracts the value for a locale from an i18n object:

```php
use Core\I18n;

$title = I18n::resolve($item['title'], 'ru');  // {"ru":"Hello","en":"Привет"} → "Hello"
$title = I18n::resolve($item['title'], 'en');  // → "Привет"
```

### resolveArray()

Recursively walks an array and resolves all i18n fields:

```php
$items = I18n::resolveArray($items, 'en');
// All fields like {"ru":"...","en":"..."} are replaced with English values
```

Used in `FrontController::resolveI18nData()` for automatic translation of data before rendering.

### searchExpr()

Escapes a search query for different locales:

```php
$expr = I18n::searchExpr($query, 'en');
```

Useful when implementing search on multilingual sites.

### encode()

Encodes a value with locale awareness:

```php
$encoded = I18n::encode($value, 'en');
```

## External Pages and content_url_{locale}

For external pages (changelog, plugins), you can specify different content URLs for different languages.

Instead of a single `content_url`, use:

| Field | Purpose |
|-------|---------|
| `content_url` | Default URL (base language) |
| `content_url_ru` | URL for Russian version |
| `content_url_en` | URL for English version |

If `content_url_{locale}` is not set, `content_url` is used as fallback.

## Setting <html lang>

The `base.html.twig` template should use `site_lang` for the `lang` attribute:

```twig
<html lang="{{ site_lang }}">
```

This is important for SEO and accessibility (screen readers).

## Known Limitations

- **No auto-redirect.** The system does not redirect from a bare URL to a prefixed one. If the user has a `site_lang=en` cookie and visits `/changelog`, the page renders in English but the URL stays bare.
- **Prefix method only.** Subdomain-based (`en.example.com`) or parameter-based (`?lang=en`) persistent routing is not supported.
- **Translations in templates.** All `_t` strings live in `base.html.twig`. For larger projects, consider extracting translations to a separate Twig file via `include`.
- **No content translation.** The system only translates the UI (navigation, buttons, headings). Page content (articles, text) requires manual duplication for each locale.
