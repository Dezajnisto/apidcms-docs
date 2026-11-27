---
title: Plugins Overview
slug: obzor-plaginov
section: plugins-dev
order: 60
description: "Plugins Overview — extending apidcms functionality without modifying the core."
---

# 🔌 Plugins

Plugins extend apidcms functionality without modifying the core. Each plugin is an isolated folder in `plugins/plugin_name/`.

The core ships with two plugins: **Account** (user cabinet) and **Cart** (shopping cart).

---

## Where to Find

In the admin panel: sidebar menu → **Plugins** (🧩 icon).

---

## Plugin Management

### Plugin List (`/admin/plugins`)

A table of all plugins: name, version, description, status. Buttons for **Activate** / **Deactivate** and **Settings**.

### Plugin Page (`/admin/plugins/name`)

Three tabs:

- **Info** — name, version, status, dependencies, contents of `plugin.json`
- **Templates** — list of plugin `.twig` files, "Edit" button
- **Settings** — plugin parameter form (currency, limits, etc.)

### Template Editor (`/admin/plugins/name/templates/file.twig`)

Full-featured code editor. Save with Ctrl+S or the button.

---

## Installation and Removal

- **Installation:** copy the plugin folder to `plugins/plugin_name/` on the hosting, then activate via the admin panel
- **Removal:** deactivate in the admin panel, then delete the `plugins/plugin_name/` folder
- **Enable/disable:** via the admin panel using Activate/Deactivate buttons

> ⚠️ When deleting the plugin folder, the database tables **remain**. Same on deactivation.

### Where the on/off state lives

`"enabled"` in `plugin.json` is the **default** — the value that ships with the
plugin. The current state of the site lives in
`storage/database/plugins-state.json` and overrides the file:

```json
{ "webchat": true, "cart": false }
```

That is why the admin toggle changes no file tracked by git. It used to rewrite
`plugin.json`, so the server copy drifted from the commit and the deploy refused
on the "files the CMS edits" check until you passed `--force`.

The state cannot live in the database: `bootstrap.php` loads plugins before the
database connection exists, so a disabled plugin still ran its `init.php` from
the value in the file. `storage/database/` is not deployed and is backed up
together with the database.

Deleting the state file returns every plugin to the default from its manifest.

---

## 🔐 Account Plugin — User Cabinet

Registration, login, and user profile.

**Routes:**
- `/register` — registration
- `/login` — login
- `/profile` — profile
- `/logout` — logout

**Features:**
- Remember-me (30-day cookie)
- Twig functions: `is_logged_in()`, `current_user()`

**Tables:** `users`, `user_tokens`

---

## 🛒 Cart Plugin — Shopping Cart

Add products to cart for catalog sites.

**AJAX endpoints:**
- `POST /cart/add` — add item
- `POST /cart/update` — change quantity
- `POST /cart/remove` — remove item

**Pages:**
- `/cart` — cart with +/- and delete

**Twig functions:** `cart_count()`, `cart_total()`

**Table:** `cart_items`

---

## 💬 WebChat Plugin — chat answered from MAX or Telegram

A chat widget for the site: the visitor writes in a window on the page, the owner
replies from a messenger. No operator panel to keep open, and the conversation
history stays in the site database.

**Features:**
- Chat button on every page; window, history, mobile layout
- Reply channel — MAX or Telegram (selected in settings, can be changed later)
- Admin page with conversations, unread count, search, and replying from the panel
- "Check connection" button: subscribes the webhook and sends a test message
- Anti-spam: limits on conversations and messages, IP stored only as a hash

**Routes:**

| Path | Purpose |
|---|---|
| `GET /chat/widget` | widget script (CSS embedded in the response) |
| `POST /chat/api/start` | create a conversation, get the visitor token |
| `POST /chat/api/send` | visitor message |
| `GET /chat/api/poll` | new messages |
| `POST /chat/api/hook/max` | MAX webhook |
| `POST /chat/api/hook/telegram` | Telegram webhook |
| `/admin/chats` | conversations and replies |

