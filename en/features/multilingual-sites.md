# Multilingual Sites

apidcms supports optional frontend multilingual mode. It's **disabled by default** — your site works as usual. Additional languages are enabled only when needed.

## How It Works

The base language always has no URL prefix. Additional languages use a prefix:

- `https://mysite.ru/` — base language (usually Russian)
- `https://mysite.ru/en/` — additional language (English)

When a visitor hits `/en/...`, a `site_lang` cookie is set for one year. On the next visit without a prefix, they're automatically redirected to the correct language version.

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
<a href="/">{{ _t[site_lang].home }}</a>
<input placeholder="{{ _t[site_lang].search }}">
```

The `site_lang` variable is available in all templates and contains the current language (`ru`/`en`).

### Adding Translations for a New Page

1. Add keys to the `_t` object in `base.html.twig` (in both `ru` and `en` sections)
2. In the template: `{{ _t[site_lang].your_key }}`
3. Done

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
