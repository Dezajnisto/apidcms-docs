---
title: AI Assistant
slug: ai-assistent
section: ai
order: 300
description: "Built-in AI powered by DeepSeek: template generation, table creation, content filling, prompt configuration."
---

# 🤖 AI Assistant

The CMS has a built-in AI assistant powered by DeepSeek. It helps create templates, design tables, fill content, and answer questions.

---

## 🎯 How to Invoke

**Method 1 — Floating Button**
A purple round button with a star ✨ in the bottom-right corner — click it.

**Method 2 — Contextual Buttons**
In sections where AI can help, there are dedicated buttons:
- Template editor — "AI" button
- Table creation — "AI — Create Table" button
- Table view — "AI — Generate Content" button
- Record creation — "AI — Fill Fields" button

---

## 📝 Operation Modes

### 1. Universal Assistant (floating button)
Just ask a question:
- *"How do I make a slider in Twig?"*
- *"What tables are in the DB?"*
- *"Write an SQL query to fetch the last 5 posts"*

### 2. Template Generation (button in the editor)
AI receives context: what tables and fields exist in the DB.
Write:
- *"Make product cards from the products table: photo, name, price, 'Buy' button"*
- *"Add a feedback form block below the page text"*
- *"Turn the list into a 3-column grid"*

After generation, click **"📋 Insert into Editor"**.

### 3. Table Creation (button on the creation page)
Describe in words what data you want to store:
- *"Table for a product catalog: name, description, price, photo, category"*
- *"Table for customer reviews: name, city, rating 1-5, review text, date"*
- *"Table for promotions: title, description, discount percentage, start date, end date"*

AI returns a JSON structure. Click **"📋 Apply Structure"**.

### 4. Content Generation (button on the table page)
Fill the table with data:
- *"Generate 10 blog posts about web development"*
- *"Make 5 customer reviews with different ratings"*
- *"Add 3 products from the 'Interior Design' category"*

After generation, click **"💾 Insert N Records"**.

### 5. Form Filling (button on the record creation page)
AI sees all form fields and picks suitable values:
- *"Create an 'About Us' page with a company description"*
- *"Add a contact: name Ivan, email ivan@mail.ru, phone +7 999 123-45-67"*
- *"Make 3 news items on technology"*

After generation, click **"📋 Fill Form"**.

---

## 🧹 Chat Management

- **"🗑 Clear"** — reset the dialog and start over
- **Auto-clear** — when switching context (template → table), the chat clears automatically
- Close the modal — cross ✕ or click outside

---

## ⚙️ AI Settings

Managed in **Settings → AI (DeepSeek)** (the "AI" tab):

| Parameter | Description |
|-----------|-------------|
| `ai_api_key` | 🔑 DeepSeek API key |
| `ai_model` | Model (default deepseek-chat) |
| `ai_prompt_template` | System prompt for template generation |
| `ai_prompt_table` | Prompt for table structure |
| `ai_prompt_content` | Prompt for content generation |
| `ai_prompt_fill_form` | Prompt for form filling |
| `ai_prompt_assistant` | Prompt for the general assistant |

Prompts can be edited — customize AI behavior for your tasks.

### AI Frontend (the "AI Frontend" tab)

Separate tab for configuring AI on the public part of the site (chat with visitors):

| Parameter | Description |
|-----------|-------------|
| `ai_frontend_use_system` | Use system AI settings or separate ones |
| `ai_frontend_api_key` | Separate API key for the frontend |
| `ai_frontend_model` | Model for the public chat |
| `ai_frontend_prompt` | System prompt for chat with visitors |
| `ai_frontend_personality` | Character/tone of replies (e.g. "friendly consultant") |

---

## 🔒 AI Context Security

The AI assistant **does not see** system and private tables. By default, the following are blocked:

- `system_settings` — site settings
- `users` — users
- `user_tokens` — authorization tokens
- `sqlite_sequence` — SQLite system table

### Table Whitelist

The administrator decides which tables the AI has access to. Setting `ai_public_tables` — a JSON array of allowed tables. Located in the **AI Frontend** tab as a checklist.

By default **no tables are allowed** (security first).

### Row Limit (`ai_sample_limit`)

Limits the number of rows passed to AI as context. Default **50 rows**. A value of `0` — pass only the table structure, no data.

This setting is also in the AI Frontend tab.

---

## 📚 Knowledge Base (since v1.3.25)

The AI assistant has access to the full apidcms documentation from the `apidcms-docs` repository.

**How it works:**
- On each request, AI receives the text of all 36 documentation articles (~131 KB)
- Documentation is cached locally: manifest 1 hour, articles 6 hours
- YAML front-matter is automatically stripped before sending to AI
- If GitHub is unavailable, AI works without documentation (fallback)

**What this gives:**
- AI can answer questions about setup, plugins, templates, forms
- Knows all apidcms capabilities — not just what it sees in the DB
- Example: *"How to create a form with a GDPR checkbox and AJAX submission?"* — AI explains from the documentation

Details: [AI Knowledge Base — apidcms Documentation](/docs/baza-znanij-ai-dokumentaciya)
