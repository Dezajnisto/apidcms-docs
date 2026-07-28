---
title: Хуки (события)
slug: huki-sobytiya
section: templates
order: 280
description: "Хуки (события)"
---

# 🪝 Хуки (события)

apidcms предоставляет систему хуков для расширения функциональности через плагины.

---

## Доступные хуки

### `core.init`

Вызывается при инициализации ядра. Можно зарегистрировать глобальные настройки.

### `twig.init`

Вызывается при создании Twig-окружения. Позволяет добавить свои функции и фильтры.

```php
$pm->addAction('twig.init', function($frontController, $twig) {
    $twig->addFunction(new TwigFunction('my_helper', function($x) {
        return strtoupper($x);
    }));
});
```

### `front.router.before`

Перед обработкой маршрута. Можно изменить путь или добавить редирект.

```php
$pm->addAction('front.router.before', function($path, $frontController) {
    if ($path === 'old-page') {
        header('Location: /new-page', true, 301);
        exit;
    }
});
```

### `front.router.after`

После обработки маршрута. Можно модифицировать ответ.

### `front.render`

Вызывается перед рендером шаблона. Позволяет модифицировать данные, передаваемые в Twig.

```php
$pm->addFilter('front.render', function($data) {
    $data['cart_count'] = $cartService->getCount();
    return $data;
});
```

### `admin.menu`

Позволяет плагинам добавлять пункты в боковое меню админки.

```php
$pm->addFilter('admin.menu', function($menu) {
    $menu[] = ['title' => 'Мой плагин', 'url' => '/admin/myplugin', 'icon' => '🔧'];
    return $menu;
});
```

### `db.migrate`

Вызывается при запуске `init_system_tables.php`. Позволяет плагинам создавать свои таблицы.

```php
$pm->addAction('db.migrate', function($db) {
    $pdo = $db->getConnection();
    $pdo->exec('CREATE TABLE IF NOT EXISTS my_table (...)');
});
```

---

## Регистрация хука

В `init.php` плагина:

```php
$pm = \Core\PluginManager::getInstance();

$pm->addAction('twig.init', function($fc, $twig) {
    // ваш код
}, 10, 'my_plugin');
```

| Параметр | Описание |
|----------|----------|
| `$hook` | Имя хука |
| `$callback` | Функция-обработчик |
| `$priority` | Приоритет (10 по умолчанию, меньше = раньше) |
| `$plugin` | Имя плагина (для идентификации) |

---

## Фильтры vs Экшены

- **`addAction`** — выполнить действие, ничего не возвращает
- **`addFilter`** — модифицировать значение и вернуть

```php
$pm->addFilter('page_title', function($title) {
    return $title . ' — Мой сайт';
});
```
