---
title: External Sources (type external)
slug: external-page-type
section: content
order: 110
---

## external — Data from External JSON APIs

The `external` page type loads data from an external JSON source (API, GitHub, NocoDB) instead of a local SQLite table.

### How It Works

1. When a page is accessed, `FrontController` creates an `ExternalPageLoader`
2. The loader checks the cache (if `cache_ttl > 0`)
3. If no cache or it is stale — an HTTP request is made to `source_url`
4. The response is parsed as JSON, `json_path` extracts the target array
5. Data is passed to the Twig template as the `items` variable

### Configuration (navigation.page_config, JSON)

```json
{
  "source_url": "https://api.example.com/data.json",
  "json_path": "results",
  "cache_ttl": 3600,
  "method": "GET",
  "headers": {
    "Authorization": "Bearer {{ token }}"
  },
  "template": "catalog"
}
```

| Field | Type | Default | Description |
|------|-----|-------------|----------|
| `source_url` | string | — | **Required.** External JSON URL |
| `json_path` | string | `""` | Dot-separated path: `"data.items"` → `$data['data']['items']`. Empty = entire response |
| `cache_ttl` | int | `0` | Cache TTL in seconds. `0` = live mode. `3600` = one hour |
| `method` | string | `"GET"` | HTTP method: GET, POST, PUT, PATCH |
| `headers` | object | `{}` | Additional headers. Supports `{{ token }}` and `{{ setting.KEY }}` |
| `template` | string | `"default"` | Twig template name (without `.html.twig`). `"default"` → `external.html.twig` |

### Operation Modes

| Mode | cache_ttl | Behavior |
|-------|-----------|-----------|
| Cacheable | > 0 | First request → API → file cache. Subsequent requests → from cache until TTL expires |
| Live | 0 | Each request → API. URL query parameters are forwarded to source_url |

### Cache

- Files: `/admin/views/cache/external_{md5(url+path)}.json`
- Format: `{"ts": 1690000000, "url": "...", "data": {...}}`
- Auto-cleanup by TTL; manual — via Admin Panel → Clear Cache → External

### Placeholders in Headers

- `{{ token }}` → system setting `external_default_token`
- `{{ setting.KEY }}` → any system setting by key

### Template Variables

| Variable | Type | Description |
|------------|-----|----------|
| `items` | array | Data array (from `json_path` or the entire response) |
| `raw` | array | Full JSON response |
| `from_cache` | bool | `true` if from cache |
| `page_config` | object | Page configuration |
| `source_url` | string | Source URL |
| `nav_item` | object | Navigation item |

### Examples

**Plugin Catalog from GitHub (cacheable):**
```json
{
  "source_url": "https://raw.githubusercontent.com/Dezajnisto/apidcms-plugins/main/plugins.json",
  "json_path": "plugins",
  "cache_ttl": 3600,
  "template": "plugins"
}
```

**Changelog from GitHub Releases (cacheable):**
```json
{
  "source_url": "https://api.github.com/repos/Dezajnisto/apidcms/releases",
  "cache_ttl": 3600,
  "headers": {"Accept": "application/vnd.github.v3+json"},
  "template": "changelog"
}
```

**Catalog from NocoDB (live):**
```json
{
  "source_url": "https://app.nocodb.com/api/v2/tables/mxzy0/records",
  "json_path": "list",
  "cache_ttl": 0,
  "headers": {"xc-token": "{{ setting.nocodb_api_key }}"},
  "template": "catalog"
}
```

### Error Handling

If the API is unavailable or returns an error — `external_error.html.twig` is shown with a clear message. The site does not crash.

---

### Example: Buildin.ai (real-world case)

Integration with [Buildin.ai API v2](https://buildin.ai/developer-api/v2/getting-started/overview). Real example: wearefun.ru — a wedding guide with full Markdown content.

[/svadba](https://wearefun.ru/svadba) — table of contents list, [/svadba/{id}](https://wearefun.ru/svadba/2db6d106-ad2a-44f4-8e7e-11cfdc1c933d) — detailed page.

**Page list via Search API (POST):**

```json
{
  "source_url": "https://api.buildin.ai/v2/search",
  "json_path": "results",
  "cache_ttl": 3600,
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer {{ token }}",
    "_body": "{\"filter\":{\"property\":\"object\",\"value\":\"page\"},\"page_size\":50}"
  },
  "template": "wedding"
}
```

**Single Item Page:**

When accessing `/page/{id}`, apidcms automatically: (1) finds the page in items by id, (2) generates a content_url by replacing `/v2/search` with `/v2/pages/{id}/content/markdown`, (3) loads Markdown via fetchContent() with Authorization, (4) passes detail_content to the template.

**Single Item Template (wedding_single.html.twig):**

```twig
{% set page_title = item.properties.title.title[0].plain_text %}
{% set md = detail_content|json_decode %}

<div class="body-text">
    {{ md.markdown|markdown_to_html|raw }}
</div>
```

**Important:** the content/markdown response comes as JSON `{"markdown": "..."}`, so in the template first apply |json_decode, then access .markdown.

**Step-by-step guide:** [how we integrated Buildin.ai](https://apidcms.dezajno.ru/docs/external-page-type)
