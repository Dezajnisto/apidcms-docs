---
title: CRUD — Records
slug: crud-zapisi
section: content
order: 140
description: "CRUD — records, CSV import and export"
---

# ✏️ CRUD — Adding, Editing, Deleting Records

---

## Adding a Record

1. Admin Panel → Tables → select a table
2. Click **«Add Record»**
3. Fill in the fields
4. Click «Save»

## Editing

1. In the table, click on a row
2. Change the desired fields
3. Click the 💾 (save) button in the row

## Deleting

1. In the table, hover over a row
2. Click the 🗑 (trash) button
3. Confirm

## Bulk Actions

- Select multiple rows (checkboxes on the left)
- In the top panel: «Delete Selected»

## Search and Sort

- **Search** — field at the top of the table, searches across all text columns
- **Sort** — click on a column header (▲/▼)

---

## Via AI

The **«AI — Generate Content»** button on the table page:

> «Generate 10 blog posts about web development»

AI creates the records. Click «💾 Insert N records».

---

## CSV Import

On the table page — the **«Import CSV»** button.

1. Click «Import CSV»
2. Select a CSV file on your computer
3. The system automatically detects headers and maps them to table columns
4. Review the mapping and select the import mode
5. Click «Import»

### Import Modes

| Mode | Behavior |
|-------|-----------|
| **Add all rows** | Each CSV row is added as a new record. On slug conflict, a unique slug is automatically created (`slug-1`, `slug-2`...) |
| **Skip duplicates** | Rows where the key column matches an existing record are skipped |
| **Update existing** | Matching records are updated with CSV values. Non-matching ones are added as new records |

### Key Column

For «Skip duplicates» and «Update existing» modes, select the column used to find matches. Default is `id`.

**Scenario:** exported a table to CSV, edited the data in Excel, uploaded back in «Update existing» mode — rows are updated by id.

### Tips

- The first CSV row must contain column names
- Non-matching columns can be skipped or mapped manually
- Empty values in optional columns become NULL
- `id` and `created_at` columns are filled automatically (if not specified in CSV)

---

## CSV Export

On the table page — the **«Export CSV»** button. Downloads all data respecting the current search and sort. The file opens correctly in Excel (UTF-8 with BOM).