> The widget script is served from a path **without** the `.js` extension on
> purpose: the usual nginx rule `location ~* \.(css|js)$` serves such addresses
> as static files, and the request never reaches PHP.

**Tables:** `plugin_webchat_threads`, `plugin_webchat_messages`

---

## 🗣️ Community Plugin — forum, comments, communities

One plugin for three jobs: a plain forum, comments under site pages, and
private communities. The trick is simple: **a forum topic is an ordinary record
with an `embed_code` field**. Empty — it is a forum topic; filled — the topic is
bound to a site page and works as comments under it. Moderation, replies,
quoting, reply subscriptions and pagination are therefore shared code, not three
separate features.

**Features:**
- Forum: channels, topics, pinning, locking, pagination, view counter
- Comments under a page or blog post — without editing theme templates
- Images in posts (uploaded through the form, type checked by content)
- Markdown in safe mode: raw HTML escaped, `javascript:` and `data:` stripped
- Moderation queue in admin: "publish / spam / delete", counter in the menu
- Anti-spam: honeypot, minimum fill time, per-address hash limits, stop words
- Reply notifications by email using the site's mail settings
- Works without the Account plugin too: guest comments with moderation remain

**Channels by mode:**

| `mode` | What | Who creates topics | Where it shows |
|---|---|---|---|
| `forum` | forum channel | users | `/forum/<slug>` |
| `comments` | service channel, hidden | automatically on first comment | under the page text |
| `community` | community | members | `/community/<slug>` |

**Routes:**

| Path | Purpose |
|---|---|
| `GET /forum` | channel list |
| `GET /forum/<channel>` | topics of a channel |
| `GET /forum/<channel>/<topic>` | posts and reply form |
| `GET/POST /forum/<channel>/new` | new topic |
| `GET /community` | community list |
| `GET /community/<slug>` | community page: discussions, members |
| `POST /community/<slug>/join` | join or apply |
| `POST /community/<slug>/leave` | leave |
| `POST /community/<slug>/members` | moderator actions: approve, reject, promote, remove |
| `POST /forum/<channel>/<topic>/reply` | reply |
| `POST /community/reply` | comment on a page or record |
| `POST /community/upload` | image upload (JSON) |
| `/admin/community` | moderation queue |

**Communities.** A community is the same channel with `mode = community`.
Join policy (`join_policy`): `open` — one click, `approval` — a request approved
by a channel moderator, `closed` — invitation only. It is set from a list in
`/admin/community`, not as free text in the table editor.

A non-member sees the title, description, member count and a join button;
topics and the member list stay hidden. A channel moderator approves requests
and removes members without the site admin; nobody can demote themselves.
Moderators get an email about every new request.

> Communities need the `account` plugin: there is nobody to join on a site
> without registration. Comments and a guest forum work without it.

**Tables:** `plugin_community_channels`, `plugin_community_topics`,
`plugin_community_posts`, `plugin_community_followers`, `plugin_community_members`

**Styling:** `assets/community.css` sets no background and inherits text colour
from the theme; the accent comes from `--accent-purple` when the theme declares it.

**What a topic can be attached to:**

| `embed_code` | Meaning |
|---|---|
| `page:12` | a site page |
| `item:posts:7` | a blog post |
| `item:catalog:2` | a record of any section — catalog, products |

The table name comes from `nav_item->source_table`, so records with equal ids in
different sections never share a topic.

**If a template does not print the record text** (a catalog card loads its text
with a separate request), there is nothing to append to — the block is then
inserted before `</main>` on the `front.output` hook. The `<!--apc-comments-->`
marker prevents inserting it twice. Setting `comments_position`: `auto` (in the
article text, otherwise at the end of the page), `content` (article text only),
`end` (always at the end of the page).

To place the block at a specific spot in a template, use the function:

```twig
{{ community_comments(page) }}
```

**Anti-spam.** The IP is never stored: only `HMAC-SHA256(ip, plugin secret)`.
The hash is enough to count frequency, not enough to recover the address.
Suspicious posts are queued for review; stop words and the bot trap are rejected.

---

