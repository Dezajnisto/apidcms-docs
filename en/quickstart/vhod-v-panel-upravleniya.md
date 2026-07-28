---
title: Logging into the Admin Panel
slug: vhod-v-panel-upravleniya
section: quickstart
order: 10
description: "How to log into apidcms admin panel, troubleshooting login issues, logout."
---

# Logging into the Admin Panel

## Default Credentials

After installation, the admin panel is available at:

```
https://your-site.ru/admin/
```

Default login credentials:

- **Username:** `admin`
- **Password:** `admin`

## First Login

1. Open `https://your-site.ru/admin/` in your browser
2. Enter your username and password
3. Click "Sign In"

> Note: change the default password immediately in `admin/config/config.php`.

## Changing Password

1. Open `admin/config/config.php` on the server
2. Update the `admin_password` value
3. Save the file — changes take effect immediately

```php
'security' => [
    'admin_username' => 'admin',
    'admin_password' => 'your_new_password',
],
```

## Troubleshooting

**"Invalid username or password"**
- Check `admin/config/config.php` — credentials are case-sensitive

**White page after login**
- Check PHP error log: `log/php-errors.log`
- Clear Twig cache: delete `storage/cache/twig_admin/`

**Redirect loop**
- Check that `session.timeout` is set correctly in config
- Clear browser cookies for the domain

## Logout

Click the red "Logout" button at the bottom of the sidebar.
