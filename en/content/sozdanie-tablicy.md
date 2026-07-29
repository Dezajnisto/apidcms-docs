---
title: Creating a Table
slug: sozdanie-tablicy
section: content
order: 130
description: "Creating a table"
---

# 🗄️ Creating a Table

Tables are the foundation of data in apidcms. They are created without SQL via the admin panel.

---

## Via Admin Panel

1. Admin Panel → Tables → **«Create Table»**
2. Specify a name (latin, lowercase, underscores): `products`, `reviews`
3. Add fields:

### Field Types

| Type | SQLite | Usage Example |
|-----|--------|---------------------|
| TEXT | TEXT | Titles, names, descriptions |
| INTEGER | INTEGER | Prices, ratings, quantities |
| REAL | REAL | Decimal numbers |
| DATETIME | DATETIME | Dates |
| BOOLEAN | INTEGER (0/1) | Yes/No |

4. Click «Create»

---

## Via AI

The **«AI — Create Table»** button on the creation page. Describe in words:

> «Table for a product catalog: name, description, price, photo, category»

AI returns a ready-made structure. Click «Apply».

---

## Required Fields for Dynamic Pages

If the table will be used with `page_type = 'dynamic'`, add:

| Field | Purpose |
|------|---------|
| `slug` | TEXT — record URL address |
| `status` | TEXT DEFAULT 'active' — published/draft filter |
| `sort_order` | INTEGER — sort order |

---

## Editing the Structure

To add or delete a column — use [ALTER TABLE](https://www.sqlite.org/lang_altertable.html) via the admin panel (SQL query) or the «Edit Structure» interface.
