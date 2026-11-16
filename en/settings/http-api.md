---
title: HTTP API
slug: http-api
section: settings
order: 350
description: "REST API /api/v1/: project schema, navigation, write validation, dry_run, audit log and articles"
---

# 🔌 HTTP API

The core exposes a REST API under `/api/v1/`. It exists for the cases where
project data has to be read or written outside the browser: a mobile client, a
script, an integration, an external service or an AI agent.

The API works on top of the same tables the admin panel uses. There is no
separate database and no separate data model — the table schema is the contract.

> ⚠️ **Status: preview.** The `/api/v1` contract still changes between releases,
> and responses do not promise completeness: writes are not fully validated, some
> paths are only planned and answer `501 not_implemented`. Every response carries
> the `X-apidcms-api: v1-preview` header. Before you let an external agent write,
> read the "What is not there yet" section.

---

## Authorization

Requests without a token are rejected. The token goes in a header:

```
Authorization: Bearer <token>
```

The token is an integration key, not a user permission: it grants access to all
project data. Treat it like the admin password — not in git, not in frontend JS.
It is stored in the `api_key` system setting; there is no dedicated token screen
yet (it comes together with scopes in a later release).

Only two things are public: `GET /api/v1` and `GET /api/v1/system/info` — enough
for an agent to learn what this site is.

System tables (`system_settings`, `users`, `user_tokens`, `entity_relations`,
`visit_stats`, `table_meta`, `field_meta`, `plugin_*`) are not exposed — same
blacklist as the AI assistant.

---

## Start with one request

```bash
curl -H "Authorization: Bearer $TOKEN" https://example.com/api/v1/schema
```

`GET /api/v1/schema` returns the whole project in one payload: tables, columns
(type, required, default, allowed values), descriptions, foreign keys,
many-to-many relations and the page graph. That is cheaper and more accurate than
walking `/tables` and then every `/schema`.

If you do not have a token yet, start with `GET /api/v1` — it is public and lists
the capabilities of the API with the status of each.

### Project description

| Method | Path | Returns |
|---|---|---|
| `GET` | `/api/v1` | `{cms, version, status: "preview", capabilities:[...]}` |
| `GET` | `/api/v1/system/info` | core version, PHP, site title |
| `GET` | `/api/v1/schema` | all tables + relations + page graph |
| `GET` | `/api/v1/navigation` | page graph: `slug`, `url`, `type`, `template`, `source_table`, `filters` |
| `GET` | `/api/v1/relations` | many-to-many relations from `entity_relations` |
| `GET` | `/api/v1/relations/{table}` | the same for one table |
| `GET` | `/api/v1/audit` | what was written through the API: `limit`, `table`, `id` |

A column in `/schema` looks like this:

```json
{
  "name": "price",
  "type": "INTEGER",
  "pk": false,
  "notnull": true,
  "default": null,
  "enum": null,
  "label": "Price, RUB",
  "description": "Price in rubles, VAT excluded",
  "format": "integer",
  "example": "1500"
}
```

Types and limits come from the database itself (`PRAGMA table_info`, `CHECK`
constraints, `PRAGMA foreign_key_list`). Labels, purpose and allowed values come
from project metadata (see below). An undescribed field is `null` — that means
"not described", not "irrelevant": values are never invented.

---

## Table semantics

The shape of a table is visible in the database, its meaning is not: `INTEGER`
does not say the number is rubles, and `TEXT` does not say the value is `active`
rather than `published`.

So tables and columns have a description, edited in the admin panel:

> Tables → **Structure** → the **"Semantics"** button

For a table: label, purpose, longer description. For a column: label, note,
format (`integer`, `money`, `date`, `url`, …), an example and the list of allowed
values. It is stored in the `table_meta` and `field_meta` tables and served
through `/api/v1/schema` and `/api/v1/tables/{table}/schema` — the only part of
the description that cannot be recovered from code.

---

## Page graph: navigation.page_config

The "table → URL → template" link lives in the `navigation.page_config` column
(JSON):

```json
{
  "source_table": "posts",
  "template": "blog",
  "items_per_page": 10,
  "filters": {"category": "news"},
  "get_filters": {"tag": "tags"},
  "sort": {"field": "created_at", "order": "DESC"}
}
```

It is usually filled in by the admin panel when a page is created — and this is
what makes a site a site: without `page_config` data does not become pages.
`GET /api/v1/navigation` returns it already resolved, without JSON strings and
without guessing:

```json
{
  "slug": "blog",
  "url": "/blog",
  "title": "Blog",
  "type": "dynamic",
  "template": "blog",
  "source_table": "posts",
  "filters": {"category": "news"},
  "items_per_page": 10,
  "page_config_error": null
}
```

If the JSON in `page_config` is broken, the page is still listed and
`page_config_error` explains why.

---

## Tables

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/v1/tables` | list project tables |
| `GET` | `/api/v1/tables/{table}` | records: `limit`, `offset`, `sort`, `order`, `search` |
| `GET` | `/api/v1/tables/{table}/{id}` | a single record by `id` |
| `POST` | `/api/v1/tables/{table}` | create a record (JSON body) |
| `PUT` | `/api/v1/tables/{table}/{id}` | update a record |
| `DELETE` | `/api/v1/tables/{table}/{id}` | delete a record |
| `GET` | `/api/v1/tables/{table}/schema` | table schema: columns, relations, metadata |
| `GET` | `/api/v1/tables/{table}/json-schema` | JSON Schema of the table: the write rules for client-side validation |

Example — fetch the latest catalog records:

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/tables/catalog?limit=20&offset=0&sort=id&order=desc"
```

