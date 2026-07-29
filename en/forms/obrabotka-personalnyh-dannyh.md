---
title: Processing Personal Data
slug: obrabotka-personalnyh-dannyh
section: forms
order: 220
description: "PD consent checkbox, enabling methods, storing logs, legal requirements."
---

# 📜 Processing Personal Data and Consent

> How the personal data processing consent works on forms, where to view logs, how to enable it for your project.

---

## 🔄 How It Works

The personal data processing consent checkbox is **not added automatically** to all forms. It must be explicitly enabled using one of two methods:

### Method 1. `pd_consent` Column in the Data Table

If the table linked to the form has a `pd_consent` column (INTEGER, DEFAULT 0), the checkbox appears automatically. The value is saved in the database (1 — consent obtained, 0 — no).

**Example for prostostihi:**
```sql
ALTER TABLE contacts ADD COLUMN pd_consent INTEGER DEFAULT 0;
```
→ The checkbox appeared, data is written to the `contacts` table.

### Method 2. `consent_enabled` Option in form_config

If the column does not exist in the table but the checkbox is needed — add it to `form_config`:

```json
{
  "consent_enabled": true,
  "consent_required": true,
  ...
}
```

The checkbox will appear, data will be **stored in the log only** (not in the database).

**Parameters:**
| Field | Default | Description |
|-------|---------|-------------|
| `consent_enabled` | `false` | Show the checkbox |
| `consent_required` | `true` | Make the checkbox required |

---

## ✅ What Needs to Be Done on a New Project

### 1. Create a Privacy Policy Page

1. Create a page in the admin panel or insert directly into the `pages` table:
   - **slug:** `confidential`
   - **title:** "Privacy Policy"
   - **status:** `active`

2. The `page.html.twig` template already contains styles for legal documents.

3. **Standard template** — in the project files: `apidcms/confidential_template.md`

### 2. Create a Public Offer Page

slug `oferta`. Standard template: `apidcms/oferta_template.md`

### 3. Submit Notification to Roskomnadzor

If the site collects personal data → submit a notification via **Gosuslugi** to be included in the PD operators registry. Fine for non-compliance — up to 300,000 ₽.

---

## 📋 Storing Consent Evidence

On every form submission with the checkbox checked, the system writes a line to the log:

```
admin/storage/logs/pd_consent.log
```

**Line format:**
```
2026-06-19 23:15:15 | 192.168.1.1 | contacts | id=42 | Mozilla/5.0...
```

**What is recorded:**
- Date and time
- Sender's IP address
- Table where the data was saved
- Record ID in the table
- User-Agent

---

## 🔍 How to Check

Via SSH:
```bash
tail -f www/admin/storage/logs/pd_consent.log
```

Or via the admin file manager → `storage/logs/pd_consent.log`