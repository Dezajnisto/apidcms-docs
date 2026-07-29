---
title: Security
slug: bezopasnost
section: quickstart
order: 50
description: "Where the database is located, how to protect it from direct download. Configuring .htaccess and nginx, verifying protection, regular audits."
---

# Security

apidcms uses SQLite — a database file on the server. This architecture provides simplicity but requires attention to security.

## Where the Database Is

The database is **always** located at `/admin/storage/database/cms.db` — inside the admin panel. This is the correct path, set in the core fallback config (`core_lib/front/config/config.php`).

**Important:** there should be no other copies of `cms.db` in the project. After a `core update`, verify that the database has not appeared in `core_lib/admin/storage/` or `storage/database/`.

## Main Vulnerability: Direct Database Access

If the web server is not configured to block `.db` files, the database can be downloaded via a direct link:

```
https://yoursite.ru/admin/storage/database/cms.db
```

The database stores all site data: pages, settings, navigation, form submissions, content.

## How to Protect

### Protecting the Admin Panel (First Layer)

The database is located in `/admin/` — all requests to the admin panel go through the CMS with authorization check:

```apache
RewriteRule ^admin(.*)$ index.php [QSA,L]
```

### .htaccess — Blocking .db Files (Second Layer)

**RewriteRule — block any `.db` files:**

```apache
RewriteRule \.db$ - [F,L]
```

This fires before all other rules. Any URL ending in `.db` returns an instant 403.

**Files Directive — banned extensions (Third Layer):**

```apache
<Files ~ "\.(env|json|config\.js|md|gitignore|gitattributes|lock|sql|db)$">
    Order allow,deny
    Deny from all
</Files>
```

### Nginx

For nginx, `.htaccess` does not work. Add the following to your server config:

```nginx
location ~ \.db$ {
    deny all;
    return 403;
}
```

## How to Verify Protection

```bash
curl -I https://yoursite.ru/admin/storage/database/cms.db
```

Should return `403 Forbidden`. If it returns `200 OK` with a content length — the hole is open. Fix it immediately.

Also make sure the file exists **only** at `/admin/storage/database/cms.db`:

```bash
find . -name 'cms.db' -type f
# Should be exactly one file
```

## If .htaccess Does Not Work

On some hosting providers, `AllowOverride` may be disabled in the Apache config. In this case:

1. Contact hosting support
2. Ask them to enable `AllowOverride All` for your site
3. As a temporary measure — add `.db` blocking rules directly to the Apache config
4. The database is under the admin panel — even without `.htaccess`, authorization protection remains

## Regular Auditing

We recommend checking database protection **after every core update** (`core update`) and at least **once a month**:

```bash
# 1. Verify the database is only in the admin panel
find . -name 'cms.db' -type f
# Should only show ./admin/storage/database/cms.db

# 2. Verify the database is not accessible via HTTP
curl -s -o /dev/null -w '%{http_code}' https://yoursite.ru/admin/storage/database/cms.db
# Should be 403

# 3. Check for leftover database files in core_lib (from core updates)
ls core_lib/admin/storage/database/cms.db 2>/dev/null && echo "JUNK — delete!"
```

## Additional Measures

- **Do not publish `cms.db` in git repositories** — add it to `.gitignore`
- **Set up regular database backups**
- **Use HTTPS** — mandatory for forms containing personal data
- **Check after `core update`** — a core update may create duplicate database files
