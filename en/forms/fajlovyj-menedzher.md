---
title: File Manager
slug: fajlovyj-menedzher
section: forms
order: 210
description: "Upload, view, rename, delete files. Display modes, pagination, image previews."
---

# 📁 File Manager

Control Panel → **File Manager**

Manage site files and images: upload, view, rename, delete, copy links.

---

## 🎯 Features

### Display Modes

Files can be viewed in two modes — the switch is saved in the browser:

| Mode | Appearance |
|---|---|
| **Grid** (default) | Icon cards, thumbnails for images |
| **Table** | Rows with columns: name, size, modification date, actions |

Toggle — buttons in the top panel:

```
[⊞ Grid] [☰ Table]
```

### Pagination

When a folder has many files, page navigation is enabled:

- **10 / 20 / 50** files per page — selectable in the top panel
- Page numbers at the bottom with first/last page navigation
- Counter: _"1–10 of 115"_

### File Actions

Hover over a file/folder — action buttons appear:

- **✏️ Rename** — automatically transliterates Russian names
- **🔗 Copy Link** — file URL is copied to clipboard
- **🗑 Delete** — with confirmation

### Folder Navigation

- **Grid:** double-click on a folder — enter it
- **Table:** single click on a folder — enter it
- **Breadcrumbs:** path at the top, click any part to navigate
- **Click on a file** (in grid) — highlight in blue
- **Double-click on a file** — file info (size, date, dimensions for images)

---

## 📤 Uploading Files

1. **"Upload File"** button in the top right corner
2. Select a file (up to 5 MB)
3. Allowed types: `jpg, jpeg, png, gif, webp, svg, pdf, doc, docx, txt, xls, xlsx, zip`
4. The file is uploaded to the current folder

Russian file names are automatically transliterated to Latin characters.

---

## 📂 Creating Folders

**"Create Folder"** button → enter a name (Latin characters, digits, hyphens, underscores only).

---

## 🖼 Image Previews

The grid thumbnail (64×64) does not load the original — it is generated **on the fly** via the server (Imagick):

- Original 450 KB → thumbnail ~4 KB
- Cached by the browser (ETag) — no reload on re-view
- `loading="lazy"` attribute — off-screen images load only on scroll

**Result:** a grid of 10 images loads ~40 KB instead of 4.5 MB.

---

## 🔧 Display Modes (Details)

### Grid
- Images: thumbnail + file name + size
- Folders: yellow folder icon + name + "Folder" label
- Other files: blue file icon + name + size

### Table
- Icon/thumbnail + name in the first column
- Size (hidden on narrow screens)
- Modification date (hidden on mobile)
- Action buttons on the right
- Row highlighted in blue on click