---
title: Hooks & Events
slug: huki-sobytiya
section: templates
order: 280
description: "apidcms plugin hooks and events system reference"
---

# Hooks (Events)

apidcms provides a hook system for extending functionality through plugins.

---

## Available Hooks

### `core.init`

Called during core initialization. Register global settings.

### `twig.init`

Called when creating the Twig environment. Add custom functions and filters.

```php
$pm->addAction('twig.init', function($frontController, $twig) {
    $twig->addFunction(new TwigFunction('my_helper', function($x) {
        return strtoupper($x);
    }));
});
```

### `front.router.before`

Before route processing. Modify the path or add a redirect.

```php
$pm->addAction('front.router.before', function($path, $frontController) {
    if ($path === 'old-page') {
        header('Location: /new-page', true, 301);
        exit;
    }
});
```

### `front.router.after`

After route processing. Modify the response.

### `front.render`

Called before template rendering. Modify data passed to Twig.

```php
$pm->addFilter('front.render', function($data) {
    $data['cart_count'] = $cartService->getCount();
    return $data;
});
```

### `admin.menu`

Allows plugins to add items to the admin sidebar menu.

```php
$pm->addFilter('admin.menu', function($menu) {
    $menu[] = ['title' => 'My Plugin', 'url' => '/admin/myplugin', 'icon' => '🔧'];
    return $menu;
});
```

### `db.migrate`

Called when `init_system_tables.php` runs. Allows plugins to create their tables.

```php
$pm->addAction('db.migrate', function($db) {
    $pdo = $db->getConnection();
    $pdo->exec('CREATE TABLE IF NOT EXISTS my_table (...)');
});
```

---

## Registering a Hook

In the plugin's `init.php`:

```php
$pm = \Core\PluginManager::getInstance();

$pm->addAction('twig.init', function($fc, $twig) {
    // your code
}, 10, 'my_plugin');
```

| Parameter | Description |
|-----------|-------------|
| `$hook` | Hook name |
| `$callback` | Handler function |
| `$priority` | Priority (10 default, lower = earlier) |
| `$plugin` | Plugin name (for identification) |

---

## Filters vs Actions

- **`addAction`** — execute an action, no return value
- **`addFilter`** — modify a value and return it

```php
$pm->addFilter('page_title', function($title) {
    return $title . ' — My Site';
});
```
