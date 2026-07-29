---
title: Installation and Requirements
slug: ustanovka-i-trebovaniya
section: quickstart
order: 30
description: "Installation and Requirements"
---

# 🚀 Installation and Requirements

The CMS runs on regular PHP hosting. No database is needed — it uses an SQLite file.

---

## System Requirements

| Requirements | Minimum |
|-----------|---------|
| **PHP** | version 8.1 or higher |
| **PHP Extensions** | `sqlite3`, `curl`, `mbstring`, `json`, `gd`, `openssl`, `fileinfo`, `zip`, `xml` |
| **Composer** | PHP dependency manager |
| **Disk** | about 10 MB |

> The installer will check everything needed before starting.

---

## Installation

### 1. Create a site in the hosting panel

Create a site for your domain. The hosting panel will create the `www/` folder and configure PHP.

### 2. Upload the CMS

Open the terminal. Replace `my-site.ru` with your project folder:

```bash
cd ~/my-site.ru
```

All subsequent commands are copied as-is:

```bash
git clone https://github.com/Dezajnisto/apidcms.git _clone
```

```bash
cp -r _clone/www/* www/ && rm -rf _clone
```

### 3. Run the installer

Open `https://your-site.ru/install.php` in your browser and follow the on-screen instructions. The installer will check requirements, install dependencies, and create the database.

### 4. Open the site

Done. The admin panel is available at `/admin`. Login and password are the ones you set during installation.

> ⚠️ After installation, remove the installer:
> ```bash
> rm www/install.php
> ```

---

## Without Git

```bash
cd ~/my-site.ru
wget https://github.com/Dezajnisto/apidcms/archive/refs/heads/main.zip
unzip main.zip && cp -r apidcms-main/www/* www/ && rm -rf apidcms-main main.zip
```

Then open `https://your-site.ru/install.php` in your browser.

---

## After Installation

1. **Change the password** (if needed). Admin panel -> Settings.
2. **Configure AI.** If needed — provide an API key in Settings.
3. **Remove install.php** (see above).

---

## Common Issues

**"Composer not found"** — download Composer manually:

```bash
cd www
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php composer.phar install --no-dev
```

**Site not loading (502/503)** — make sure the site is created in the hosting panel and PHP is enabled.

**"Extension pdo_sqlite not found"** — ask hosting support to enable `pdo_sqlite` and `sqlite3`.
