# Internationalization (i18n) in apidcms

apidcms supports a multilingual admin panel interface. The system is built on JSON language packs, the `lang()` Twig function, and the `__t()` JavaScript helper.

## How It Works

```
system_settings.admin_language = "en"
         ↓
Lang::getInstance('en')
         ↓
1. Loads admin/lang/ru.json as base (fallback)
2. Overlays admin/lang/en.json on top (overrides matching keys)
         ↓
{{ lang('sidebar.home') }} → "Home" (if en.json contains the key)
                              "Главная" (if key is missing — fallback to ru)
```

## Language File Structure

```
core_lib/admin/lang/
├── ru.json    ← 1200 keys (required, fallback)
└── en.json    ← 1200 keys (canonical key names)
```

**Key format:** `section.subsection.element`

```json
{
  "sidebar.home": "Home",
  "common.save": "Save",
  "table.back_to_list": "Back to table list",
  "settings.maintenance_on": "On — visitors see a maintenance page"
}
```

**Parameters via `{name}` placeholders:**

```json
{
  "forms.saved": "Form '{name}' saved",
  "table.confirm_delete": "Delete record #%id%?"
}
```

## Strict Rules

1. **Canonical language is English.** Keys must be in English (`sidebar.home`, not `sidebar.main`)
2. **Russian must always be complete.** Every key in `en.json` MUST have an entry in `ru.json`
3. **Fallback to Russian.** If a key is missing in the selected locale, the Russian translation is returned
4. **Double braces in Twig.** `{{ lang('key') }}` — must be double. Single braces `{ }` are output as literal text by Twig
5. **`$this->lang->t()` in controllers.** Not `Lang::t()` statically — `__callStatic` does not work in PHP 8

## Using in Twig Templates

```twig
{# Simple key #}
<h1>{{ lang('sidebar.home') }}</h1>

{# With parameters #}
<p>{{ lang('table.empty_no_data')|replace({'%table%': tableName}) }}</p>

{# In attributes #}
<input placeholder="{{ lang('table.search_placeholder') }}">

{# In JavaScript (via __t) #}
<script>
alert(__t('filemanager.link_copied'));
</script>
```

## Using in PHP Controllers

```php
// Page title
$data['title'] = $this->lang->t('settings.page_title');

// Flash message with parameters
$this->setFlash('success', $this->lang->t('forms.form_saved', ['name' => $name]));

// Exception
throw new Exception($this->lang->t('settings.key_not_specified'));
```

## Using in JavaScript

```javascript
// window.__lang contains all 1200 keys
// __t(key) is a shorthand helper
showNotification(__t('filemanager.link_copied'), 'success');

// With concatenation
alert(__t('filemanager.error_upload') + ': ' + error.message);
```

## Adding a New Language

### 1. Create a JSON language pack

```bash
cp core_lib/admin/lang/en.json core_lib/admin/lang/de.json
```

### 2. Translate the values

Edit `de.json` — replace English values with the target language. Do not touch the keys.

```json
{
  "sidebar.home": "Startseite",
  "common.save": "Speichern",
  ...
}
```

You don't need to translate all 1200 keys at once. Missing keys will automatically fall back to Russian.

### 3. Add language to settings dropdown

In `admin/app/views/settings/index.html.twig`, find the language selector:

```twig
<select name="settings[admin_language]" ...>
    <option value="ru">Русский</option>
    <option value="en">English</option>
    <option value="de">Deutsch</option>   ← add this
</select>
```

### 4. Deploy

```bash
sync-core all
# Clear caches on all sites
```

The new language will appear in Settings → General → Interface Language.

## Translation Audit

To check translation completeness:

```bash
# Find all lang() calls in templates
grep -roh "lang('[^']*'" admin/app/views/ | sort -u

# Check against JSON (Python)
python3 -c "
import json, re, os
with open('admin/lang/en.json') as f:
    keys = set(json.load(f).keys())
# Find template keys missing from JSON
...
"
```

## The Lang Class

```php
namespace Admin\Core;

class Lang {
    public static function getInstance(string $locale = 'ru'): self;
    public static function reload(string $locale): self;  // switch language
    public function t(string $key, array $params = [], ?string $default = null): string;
    public function getLocale(): string;
    public function all(): array;  // all translations for window.__lang
}
```

## Current Status

| Language | Keys | Status |
|----------|------|--------|
| Russian (ru) | 1200 | Complete ✅ |
| English (en) | 1200 | Complete ✅ |