> **If a plugin renders its own pages**, it bypasses
> `FrontController::render()` — and therefore the `front.output` filter does not
> run by itself. Everything subscribed to that filter (plugin styles, widgets,
> counters) will be missing on such a page. Run the finished HTML through the
> filter yourself:
>
> ```php
> $html = \Core\PluginManager::getInstance()->applyFilters('front.output', $html, $fc);
> echo $html;
> ```
>
> This is a real trap: the forum was served without CSS until this was added.

## 🧩 Extension points (hooks)

A plugin subscribes to hooks through `PluginManager`. Actions (`addAction`)
return nothing, filters (`addFilter`) take a value and return the changed one.

| Hook | Type | When | Why |
|---|---|---|---|
| `core.init` | action | core loaded | early plugin initialisation |
| `db.migrate` | action | loaded with a database | create and verify tables |
| `twig.init` | action | frontend Twig ready | own templates and functions on the site |
| `front.router.before` | action | before routing | own public routes (`$path`) |
| `front.router.after` | action | after routing | does not run if the page was sent |
| `front.render` | filter | before rendering | add data to the template |
| `front.output` | filter | finished HTML | insert a widget/counter on every page |
| `admin.routes` | action | admin routes being built | own admin page |
| `admin.twig.init` | action | admin Twig ready | path to own templates |
| `admin.menu` | filter | menu rendering | sidebar item (with `order` and `badge`) |
| `admin.current_section` | filter | menu highlighting | which item counts as active |

**Own admin page.** The plugin controller lives in its own namespace; the core
finds it by a class name containing a backslash:

```php
$pm->addAction('admin.routes', function ($app) {
    $app->addRoute('/chats', 'Plugins\Webchat\AdminChatsController', 'index');
    $app->addRoute('/chats/{id}', 'Plugins\Webchat\AdminChatsController', 'thread');
}, 10, 'webchat');

$pm->addAction('admin.twig.init', function ($controller, $twig) use ($pluginDir) {
    $twig->getLoader()->addPath($pluginDir . '/views', 'webchat');
}, 10, 'webchat');
```

The controller extends `Admin\BaseController` and renders with the namespace:
`$this->render('@webchat/chats.html.twig', [...])`.

> ⚠️ **Always end your own response in `front.router.before` with `exit`.** If
> you return control to `FrontController`, it draws its own 404 and overwrites
> the status — a `304 Not Modified` or `204` becomes a `404` with an empty body.
> That is how "the widget disappears every other reload" was caught.

**Menu item** — an array with `title`, `url`, `section` (matches the value
returned by `admin.current_section`), `badge` (number for the red counter) and
`order` (sorting, 50 by default):

```php
$pm->addFilter('admin.menu', function ($items) {
    $items[] = ['title' => 'Chats', 'url' => '/admin/chats', 'section' => 'chats', 'badge' => 3, 'order' => 60];
    return $items;
}, 10, 'webchat');
```

**A widget on every page** is inserted with the `front.output` filter — the only
way that does not require editing the templates of every theme:

```php
$pm->addFilter('front.output', function ($html) {
    return str_replace('</body>', '<script src="/chat/widget" defer></script></body>', $html);
}, 10, 'webchat');
```

---

## Plugin Settings

A plugin declares its configurable parameters in `plugin.json` — currency, limits,
behaviour. The actual values are stored in the site database, in the
`plugin_settings` table, not in the manifest: the manifest is code, it lives in
git and is deployed, while settings belong to one site and some of them are
secrets. The table is created automatically on first use, no migration needed.

```json
"settings": [
  {
    "key": "currency",
    "type": "text",
    "value": "₽",
    "label": "Currency",
    "description": "Currency symbol used to format prices"
  },
  {
    "key": "payment_secret",
    "type": "text",
    "secret": true,
    "value": "",
    "label": "Payment gateway secret key"
  }
]
```

`value` is the default here — the part git is allowed to see. The settings editor
in the admin panel (the "Settings" tab on the plugin page) writes what you enter
into `plugin_settings`. A plugin reads the value as usual:

