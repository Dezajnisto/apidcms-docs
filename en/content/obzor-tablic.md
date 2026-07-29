---
title: Table Overview
slug: obzor-tablic
section: content
order: 120
description: "What is a table, system tables, user tables, how to create them."
---

# 🗄️ Table Overview

## What is a Table?

A table is a structure for storing uniform data. Simply put, it's like a **spreadsheet in Excel**, but in a database.

Each row is one record.
Each column is one field (e.g., title, text, image).

**Example:** «Contacts» table

| id | name | email | message |
|----|------|-------|---------|
| 1 | Ivan | ivan@mail.ru | I want to order a website |
| 2 | Anna | anna@ya.ru | Question about services |

---

## System Tables (mandatory)

During installation, the CMS automatically creates 5 system tables:

| Table | Purpose |
|---------|----------|
| **pages** | Site pages (home, blog, contacts, etc.) |
| **forms** | Contact and application forms |
| **navigation** | Navigation and page types (page, blog, dynamic, ai, etc.) |
| **system_settings** | All site settings (name, AI, email, etc.) |
| **entity_relations** | Relations between tables (M:M — many-to-many) |

Plus optional: **visit_stats** — created automatically when visit statistics are enabled in settings.

---

## User Tables

You can **create your own tables** for any kind of data:
- Product catalog
- Work portfolio
- Customer reviews
- Categories and tags
- Anything you want

**How it works:**
1. Create a table with the desired fields
2. Add records via the admin panel
3. Configure display through the «Navigation» section
4. Write (or choose a ready-made) template

> 💡 **Tip:** A new table = a new data type on your site.
> Create «products» — a catalog appears. Create «reviews» — a reviews section appears.
