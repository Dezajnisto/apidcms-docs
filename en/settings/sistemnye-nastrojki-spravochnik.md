---
title: System Settings Reference
slug: sistemnye-nastrojki-spravochnik
section: settings
order: 320
description: "System Settings Reference"
---

# ⚙️ System Settings Reference

All settings are stored in the `system_settings` table. Managed through the admin panel: Settings → three tabs.

---

## Main Site Settings

| Key | Default | Description |
|-----|---------|-------------|
| `site_title` | `My site` | Site title |
| `site_description` | `Site description` | Meta description |
| `site_email` | — | Contact email |
| `site_favicon` | — | Path to favicon |
| `posts_per_page` | `10` | Posts per page on the blog |
| `maintenance_mode` | `0` | `1` — site in maintenance mode |

---

## AI (DeepSeek)

| Key | Description |
|-----|-------------|
| `ai_api_key` | DeepSeek API key |
| `ai_model` | Model (default `deepseek-chat`) |
| `ai_prompt_template` | System prompt for template generation |
| `ai_prompt_table` | Prompt for table structure creation |
| `ai_prompt_content` | Prompt for content generation |
| `ai_prompt_fill_form` | Prompt for form filling |
| `ai_prompt_assistant` | Prompt for the universal assistant |

---

## AI Frontend (ai-pages)

| Key | Default | Description |
|-----|---------|-------------|
| `ai_frontend_use_system` | `1` | Use system key and model |
| `ai_frontend_api_key` | — | Separate key for the frontend |
| `ai_frontend_model` | — | Separate model for the frontend |
| `ai_frontend_prompt` | (see below) | System prompt for AI chat |
| `ai_frontend_personality` | — | Assistant name/role |
| `ai_public_tables` | — | JSON array: tables accessible to AI |
| `ai_sample_limit` | `50` | Max rows from a table for context |

---

## Statistics

| Key | Default | Description |
|-----|---------|-------------|
| `stats_enabled` | `0` | `1` — collection enabled |
| `stats_retention_days` | `90` | How many days to keep data |

---

## Email

| Key | Default | Description |
|-----|---------|-------------|
| `email_driver` | `api` | `api` or `smtp` |
| `email_api_provider` | — | Provider (e.g. `smtp.bz`) |
| `email_api_key` | — | API key |
| `email_api_endpoint` | — | Endpoint URL |
| `email_smtp_host` | — | SMTP server |
| `email_smtp_port` | `587` | Port |
| `email_smtp_username` | — | Login |
| `email_smtp_password` | — | Password |
| `email_smtp_encryption` | `tls` | `tls` / `ssl` |
| `email_from_email` | — | Sender email |
| `email_from_name` | `APIDCMS` | Sender name |
