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
- `/account/forgot` — password recovery (magic-link)
- `/account/reset?token=***` — magic-link login
- `/profile` — profile
- `/account/change-password` — change password
- `/logout` — logout

**Features:**
- Remember-me (30-day cookie)
- Magic-link: passwordless login, one-time token
- Password change for authenticated users
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
  "tables": ["users", "user_tokens"]
}
```

After adding the `tables` field, plugin tables will appear in the "Plugins" section on the admin dashboard.

---

##
