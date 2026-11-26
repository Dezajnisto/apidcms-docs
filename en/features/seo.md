---
title: SEO — Meta Tags & Advisor
slug: seo
section: features
order: 56
description: SEO plugin — meta tags, Open Graph and JSON-LD, sitemap.xml, 301 redirects, a 404 log and a per-page checklist with a score.
---

# SEO — Meta Tags & Advisor

The plugin covers the technical side of SEO: what search engines formally expect
and what is most often forgotten on a live site.

The section lives in the admin panel: **SEO** in the left menu (`/admin/seo`).

## What the plugin does on its own

Once installed and enabled, the site gets:

| Item | Where it shows |
|---|---|
| `title`, `description`, `robots`, `canonical` | `<head>` of every page |
| Open Graph and Twitter Card | link previews in messengers and social media |
| JSON-LD (`WebSite`, `BreadcrumbList`, `Article`) | structured data for search engines |
| `hreflang` | when extra site languages are configured |
| `/sitemap.xml` | site map of pages, sections and records |
| `/robots.txt` | served only when the site has no robots.txt page of its own |

No template changes are needed: the plugin edits the finished HTML. When the
theme already prints `title` and `description`, the duplicates are removed — one
set of tags stays in `<head>`.

## Configuration

**Plugins → SEO → Settings.** Values live in the site database, not in the plugin file.

| Setting | Meaning |
|---|---|
| Output meta tags | master switch; turn it off and the plugin stops touching `<head>` |
| Open Graph and Twitter | social block |
| JSON-LD | structured data |
| Default social image | used when a page has none |
| Title template | e.g. `{title} — My site`; empty — title as is |
| Serve sitemap.xml | site map |
| robots.txt | "auto" (default), "always" or "leave alone" |
| noindex for 404 | the error page stays out of the index |
| "Generate" button | AI access through the DeepSeek key from AI settings |

## Site overview

The **SEO** section opens a list of every page with a score from 0 to 100:

- **average score** across checked pages;
- **pages in the index** — how many URLs search engines can see;
- **needs work** — score below 60;
- filters "All / Needs work / Not checked / Good" and search by URL, title and
  focus keyword.

**"Check site"** recalculates every page. The check runs in batches of 25–100
pages, so a catalogue of thousands of records does not stall the server. The check
is offline: the plugin reads the site database and never calls the network.

## Page editor

Opened by clicking a page in the overview. It contains:

- **SEO title** and **description** with length counters and a hint about the
  fallback value when the field is empty;
- **focus keyword** — the phrase checked in title, description, content, URL and density;
- **robots** — `index`/`noindex` for service pages;
- **canonical URL** — when a page opens under several addresses;
- **Open Graph** — separate social values when they differ from search ones;
- **checklist** with the score and an explanation for every item;
- **snippet preview** — how the page looks in search results;
- **"Generate title and description"** — the AI drafts text from the page content.

An empty field means "use the content value": the page title, its description and
image. **"Reset to content"** returns the page to that state.

## What the checklist checks

| Group | Items |
|---|---|
| Title | present; 30–65 chars; not repeated on other pages |
| Description | present; 70–170 chars; not repeated |
| Content | 300+ words; one H1; H2–H4 subheadings |
| Focus keyword | chosen; in title, description, content and URL; density 0.4–4.5% |
| Images and links | image alt text; internal links |
| URL | latin letters and hyphens; no deeper than three levels |
| Social and indexing | social image; noindex; canonical URL |

Checks that do not apply (word count on a listing page, links inside a product
card) are excluded from the score. An empty page does not score 0 just because
those checks are irrelevant.

The score is a guide, not a verdict: 90 guarantees nothing, but 40 almost always
means lost traffic.

## Sitemap and robots.txt

`sitemap.xml` is built automatically from:

- `pages` with an active status;
- navigation sections;
- table records shown through those sections.

Not included: pages with `noindex`, inactive records, search and AI chat
sections. The map is cached for an hour; **"Rebuild sitemap"** drops the cache
immediately.

Important: if the site already has its own `sitemap.xml` or `robots.txt` page in
navigation (type `raw`), the plugin **does not override** it — your version stays
in charge. This protects against silent robots.txt edits disappearing.

## Redirects

**SEO → Redirects.** A permanent (301) redirect is for a page that moved: the old
address keeps collecting links and traffic but leads to the new one.

- a redirect matches the exact path, ignoring `?parameters`;
- the 301 is sent before the page renders, so it works for deleted content too;
- the hit counter shows whether anyone still uses the old address.

The opposite case: the old address still exists in content — then the page opens
and the redirect never fires. Delete or rename the page first.

## 404 log

**SEO → 404 errors.** Addresses that do not exist, with hit counts and the last
date. For every address the plugin suggests a similar existing one — a hint, not
an answer: check it by eye.

Technical addresses (styles, images, `/admin`, `/api`) never reach the log.
Entries are not cleaned automatically — delete them by hand.

## AI

Generation uses the DeepSeek key from **Settings → AI**, the same key as the
built-in assistant. No extra keys.

The page editor has a **"Fill with AI"** panel at the top with four buttons:

| Button | Returns |
|---|---|
| **Everything at once** | title, description and focus keyword in one request |
| **Title** | only the SEO title, 30–65 chars |
| **Description** | only the description, 120–165 chars |
| **Focus keywords** | three variants to pick from, broad to narrow |

The same sparkle buttons sit next to the title, description and focus keyword
fields — no need to hunt for the generator elsewhere.

**Results are never inserted silently.** They appear as a suggestion under the
field with "Apply" and "Cancel": you see exactly what will be written, and the old
value is not lost. Keyword variants arrive as clickable chips.

What goes to the model: the page text (first 4000 characters), its title, URL and
the focus keyword when already typed. Per-field prompts return sharper answers
and spend fewer tokens than "everything at once".

The score is computed by the checklist, not by the model, so it does not drift
with the model's mood. After applying suggestions press "Save": the score is
recalculated on save.

No key — the generation panel is disabled, everything else works.

## If the theme renders tags itself

By default the plugin edits the finished HTML. If the theme should stay in
control, call this in its `<head>`:

```twig
<head>
    {{ seo_meta() }}
</head>
```

The plugin will not inject a second time. There is also `seo_sitemap_url()` — the
absolute sitemap address, for themes building their own index.

## Limitations

- The plugin does not check site speed, backlinks or behavioural factors.
- The model writes from the page content: no text means mediocre drafts.
- The sitemap is limited to 3000 records per table; very large catalogues need a
  dedicated generator.
- The plugin does not change the core schema: SEO fields live in their own tables
  (`plugin_seo_meta`, `plugin_seo_redirect`, `plugin_seo_404`).
