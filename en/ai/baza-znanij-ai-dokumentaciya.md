---
title: AI knowledge base — how the assistant reads the docs
slug: baza-znanij-ai-dokumentaciya
section: ai
order: 310
description: "The AI assistant and MCP pull documentation on demand through tools (DocsFetcher/DocIndex) instead of loading the whole library into every prompt."
---

# 📚 AI knowledge base — how the assistant reads the docs

Starting with **2.5.0** the AI assistant in the admin panel **no longer loads the
entire documentation into every request**. It gets a compact description of what
it can ask for and decides itself which article or table to open.

What changed is not only token spend but behaviour: the assistant used to give
advice only, now it has tools — it reads the real schema, the real rows, and can
change them.

---

## 🧠 How it works

The assistant has a set of tools (`Core\AgentTools`), the same ones the
[MCP server](../settings/mcp.md) exposes:

| Tool | Returns |
|---|---|
| `schema.get` | tables, columns, allowed values, relations, page map |
| `records.list` / `records.get` | real rows of a table |
| `records.create` / `update` / `delete` | data changes with field validation |
| `navigation.get` | which table feeds which URL with which template |
| `docs.list` / `docs.get` / `docs.search` | documentation: contents, one article, search |

The answer loop:

1. The user asks a question.
2. The model decides what it is missing and calls a tool (`docs.search` →
   `docs.get` for CMS questions, `schema.get` for site data).
3. The result comes back as one `role: tool` message.
4. The model answers. Usually 1–3 steps.

The step limit is `ai_assistant_max_steps` (default 6), and each result is
truncated to 16 KB so one big table cannot swallow the context.

---

## 📉 What it costs

| | Before 2.5.0 | Now |
|---|---|---|
| Documentation in the prompt | the whole library: ~163 KB of text | contents: ~10 KB, articles on demand |
| Site table dump | always | only if the model asks |
| Order of magnitude | ~55–60k tokens **per message** | ~3k in the first request + 1–2 articles of ~1k |

"How do I set up caching" now costs one contents search and one article instead
of the whole library. A question about site data costs one `schema.get` and
nothing at all if the model already called it.

---

## 🗄️ Technical details

### `Core\DocIndex` — contents and one article

```php
$index = new \Core\DocIndex();
$toc = $index->toc();               // slug, title, section, description
$doc = $index->get('http-api');     // one article, front-matter stripped
$hits = $index->searchDeep('cache');// search: titles first, then bodies
```

### `Core\DocsFetcher` — fetching and caching

```php
$fetcher = new \Core\DocsFetcher();
$manifest = $fetcher->getManifest();          // manifest
$content  = $fetcher->getDocContent($url);    // one article
```

| Data | TTL | Location |
|---|---|---|
| manifest.json | 1 hour | `storage/cache/docs/manifest.json` |
| Articles (.md) | 6 hours | `storage/cache/docs/<md5>.md` |

The cache lives in the project's `storage/`, not inside the core: it used to be
written to `core/admin/storage/docs_cache` and survived core updates as junk. It
is cleared with "Clear cache" in the admin panel.

The source is `raw.githubusercontent.com/Dezajnisto/apidcms-docs` (branch
`master`, language from the `admin_language` setting). If GitHub is unreachable,
the stale cache is used; without a cache the assistant works without
documentation but still with the site data tools.

---

## ⚙️ Settings

| Key | Default | Purpose |
|---|---|---|
| `ai_assistant_tools` | `1` | `0` — bring back the old mode: the whole library and a table dump in the system prompt |
| `ai_assistant_max_steps` | `6` | how many tool rounds are allowed (1–10) |

Both are edited in the admin panel: Settings → AI.

---

## 🔮 Next

- a dedicated "reset documentation cache" button (today it is the generic "Clear cache");
- a local documentation mirror for projects that cannot reach GitHub;
- template tools (`templates.read/write`) so the assistant finishes a task instead
  of stopping at the data.
