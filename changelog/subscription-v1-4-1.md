# Subscription v1.4.1 — V2 Compatibility Fix

**Дата:** 30 июля 2026
**Тип:** hotfix

**Исправления:**
- `getDb()` теперь использует `$fc->getDatabase()` (новый метод FrontController) при наличии.
- Fallback: загрузка конфига через v2-путь (`config/front.php` + core defaults) вместо старого `front/config/config.php`.

---

# Subscription v1.4.1 — V2 Compatibility Fix

**Date:** July 30, 2026
**Type:** hotfix

**Fixes:**
- `getDb()` now uses `$fc->getDatabase()` (new FrontController method) when available.
- Fallback: loads config via v2 path (`config/front.php` + core defaults) instead of the old `front/config/config.php`.