---

## Writes: what is checked

A write is validated before it reaches the database. The rules come from the
table itself (`PRAGMA`) and from the metadata, so the schema is the contract:

| Rule | Source |
|---|---|
| `type` | column affinity: `INTEGER`, `REAL`, `TEXT`, `BLOB`; `"1500"` is accepted for INTEGER, `"1500 RUB"` is not |
| `required` / `notnull` | `NOT NULL` without a default |
| `enum` | `field_meta.enum`, otherwise `CHECK (col IN (...))` from the DDL |
| `foreign_key` | `PRAGMA foreign_key_list`: the referenced row must exist |
| `format` | `field_meta.format`: `integer`, `money`, `number`, `boolean`, `date`, `datetime`, `url`, `email`, `slug` |
| `max_length` | `VARCHAR(n)` |
| `unknown_column` | fields the table does not have |

A refusal is `422` with a machine-readable list. Nothing is written, and the
error names the column, the rule, what arrived and what was expected:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"title":"Sneakers","price":"1500 RUB"}' \
  https://example.com/api/v1/tables/catalog
```

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Record rejected: check the fields",
    "table": "catalog",
    "action": "insert",
    "fields": [
      {"column": "price", "rule": "type", "got": "1500 RUB", "expected": "integer (INTEGER)"}
    ],
    "hint": "Rules come from the table schema: GET /api/v1/tables/catalog/json-schema"
  }
}
```

Constraints the database refuses on its own also come back as codes instead of
exception text: `conflict` (`409`, `UNIQUE`), `foreign_key_violation` (`409`),
`notnull_violation` (`422`), `database_error` (`422`).

---

## "What would happen?": `dry_run`

Every write and delete accepts `?dry_run=1`: validation runs, nothing is stored,
and the response says what would change and which pages show it.

```bash
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/tables/catalog/42?dry_run=1"
```

```json
{
  "dry_run": true,
  "table": "catalog",
  "id": 42,
  "action": "delete",
  "would_change": true,
  "validation": [],
  "row": {"id": 42, "title": "Sneakers"},
  "affected_urls": [
    {"slug": "catalog", "url": "/catalog", "title": "Catalog", "template": "catalog", "filters": {}}
  ]
}
```

`affected_urls` answers "I wrote it, why is the site unchanged?": those are the
pages assembled from this table (via `navigation.page_config`).

With `dry_run=1` bad data does not produce a `422`: the answer is `200` with
`"would_change": false` and a `validation` list — so one request answers both "is
it allowed" and "what would change".

---

## Retries: `Idempotency-Key`

Agents retry — on a timeout, on a `502` from a proxy, after a client crash. So a
retry does not create a second row, send a key:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -H "Idempotency-Key: 6f1c2a90-4b09-4b9a-a3d4-0c1f7f6b1e42" \
  -d '{"title":"Sneakers"}' \
  https://example.com/api/v1/tables/catalog
```

A repeat with the same key returns the first response (`201` and the same `id`)
plus an `Idempotent-Replay: true` header. Keys live for 24 hours. Only
successful responses are stored: a failed attempt stays retryable.

---

## Log: what the agent did

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/audit?limit=20&table=catalog"
```

Every write through the API lands in `system_audit`: method, path, table,
`row_id`, IP and the diff (`before`, `after`, `changed`). Entries older than 90
days are pruned. Filters: `limit` (up to 500), `table`, and a row `id`.

---

## Articles

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/v1/articles` | list articles |
| `GET` | `/api/v1/articles/{slug}` | a single article |
| `POST` | `/api/v1/articles` | create an article |
| `PUT` | `/api/v1/articles/{slug}` | update an article |
| `DELETE` | `/api/v1/articles/{slug}` | delete an article |

---

## Response format

A successful response is a JSON object or an array of objects. Errors always look
the same:

```json
{
  "error": {
    "code": "not_found",
    "message": "Table 'nope' does not exist"
  }
}
```

Codes: `unauthorized` (missing or invalid token), `not_found` (table, record or
article missing), `bad_request` (bad path or body), `validation_failed` (a field
failed a rule — see the `fields` list), `conflict` / `foreign_key_violation` /
`notnull_violation` / `database_error` (the database refused the row), `forbidden`
(system table), `not_implemented` (the path is planned but not built yet — it
answers `501`, not `404`, so you do not hunt for a typo), `internal_error` (code
failure).

---

## What is not there yet

Planned and marked as `planned` in `capabilities`:

- **scope tokens** (`read` / `content` / `full`) and a token screen: today there
  is one token that grants access to everything;
- **templates** (`/api/v1/templates`) — without them an agent cannot finish the
  "add a section" task;
- **MCP** (`/mcp`) — the same API exposed as agent tools.

The API token lives in the `api_key` system setting and is still set by hand —
there is no screen for it yet.

---

## External content cache

When a page is assembled from an external source (page type `external`), the core
caches the response per URL for an hour and returns the **stale** cache when the
source is unreachable. An unavailable external API therefore costs neither a
15-second wait nor a broken page — visitors see data that is up to an hour old.

The cache lives in `storage/cache/` and is cleared with "Clear cache" in the
admin panel.