```php
$pm = \Core\PluginManager::getInstance();
$cfg = $pm->getPlugin('cart');
foreach ($cfg['settings'] as $s) {
    if ($s['key'] === 'currency') { $currency = $s['value']; }
}
```

Where the value came from is none of the plugin's business.

Field types in the settings form: `text` (default), `textarea`, `number`,
`checkbox`, `select` (with an `options` map) and `secret` — a flag that applies
on top of any type.

### The `secret` flag

`"secret": true` marks keys and passwords. Such a value is never rendered into
HTML (password field, an empty submit means "keep it", clearing is a separate
checkbox), it is masked as `***` on the "Info" tab, and it is never written back
to `plugin.json` — neither when settings are saved nor when the plugin is
activated or deactivated.

If the manifest still carries a value with this flag (an older plugin version),
the core moves it into the database on the first request, blanks it in the file
and writes a line to `error_log`. An update therefore neither loses the key nor
leaves it in the code.

---

## 📊 Plugin Tables in the Admin Panel

Plugins can declare their tables via the `tables` field in `plugin.json`. This lets the admin panel show plugin tables on the dashboard in a separate "Plugins" group (alongside system and user tables).

**Example plugin.json with tables:**

```json
{
  "name": "account",
  "version": "1.0.0",
  "description": "User cabinet",
  "tables": ["plugin_account_users", "plugin_account_tokens"]
}
```

After adding the `tables` field, plugin tables will appear in the "Plugins" section on the admin dashboard.

---

##


## 🏷️ Table Naming Convention

All plugin tables MUST use the `plugin_{name}_` prefix. This prevents name conflicts with user-created tables (e.g. if a user creates a `users` table via admin panel, and the Account plugin tries to create its own).

**Format:** `plugin_{plugin_name}_{purpose}`

**Examples:**
- `plugin_account_users` — Account plugin users table
- `plugin_cart_items` — Cart plugin items table
- `plugin_credits_transactions` — Credits plugin transactions table

**Rules:**
- Plugin name — only `[a-z][a-z0-9]*` (no underscores)
- Table name — lowercase with underscores
- Full name: `plugin_{name}_{purpose}`

### Migrations with Schema Verification

Every plugin migration must not only create tables (`CREATE TABLE IF NOT EXISTS`), but also verify the schema of existing tables via `PRAGMA table_info` and `ALTER TABLE ADD COLUMN`. This ensures that when a plugin is updated, new columns are added without data loss.

**Migration example with PRAGMA verification:**

```php
$pm->addAction('db.migrate', function ($db) use ($pluginDir) {
    $migrationFile = $pluginDir . '/migrations/001_create.sql';
    if (!file_exists($migrationFile)) return;
    $sql = file_get_contents($migrationFile);
    $statements = array_filter(array_map('trim', explode(';', $sql)));
    foreach ($statements as $stmt) {
        if (!empty($stmt)) {
            try { $db->exec($stmt); }
            catch (\Exception $e) { error_log("Migration error: " . $e->getMessage()); }
        }
    }

    // Schema verification
    $expectedColumns = [
        'plugin_myplugin_items' => [
            'id' => 'INTEGER',
            'name' => 'TEXT',
            'created_at' => 'DATETIME'
        ]
    ];

    foreach ($expectedColumns as $table => $columns) {
        try {
            $existing = $db->query("PRAGMA table_info(\"{$table}\")")->fetchAll();
            $existingNames = array_column($existing, 'name');
            foreach ($columns as $colName => $colType) {
                if (!in_array($colName, $existingNames)) {
                    $db->exec("ALTER TABLE \"{$table}\" ADD COLUMN \"{$colName}\" {$colType} DEFAULT ''");
                    error_log("Added missing column {$table}.{$colName}");
                }
            }
        } catch (\Exception $e) {
            error_log("Schema verification error: " . $e->getMessage());
        }
    }
}, 10, 'myplugin');
```

### Foreign Keys

If a table references another plugin's table (e.g. `plugin_account_users`), the plugin must:
- Declare the dependency in `plugin.json`: `"dependencies": ["account"]`
- Use the full table name in FOREIGN KEY
