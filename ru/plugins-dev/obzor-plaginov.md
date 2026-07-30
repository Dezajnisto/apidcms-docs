---
title: Обзор плагинов
slug: obzor-plaginov
section: plugins-dev
order: 60
---

# 🔌 Плагины

Плагины расширяют функциональность apidcms без правки ядра. Каждый плагин — изолированная папка в `plugins/имя_плагина/`.

В ядро входят два плагина: **Account** (личный кабинет) и **Cart** (корзина).

---

## Где найти

В админ-панели: боковое меню → **Плагины** (иконка 🧩).

---

## Управление плагинами

### Список плагинов (`/admin/plugins`)

Таблица всех плагинов: название, версия, описание, статус. Доступны кнопки **Активировать** / **Деактивировать** и **Настроить**.

### Страница плагина (`/admin/plugins/имя`)

Три вкладки:

- **Инфо** — название, версия, статус, зависимости, содержимое `plugin.json`
- **Шаблоны** — список `.twig`-файлов плагина, кнопка «Редактировать»
- **Настройки** — форма с параметрами плагина (валюта, лимиты и т.д.)

### Редактор шаблонов (`/admin/plugins/имя/шаблонов/файл.twig`)

Полноценный редактор кода. Сохранение по Ctrl+S или кнопкой.

---

## Установка и удаление

- **Установка:** скопировать папку плагина в `plugins/имя_плагина/` на хостинге, затем активировать через админку
- **Удаление:** деактивировать в админке, затем удалить папку `plugins/имя_плагина/`
- **Вкл/выкл:** через админку кнопками Активировать/Деактивировать

> ⚠️ При удалении папки плагина таблицы в базе данных **остаются**. При деактивации — тоже.

---

## 🔐 Плагин Account — Личный кабинет

Регистрация, авторизация и профиль пользователей.

**Маршруты:**
- `/register` — регистрация
- `/login` — вход
- `/profile` — профиль
- `/logout` — выход

**Особенности:**
- Remember-me (cookie на 30 дней)
- Twig-функции: `is_logged_in()`, `current_user()`

**Таблицы:** `users`, `user_tokens`

---

## 🛒 Плагин Cart — Корзина

Добавление товаров в корзину для сайтов-каталогов.

**AJAX-эндпоинты:**
- `POST /cart/add` — добавить товар
- `POST /cart/update` — изменить количество
- `POST /cart/remove` — удалить товар

**Страницы:**
- `/cart` — корзина с +/- и удалением

**Twig-функции:** `cart_count()`, `cart_total()`

**Таблица:** `cart_items`

---

## Настройки плагинов

Каждый плагин может иметь настраиваемые параметры (валюта, лимиты, поведение). Настройки находятся во вкладке «Настройки» на странице плагина и сохраняются в `plugin.json`.
---

## 📊 Таблицы плагинов в админке

Плагины могут объявлять свои таблицы через поле `tables` в `plugin.json`. Это позволяет админке показывать таблицы плагина на главной странице в отдельной группе «Плагины» (рядом с системными и пользовательскими).

**Пример plugin.json с таблицами:**

```json
{
  "name": "account",
  "version": "1.0.0",
  "description": "Личный кабинет",
  "tables": ["plugin_account_users", "plugin_account_tokens"]
}
```

После добавления поля `tables` таблицы плагина появятся в секции «Плагины» на дашборде админки.

---

##


## 🏷️ Соглашение об именовании таблиц

Все таблицы плагинов должны использовать префикс `plugin_{name}_`. Это предотвращает конфликты имён с пользовательскими таблицами (например, если пользователь создаст таблицу `users` через админку, а плагин Account попытается создать свою).

**Формат:** `plugin_{имя_плагина}_{назначение}`

**Примеры:**
- `plugin_account_users` — таблица пользователей плагина Account
- `plugin_cart_items` — таблица элементов корзины плагина Cart
- `plugin_credits_transactions` — таблица транзакций плагина Credits

**Правила:**
- Имя плагина — только `[a-z][a-z0-9]*` (без подчёркиваний)
- Имя таблицы — строчные буквы и подчёркивания
- Полное имя: `plugin_{name}_{purpose}`

### Миграции с верификацией схемы

Каждая миграция плагина должна не только создавать таблицы (`CREATE TABLE IF NOT EXISTS`), но и верифицировать схему существующих таблиц через `PRAGMA table_info` и `ALTER TABLE ADD COLUMN`. Это гарантирует, что при обновлении плагина новые колонки будут добавлены без потери данных.

**Пример миграции с PRAGMA-верификацией:**

```php
$pm->addAction('db.migrate', function ($db) use ($pluginDir) {
    $migrationFile = $pluginDir . '/migrations/001_create.sql';
    if (!file_exists($migrationFile)) return;
    $sql = file_get_contents($migrationFile);
    $statements = array_filter(array_map('trim', explode(';', $sql)));
    foreach ($statements as $stmt) {
        if (!empty($stmt)) {
            try { $db->exec($stmt); }
            catch (\Exception $e) { error_log("Migration error: " . $e->getMessage()); }
        }
    }

    // Schema verification
    $expectedColumns = [
        'plugin_myplugin_items' => [
            'id' => 'INTEGER',
            'name' => 'TEXT',
            'created_at' => 'DATETIME'
        ]
    ];

    foreach ($expectedColumns as $table => $columns) {
        try {
            $existing = $db->query("PRAGMA table_info(\"{$table}\")")->fetchAll();
            $existingNames = array_column($existing, 'name');
            foreach ($columns as $colName => $colType) {
                if (!in_array($colName, $existingNames)) {
                    $db->exec("ALTER TABLE \"{$table}\" ADD COLUMN \"{$colName}\" {$colType} DEFAULT ''");
                    error_log("Added missing column {$table}.{$colName}");
                }
            }
        } catch (\Exception $e) {
            error_log("Schema verification error: " . $e->getMessage());
        }
    }
}, 10, 'myplugin');
```

### Внешние ключи

Если таблица ссылается на таблицу другого плагина (например, `plugin_account_users`), плагин должен:
- Указать зависимость в `plugin.json`: `"dependencies": ["account"]`
- Использовать полное имя таблицы в FOREIGN KEY
