---
title: AI Knowledge Base — apidcms Documentation
slug: baza-znanij-ai-dokumentaciya
section: ai
order: 310
description: "How the AI assistant uses apidcms documentation as a knowledge base: architecture, caching, DocsFetcher."
---

# 📚 AI Knowledge Base — apidcms Documentation

Since version **v1.3.25** the AI assistant in the admin panel has access to the entire apidcms documentation. When you ask a question, AI sees not only the database structure but also the full text of all articles from the `apidcms-docs` repository.

---

## 🧠 How It Works

On each request to the AI assistant, the system:

1. **Loads the manifest** (`manifest.json`) from the GitHub repo `Dezajnisto/apidcms-docs`
2. **Fetches all articles** (36 Markdown files)
3. **Strips YAML front-matter** — AI receives only the content text
4. **Builds the context** (~131 KB of text, ~32K tokens)
5. **Inserts it into the system prompt** before the database data

Result: AI answers questions about apidcms with knowledge of the official documentation.

### Example Questions AI Can Now Answer

- *"How to create a feedback form with a GDPR consent checkbox?"*
- *"What hooks are available when developing a plugin?"*
- *"How does the external page type work?"*
- *"Explain the template structure: what variables are available in single?"*
- *"How to configure caching?"*
- *"What system settings exist and what are they for?"*

---

## 🗄️ DocsFetcher — Technical Details

### Class `Core\DocsFetcher`

Handles loading and caching of documentation.

```php
$docsFetcher = new \Core\DocsFetcher();
$kb = $docsFetcher->getKnowledgeBase();
// $kb — full documentation text for AI
```

### Caching

| Data | TTL | Location |
|------|-----|----------|
| manifest.json | 1 hour | `admin/storage/docs_cache/manifest.json` |
| Articles (.md) | 6 hours | `admin/storage/docs_cache/<md5-hash>.md` |

The cache directory is created automatically on the first request.

### Data Source

- **Repository:** `github.com/Dezajnisto/apidcms-docs` (branch `master`)
- **Format:** Markdown with YAML front-matter (automatically stripped)
- **Source:** GitHub Raw (`raw.githubusercontent.com`)

### Fault Tolerance

If GitHub is unavailable:
- Stale cache is used — for up to 6 hours
- If no cache exists, AI works **without documentation**, with DB context only
- A load error does not break the AI assistant — it continues working

---

## 🔄 Cache Invalidation

After updating documentation on GitHub, you can forcibly clear the cache:

```php
$fetcher = new \Core\DocsFetcher();
$count = $fetcher->invalidateCache();
// Removed $count cache files
```

On the next request, documentation will be loaded fresh.

In the future, invalidation will be available from the admin panel — a "Clear Documentation Cache" button in the AI settings section.

---

## 📊 Context Size

| Parameter | Value |
|-----------|-------|
| Articles | 36 |
| Sections | 7 |
| Text size | ~131 KB |
| Tokens (approx.) | ~32,000 |
| Model | DeepSeek (128K context) |

32K tokens is ~25% of DeepSeek's context window. Plenty of room remains for conversation and DB data.

---

## 🔮 Future Plans

- **RAG instead of full load** — if the number of articles grows, we'll switch to retrieval-augmented generation: AI will select relevant articles for each specific question
- **Admin panel invalidation** — a cache-clear button in the UI
- **Local mirror** — `git pull` via cron for projects without GitHub access
