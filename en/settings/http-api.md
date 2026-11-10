---
title: HTTP API
slug: http-api
section: settings
order: 350
description: "REST API /api/v1/ for tables and articles: token auth, response formats"
---

# 🔌 HTTP API

The core exposes a REST API under `/api/v1/`. It exists for the cases where
project data has to be read or written outside the browser: a mobile client, a
script, an integration, an external service.

The API works on top of the same tables the admin panel uses. There is no
separate database and no separate data model — the table schema is the contract.

---

## Authorization

Requests without a token are rejected. The token goes in a header:

```
Authorization: Bearer <token>
```

Tokens live in the `user_tokens` table and are created in the admin panel. A
token grants access to the whole project data — it is an integration key, not a
user permission. Treat it like the admin password: not in git, not in frontend JS.

System tables (`system_settings`, `users`, `user_tokens`) are not exposed — same
blacklist as the AI assistant.

---

## Tables

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/v1/tables` | list project tables |
| `GET` | `/api/v1/tables/{table}` | records: `page`, `per_page`, `sort`, `dir` |
| `GET` | `/api/v1/tables/{table}/{id}` | a single record by `id` |
| `POST` | `/api/v1/tables/{table}` | create a record (JSON body) |
| `PUT` | `/api/v1/tables/{table}/{id}` | update a record |
| `DELETE` | `/api/v1/tables/{table}/{id}` | delete a record |
| `GET` | `/api/v1/tables/{table}/schema` | schema: columns, types, indexes |

Example — fetch the latest catalog records:

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://example.com/api/v1/tables/catalog?page=1&per_page=20&sort=id&dir=desc"
```

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
article missing), `validation_error` (missing fields, wrong type),
`internal_error` (database or code failure).

---

## External content cache

When a page is assembled from an external source (page type `external`), the core
caches the response per URL for an hour and returns the **stale** cache when the
source is unreachable. An unavailable external API therefore costs neither a
15-second wait nor a broken page — visitors see data that is up to an hour old.

The cache lives in `storage/cache/` and is cleared with "Clear cache" in the
admin panel.
