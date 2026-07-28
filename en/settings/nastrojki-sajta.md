---
title: Site Settings
slug: nastrojki-sajta
section: settings
order: 310
description: "General settings, AI (DeepSeek), system parameters: email, integrations."
---

# Site Settings

The Settings page is divided into tabs:

---

## General Tab

### Site Name
Used in `<title>` tag and page headers.

### Site Description
SEO meta-description for search engines.

### Site Email
Contact email of the administrator.

### Posts Per Page
Number of items per page in lists (blogs, catalogs).

### Favicon
Path to the favicon file, e.g. `/storage/uploads/favicon.ico`.

### Maintenance Mode
When enabled, visitors see a maintenance page instead of the site.

### Admin Language
Interface language for the admin panel. Currently: Russian, English.

---

## AI Tab

API settings for the built-in AI assistant (DeepSeek):

- `ai_api_key` — API key
- `ai_model` — model name (default: `deepseek-chat`)
- `ai_prompt_template` — system prompt for template generation
- `ai_prompt_table` — system prompt for table creation
- `ai_prompt_content` — system prompt for content generation
- `ai_prompt_assistant` — system prompt for AI assistant

---

## AI Frontend Tab

Settings for AI-powered pages on the frontend (type `ai`):

- Can use system AI settings or a separate API key/model
- System prompt and personality instructions
- Public tables accessible to AI context
- Row limit for context data

---

## Statistics Tab

Built-in visit tracking:

- **Enable/disable** statistics collection
- **Retention period:** 30, 60, 90, 180, or 365 days
- Auto-cleanup runs approximately every 100 requests

---

## System Tab

All other system settings: email driver, SMTP, API endpoints.

Each setting can be edited inline or deleted. New settings can be added at the bottom.
