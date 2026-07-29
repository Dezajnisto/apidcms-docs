---
title: Visit Statistics
slug: statistika-poseshchenij
section: settings
order: 340
description: "Visit Statistics"
---

# 📊 Visit Statistics

Built-in analytics system. No external services required (Yandex.Metrica, Google Analytics).

---

## Enabling

1. Admin panel → Settings → **Statistics** tab
2. Enable "Collect statistics" (`stats_enabled = 1`)
3. Set the retention period (default 90 days)

---

## What Is Collected

- Page views by page and day
- Unique visitors
- Referrer sources
- Devices (desktop, mobile, tablet)
- Browsers
- Peak hours
- Days of the week

**What is NOT collected:**
- Personal data (IP is not stored)
- Cookies (not used)
- On-page behavior

---

## Dashboard

Address: `/admin/stats`

Shows:
- 4 KPIs: views, uniques, pages, sources (over 30 days)
- Daily chart
- Top-20 pages
- Distribution by source, device, browser
- Heatmap by hour and day of the week

---

## Cleanup

Automatic: every ~100 requests, records older than `stats_retention_days` are deleted.

---

## Disabling

`stats_enabled = 0` — collection stops. Existing data is preserved.
