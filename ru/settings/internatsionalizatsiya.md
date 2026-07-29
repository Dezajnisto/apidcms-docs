# Интернационализация (i18n) в apidcms

apidcms поддерживает мультиязычный интерфейс админ-панели. Система построена на JSON-пакетах, функции `lang()` в Twig-шаблонах и хелпере `__t()` в JavaScript.

## Как это работает

```
system_settings.admin_language = "en"
         ↓
Lang::getInstance('en')
         ↓
1. Загружает admin/lang/ru.json как базу (fallback)
2. Поверх накладывает admin/lang/en.json (перекрывает совпадающие ключи)
         ↓
{{ lang('sidebar.home') }} → "Home" (если en.json содержит ключ)
                              "Главная" (если ключа нет — fallback на ru)
```

## Структура языковых файлов

```
core_lib/admin/lang/
├── ru.json    ← 1200 ключей (обязательный, fallback)
└── en.json    ← 1200 ключей (канонические имена)
```

**Формат ключей:** `section.subsection.element`

```json
{
  "sidebar.home": "Главная",
  "common.save": "Сохранить",
  "table.back_to_list": "Назад к списку таблиц",
  "settings.maintenance_on": "Включён — посетители видят заглушку"
}
```

**Параметры через `{name}` плейсхолдеры:**

```json
{
  "forms.saved": "Форма '{name}' сохранена",
  "table.confirm_delete": "Удалить запись #%id%?"
}
```

## Жёсткие правила

1. **Канонический язык — английский.** Ключи только на английском (`sidebar.home`, а не `сайдбар.главная`)
2. **Русский всегда полный.** Каждый ключ в `en.json` ОБЯЗАН иметь запись в `ru.json`
3. **Fallback на русский.** Если ключ отсутствует в выбранной локали, возвращается русский перевод
4. **Двойные скобки в Twig.** `{{ lang('key') }}` — двойные. Одинарные `{ }` Twig выводит как текст
5. **`$this->lang->t()` в контроллерах.** Не `Lang::t()` статически — `__callStatic` не работает в PHP 8

## Использование в Twig-шаблонах

```twig
{# Простой ключ #}
<h1>{{ lang('sidebar.home') }}</h1>

{# С параметрами #}
<p>{{ lang('table.empty_no_data')|replace({'%table%': tableName}) }}</p>

{# В атрибутах #}
<input placeholder="{{ lang('table.search_placeholder') }}">

{# В JavaScript (через __t) #}
<script>
alert(__t('filemanager.link_copied'));
</script>
```

## Использование в PHP-контроллерах

```php
// Заголовок страницы
$data['title'] = $this->lang->t('settings.page_title');

// Флеш-сообщение с параметрами
$this->setFlash('success', $this->lang->t('forms.form_saved', ['name' => $name]));

// Исключение
throw new Exception($this->lang->t('settings.key_not_specified'));
```

## Использование в JavaScript

```javascript
// window.__lang содержит все 1200 ключей
// __t(key) — краткий хелпер
showNotification(__t('filemanager.link_copied'), 'success');

// С конкатенацией
alert(__t('filemanager.error_upload') + ': ' + error.message);
```

## Как добавить новый язык

### 1. Создать JSON-пакет

```bash
cp core_lib/admin/lang/en.json core_lib/admin/lang/de.json
```

### 2. Перевести значения

Отредактировать `de.json` — заменить английские значения на целевой язык. Ключи не трогать.

```json
{
  "sidebar.home": "Startseite",
  "common.save": "Speichern",
  ...
}
```

Не обязательно переводить все 1200 ключей сразу. Отсутствующие ключи автоматически вернут русский перевод (fallback).

### 3. Добавить язык в настройки

В `admin/app/views/settings/index.html.twig` найти выпадашку языка:

```twig
<select name="settings[admin_language]" ...>
    <option value="ru">Русский</option>
    <option value="en">English</option>
    <option value="de">Deutsch</option>   ← добавить
</select>
```

### 4. Применить

```bash
sync-core all
# Сбросить кэши на всех сайтах
```

После этого новый язык появится в выпадашке Настройки → Основные → Язык интерфейса.

## Аудит переводов

Для проверки полноты перевода используйте поиск:

```bash
# Найти все вызовы lang() в шаблонах
grep -roh "lang('[^']*'" admin/app/views/ | sort -u

# Проверить наличие в JSON (Python)
python3 -c "
import json, re, os
with open('admin/lang/en.json') as f:
    keys = set(json.load(f).keys())
# Найти ключи из шаблонов, которых нет в JSON
for path in ...:
    ...
"
```

## Класс Lang

```php
namespace Admin\Core;

class Lang {
    public static function getInstance(string $locale = 'ru'): self;
    public static function reload(string $locale): self;  // смена языка
    public function t(string $key, array $params = [], ?string $default = null): string;
    public function getLocale(): string;
    public function all(): array;  // все переводы для window.__lang
}
```

## Текущее состояние

| Язык | Ключей | Статус |
|------|--------|--------|
| Русский (ru) | 1200 | Полный ✅ |
| Английский (en) | 1200 | Полный ✅ |
