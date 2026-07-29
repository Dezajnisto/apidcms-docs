---
title: Inline Cell Editing with EasyMDE
slug: inline-cell-editing
section: content
order: 145
description: "Double-click a cell — modal window with EasyMDE Markdown editor. Toolbar, preview, keyboard shortcuts."
---

## Inline Cell Editing with EasyMDE

Since version **v1.3.16**, apidcms supports inline editing of cell values via a modal window with a built-in **EasyMDE** — a full-featured Markdown editor.

### How It Works

- Open any table in the admin panel: `/admin/table/{table_name}`
- Double-click a cell — a modal window with EasyMDE opens
- Edit the content using the toolbar or manually
- Ctrl+Enter or **Save** button — save
- Escape or **Cancel** button — cancel

### EasyMDE Features

| Feature | Description |
|---------|-------------|
| **Bold** | Bold text |
| **Italic** | Italic text |
| **Heading** | Headings |
| **Quote** | Blockquotes |
| **UL/OL** | Lists |
| **Link** | Insert links |
| **Image** | Insert images |
| **Preview** | Markdown preview |
| **Side-by-side** | Editor and preview side by side |
| **Fullscreen** | Full screen mode |

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Ctrl` + `Enter` | Save changes |
| `Esc` | Cancel and close |
| Click outside modal | Cancel and close |

### Design Changes

- Vertical cell dividers
- Cell highlight on hover (cursor: cell)
- Green flash on successful save

### API Endpoint

```
POST /admin/table/{table}/cell/{id}
Content-Type: application/json

{ "column": "title", "value": "# New Title" }

// Response:
{ "success": true, "value": "# New Title" }

```

### Security

- Accessible only through the admin panel (ADMIN_ACCESS)
- Column name validation against the table structure
- PDO prepared statements

### Technical Details

- **EasyMDE 2.21.0** — 320 KB JS + 13 KB CSS (embedded in the template)
- Editor is initialized when the modal opens, destroyed when it closes
- Modified core files: `TableController.php`, `App.php`, `view.html.twig`
- No external dependencies — everything is local
