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
| `ai_prompt_assistant` | Extra instructions for the general assistant (appended to its system prompt) |
| `ai_assistant_max_steps` | How many tool rounds are allowed (1–10, default 6) |

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
- `entity_relations` — internal many-to-many links (read through the `relations.list` tool)
- `visit_stats` — visit statistics
- `table_meta` / `field_meta` — metadata (served through `schema.get`)
- `api_idempotency` — API retry cache
- `sqlite_sequence` — SQLite system table

### Table Whitelist

The administrator decides which tables the AI has access to. Setting `ai_public_tables` — a JSON array of allowed tables. Located in the **AI Frontend** tab as a checklist.

By default **no tables are allowed** (security first).

### Row Limit (`ai_sample_limit`)

Limits the number of rows passed to AI as context. Default **50 rows**. A value of `0` — pass only the table structure, no data.

This setting is also in the AI Frontend tab.

### Context Size (v2.2.0)

Settings that limit the amount of templates and CSS sent to the AI assistant. Located in the **AI** tab (Settings → AI → Context Size):

| Parameter | Description | Default |
|-----------|-------------|:---:|
| `ai_template_max_size` | Max bytes per .twig file | 20000 |
| `ai_template_total_max` | Total limit for all templates | 60000 |
| `ai_css_max_size` | Max bytes for custom.css | 20000 |

`0` or empty means the **built-in limit** (the values above). `-1` means
deliberately unlimited. Before 2.8.2 `0` meant "unlimited", and that was not the
owner's choice but the shipped default: the prompt grew with the site silently.

#### Which templates are sent

Templates come **from the site's active theme**; the core only fills in what the
theme does not have at all. Order: `base.html.twig` first, then the other
root-level theme templates alphabetically, then root-level core templates, then
partials from subdirectories. Names differ per site (`catalog.html.twig`,
`catalog_single.html.twig` and so on), so the list is not fixed.

Before 2.8.2 the template and CSS generators ran **without any templates at
all**: the code looked for them in old-structure paths that do not exist in the
project and silently got an empty array.

#### What goes out right now

The same AI tab has a **"What goes to the model now"** block: active theme, the
number of templates and their size, the CSS size (flagged when cut by the limit),
and the total in bytes and tokens. Prompt size should not be invisible.

---

## 📚 Knowledge base and tools (since 2.5.0)

The assistant does not answer from memory or from a documentation dump — it
answers from facts: it has **tools**, the same ones the
[MCP server](../settings/mcp.md) exposes.

| Tool | When it is used |
|---|---|
| `schema.get` | questions about the site: which tables, columns, pages exist |
| `records.list` / `records.get` | questions about actual data ("what is in the catalog?") |
| `records.create` / `update` / `delete` | "add a row", "change the price" — with field validation |
| `navigation.get` | "which table is shown on this page?" |
| `docs.search` / `docs.list` / `docs.get` | questions about CMS capabilities: it finds an article and reads it |

**What it changes:**

- the whole library (~163 KB) no longer goes into the prompt — only a compact
  description of what can be asked, with articles pulled one at a time;
- the assistant sees the real schema and real rows instead of a summary;
- it can **do**, not only advise: add a row, fix a field, preview the change
  (`dry_run`).

The step limit and the switch are in the settings (below).

**What it costs.** The assistant response carries `steps` and `usage`
(`prompt_tokens`, `completion_tokens`, `total_tokens`, summed over all steps of
one question). Measured on a site with 36 documentation articles: a data question
takes 2 steps and ~6 000 prompt tokens, a documentation question 3 steps and
~5 900. The legacy mode sent ~54 000 tokens with every message regardless of the
question.

Details: [AI knowledge base — how the assistant reads the docs](/docs/baza-znanij-ai-dokumentaciya)
