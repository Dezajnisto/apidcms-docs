---
title: Changing the Structure
slug: izmenenie-struktury
section: content
order: 150
description: "Changing the structure"
---

# 🔧 Changing the Table Structure

Adding, deleting, or modifying columns of an existing table.

---

## Via Admin Panel

1. Tables → select a table
2. Click **«Structure»** (or SQL query)
3. Use the SQL query:

### Add a Column

```sql
ALTER TABLE products ADD COLUMN discount INTEGER DEFAULT 0;
```

### Delete a Column

SQLite does not directly support `DROP COLUMN`. Workaround:

```sql
-- 1. Create a new table without the column
CREATE TABLE products_new AS SELECT id, title, price FROM products;

-- 2. Drop the old table
DROP TABLE products;

-- 3. Rename the new table
ALTER TABLE products_new RENAME TO products;
```

### Rename a Column

```sql
ALTER TABLE products RENAME COLUMN old_name TO new_name;
```

---

## ⚠️ Caution

- **Backup:** before ALTER TABLE, copy `cms.db`
- **Data:** deleting a column removes all data in it
- **Related templates:** after renaming, update Twig templates
