---
title: Изменение структуры
slug: izmenenie-struktury
section: content
order: 150
description: "Изменение структуры"
---

# 🔧 Изменение структуры таблицы

Добавление, удаление или изменение колонок существующей таблицы.

---

## Через админку

1. Таблицы → выберите таблицу
2. Кнопка **«Структура»** (или SQL-запрос)
3. Используйте SQL-запрос:

### Добавить колонку

```sql
ALTER TABLE products ADD COLUMN discount INTEGER DEFAULT 0;
```

### Удалить колонку

SQLite не поддерживает `DROP COLUMN` напрямую. Обходной путь:

```sql
-- 1. Создать новую таблицу без колонки
CREATE TABLE products_new AS SELECT id, title, price FROM products;

-- 2. Удалить старую
DROP TABLE products;

-- 3. Переименовать новую
ALTER TABLE products_new RENAME TO products;
```

### Переименовать колонку

```sql
ALTER TABLE products RENAME COLUMN old_name TO new_name;
```

---

## ⚠️ Осторожно

- **Бэкап:** перед ALTER TABLE скопируйте `cms.db`
- **Данные:** удаление колонки удаляет все данные в ней
- **Связанные шаблоны:** после переименования обновите Twig-шаблоны
