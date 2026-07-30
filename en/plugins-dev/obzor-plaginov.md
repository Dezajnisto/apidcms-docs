---
title: Plugins Overview
slug: obzor-plaginov
section: plugins-dev
order: 60
description: "Plugins Overview — extending apidcms functionality without modifying the core."
---

# 🔌 Plugins

Plugins extend apidcms functionality without modifying the core. Each plugin is an isolated folder in `plugins/plugin_name/`.

The core ships with two plugins: **Account** (user cabinet) and **Cart** (shopping cart).

---

## Where to Find

In the admin panel: sidebar menu → **Plugins** (🧩 icon).

---

## Plugin Management

### Plugin List (`/admin/plugins`)

A table of all plugins: name, version, description, status. Buttons for **Activate** / **Deactivate** and **Settings**.

### Plugin Page (`/admin/plugins/name`)

Three tabs:

- **Info** — name, version, status, dependencies, contents of `plugin.json`
- **Templates** — list of plugin `.twig` files, "Edit" button
- **Settings** — plugin parameter form (currency, limits, etc.)

### Template Editor (`/admin/plugins/name/templates/file.twig`)

Full-featured code editor. Save with Ctrl+S or the button.

---

## Installation and Removal

- **Installation:** copy the plugin folder to `plugins/plugin_name/` on the hosting, then activate via the admin panel
- **Removal:** deactivate in the admin panel, then delete the `plugins/plugin_name/` folder
- **Enable/disable:** via the admin panel using Activate/Deactivate buttons

> ⚠️ When deleting the plugin folder, the database tables **remain**. Same on deactivation.

---

## 🔐 Account Plugin — User Cabinet

Registration, login, and user profile.

**Routes:**
- `/register` — registration
- `/login` — login
- `/profile` — profile
- `/logout` — logout

**Features:**
- Remember-me (30-day cookie)
- Twig functions: `is_logged_in()`, `current_user()`

**Tables:** `users`, `user_tokens`

---

## 🛒 Cart Plugin — Shopping Cart

Add products to cart for catalog sites.

**AJAX endpoints:**
- `POST /cart/add` — add item
- `POST /cart/update` — change quantity
- `POST /cart/remove` — remove item

**Pages:**
- `/cart` — cart with +/- and delete

**Twig functions:** `cart_count()`, `cart_total()`

**Table:** `cart_items`

---

## Plugin Settings

Each plugin can have configurable parameters (currency, limits, behavior). Settings are found in the "Settings" tab on the plugin page and are saved to `plugin.json`.

---

## 📊 Plugin Tables in the Admin Panel

Plugins can declare their tables via the `tables` field in `plugin.json`. This lets the admin panel show plugin tables on the dashboard in a separate "Plugins" group (alongside system and user tables).

**Example plugin.json with tables:**

```json
{
  "name": "account",
  "version": "1.0.0",
  "description": "User cabinet",
  "tables": ["plugin_account_users", "plugin_account_tokens"]
}
```

After adding the `tables` field, plugin tables will appear in the "Plugins" section on the admin dashboard.

---

##


## 🏷️ Table Naming Convention

All plugin tables MUST use the `plugin_{name}_` prefix. This prevents name conflicts with user-created tables (e.g. if a user creates a `users` table via admin panel, and the Account plugin tries to create its own).

**Format:** `plugin_{plugin_name}_{purpose}`

**Examples:**
- `plugin_account_users` — Account plugin users table
- `plugin_cart_items` — Cart plugin items table
- `plugin_credits_transactions` — Credits plugin transactions table

**Rules:**
- Plugin name — only `[a-z][a-z0-9]*` (no underscores)
- Table name — lowercase with underscores
- Full name: `plugin_{name}_{purpose}`

### Migrations with Schema Verification

Every plugin migration must not only create tables (`CREATE TABLE IF NOT EXISTS`), but also verify the schema of existing tables via `PRAGMA table_info` and `ALTER TABLE ADD COLUMN`. This ensures that when a plugin is updated, new columns are added without data loss.

**Migration example with PRAGMA verification:**

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

### Foreign Keys

If a table references another plugin's table (e.g. `plugin_account_users`), the plugin must:
- Declare the dependency in `plugin.json`: `"dependencies": ["account"]`
- Use the full table name in FOREIGN KEY
