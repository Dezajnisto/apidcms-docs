---
title: MCP server — run the site from an AI client
slug: mcp
section: settings
order: 360
description: "Model Context Protocol server on /mcp: tools (schema, records, navigation, docs), resources and connecting from Cursor/[CC]"
---

# 🔌 MCP server

MCP (Model Context Protocol) is the standard by which an AI client (Cursor,
[CC], any MCP-capable agent) learns what a server can do and calls its tools.

apidcms serves MCP at `/mcp`. This is **not a second API**: the tools call the
same core services as `/api/v1` (schema, write validation, audit), so MCP and
HTTP cannot drift apart. The built-in assistant in the admin panel talks to the
same tool catalogue.

---

## Connecting

You need an API token — the same one `/api/v1` uses (requests without it are
rejected).

If your client supports MCP over HTTP:

```json
{
  "mcpServers": {
    "mysite": {
      "type": "http",
      "url": "https://example.com/mcp",
      "headers": {
        "Authorization": "Bearer <token>"
      }
    }
  }
}
```

Transport is Streamable HTTP, JSON-RPC 2.0, stateless: one request, one response,
nothing kept between calls. To check it by hand:

```bash
curl -X POST https://example.com/mcp \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

---

## Tools

| Tool | What it does |
|---|---|
| `schema.get` | the whole project in one call: tables, columns (type, required, allowed values, label, format), foreign keys, many-to-many relations, page map |
| `records.list` | rows of a table: `limit`, `offset`, `sort`, `order`, `search` |
| `records.get` | one row by `id` |
| `records.create` | insert a row (validated like the API) |
| `records.update` | update a row (returns the changed columns) |
| `records.delete` | delete a row |
| `navigation.get` | page map: which table feeds which URL with which template |
| `relations.list` | many-to-many relations, optionally for one table |
| `relations.add` | link a record to categories/tags: `source_table`, `source_id`, `relation_name`, `target_ids[]` |
| `relations.remove` | unlink (empty `target_ids` unlinks all) |
| `audit.list` | the write log: method, table, row, diff |
| `docs.list` | documentation table of contents (cheap) |
| `docs.get` | one documentation article by slug |
| `docs.search` | find an article by words from the question |

Every mutating tool accepts `dry_run: true` — see what would change and which
pages it would affect without writing anything.

A refused write comes back as `isError: true` with a field list:

```json
{
  "error": "validation_failed",
  "table": "catalog",
  "action": "insert",
  "fields": [
    {"column": "price", "rule": "type", "got": "1500 RUB", "expected": "integer (INTEGER)"}
  ]
}
```

The model reads that and fixes itself — which is why write validation was built
before MCP, not after.

---

## Resources

Besides tools, MCP exposes **resources** — data a client can read by URI without
calling a tool:

| URI | Content |
|---|---|
| `apidcms://schema` | project schema (JSON) |
| `apidcms://navigation` | page map (JSON) |
| `apidcms://docs/toc` | documentation contents (JSON) |
| `apidcms://docs/{slug}` | one documentation article (Markdown) |

Documentation as a resource is not decoration: a client can keep the table of
contents at hand and pull one article when it needs it instead of loading the
whole library into the context.

---

## What it changes in practice

An agent with a token, knowing only the site address, gets the full picture in
two calls: `schema.get` → which tables exist, what each column means, which pages
exist and which table feeds which URL. Then `records.*` to read and write,
`docs.*` to answer questions about the CMS itself.

It also closes the "I wrote it, why is the site unchanged?" gap: `dry_run` and the
tool result return `affected_urls` — the pages assembled from that table.

---

## Security

- access only with a Bearer token, the same one `/api/v1` uses;
- internal tables (`system_*`, `users`, `user_tokens`, `entity_relations`,
  `visit_stats`, `table_meta`, `field_meta`, `api_idempotency`) are not reachable
  through tools;
- every write through MCP lands in the audit log marked `actor: mcp`;
- tokens are scoped: a `read` token cannot call any mutating tool — such a call
  returns `isError: true` with the code `insufficient_scope`. Give a read-only
  agent a `read` token in **Settings → API**.
