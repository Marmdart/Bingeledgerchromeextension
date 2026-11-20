# Binge Ledger

A local-first Chrome extension for people whose shows are spread across too
many streaming services. One watchlist, one place to log episodes, ratings and
notes — with a small right-edge launcher that tracks whatever you're watching.

No account, no sync server, no analytics. The library lives in
`chrome.storage.local`; unlimited viewing history lives in IndexedDB, all on
your own browser.

## Install

1. Open `chrome://extensions` in Chrome.
2. Turn on **Developer mode** (top right).
3. Click **Load unpacked** and choose this folder.
4. Pin **Binge Ledger** from the puzzle-piece menu.

After changing the source, press the reload icon on the extension card.

## What it does

- **Track in one click.** On supported streaming sites, a small circular
  launcher sits at the right edge of the page. The panel offers two explicit
  choices: **Add as new show** or **This is an existing show**. If the page is
  already tracked, it shows progress and a `+1 episode` action instead.
- **Reads the show page.** The content script extracts JSON-LD, Open Graph and
  breadcrumb data — series title, season, episode number, episode total, cover,
  description and profile URL — so an episode URL such as
  `…/watch/frieren-episode-4` is still filed under *Frieren*, not under a new
  episode-named show. Metadata from the page is merged into an existing entry
  without overwriting your edits.
- **Gets out of the way.** Press **Ctrl+Shift+H** (⌃⇧H on macOS) to hide or show
  the launcher, and it disappears automatically when the video goes fullscreen.
- **Semi-automatic episodes.** Progress is saved while you watch. If an
  episode number is detected, it is logged automatically when the video nears
  the end — high-water and deduplicated. Ambiguous seasons prompt once, then
  auto-log. Rewatches prompt once with **“Just rewatching — don’t track”** and a
  **24-hour mute**.
- **Seasons and episode counts.** One series card contains its seasons, each
  with its own watched/total episode counts and metadata. Cards show the current
  season and overall series progress.
- **A contact-style library.** If the matcher gets a title wrong, use **“This is
  an existing show”** from the popup, launcher or a show's manage menu. Pick the
  series, explicitly choose *same season* or *different season*, and the page
  becomes another watch link with a saved matching rule. Existing duplicate
  entries can be merged from the same flow.
- **One manage menu per show.** `+1` and edit buttons are replaced by a single
  **Manage** menu (or `⋯` in the popup). It contains: log next episode, **This
  is an existing show…**, **Add another watch link…**, edit, watch history,
  open the primary link, and delete.
- **Multiple watch links.** The show editor keeps a list of services and URLs.
  Add, remove and star the primary link; every source keeps its own URL and
  service name, and imports/exports preserve the list.
- **Unlimited viewing history.** Every logged viewing gets an exact timestamp
  stored in IndexedDB, including service, episode URL, auto/manual mode and
  rewatch status. Watching the same episode again on another service adds a new
  record rather than overwriting the old one.
- **Cover art and title matching.** AniList provides cover images and canonical
  titles; Bangumi resolves Chinese localizations such as 游戏王. Cross-site
  merging still requires canonical identity or a saved match rule, so two
  unrelated same-name shows are not merged just because the text matches.
- **One library for every site.** The popup gives you a quick list of what
  you're watching, the current page, and instant episode logging. The full
  dashboard adds search, sorting, status filters, editing, stats and backups.
- **Track anything.** Not on the supported list? The popup's **Add a show**
  form pre-fills the current tab's title, URL and hostname (use **Use current
  tab** to refill it), or right-click any page and choose **Track this page in
  Binge Ledger**.
- **Statuses that fit.** Watching, Planned, Completed, Paused and Dropped,
  with automatic status changes when you finish a show or start a planned one.
- **Stats.** Shows per service, a status donut, episodes logged, average
  rating and recent activity — all computed locally.
- **Backups that respect your data.** Versioned JSON export/import with a
  preview, merge or replace modes, optional settings transfer, automatic
  rolling local snapshots, and a gentle reminder to keep a portable copy.

## Supported sites

Netflix · YouTube · Crunchyroll · Hulu · Disney+ · Prime Video · Max / HBO Max ·
Apple TV+ · Paramount+ · Peacock · Tubi · HIDIVE · Bilibili · Vimeo · Plex ·
iQIYI · Viki · Shudder · MUBI · Dailymotion · The Criterion Channel

Page detection and titles are heuristics; every show can be edited afterwards
in the dashboard. Anything can be tracked manually.

## Launcher shortcut

The right-edge launcher toggles with **Ctrl+Shift+H** on Windows/Linux and
**Control+Shift+H** on macOS. Chrome reserves ⌘⇧H for its Home page shortcut, so
the extension cannot claim it. You can rebind or check the shortcut at
`chrome://extensions/shortcuts`.

The launcher also hides itself whenever the page enters fullscreen.

## Backups and moving profiles

Each Chrome profile keeps its own extension storage, so moving data is a
deliberate export → import:

1. Profile A: dashboard → **Settings** → **Export JSON**.
2. Profile B: load the extension, then **Settings** → **Import JSON**.
3. Review the preview: new, updated, kept-local and unchanged shows are listed
   before anything is written.
4. Choose **Merge with library** or **Replace library**. Replace creates both a
   local safety snapshot and a downloadable safety copy first.
5. Optionally bring over the backup's settings.

The portable JSON backup includes the **full timestamped watch history**. Merge
import preserves existing events; replace import restores the backup's history
after clearing the current one.

The extension also keeps **up to seven daily local snapshots** inside
`chrome.storage.local` (toggle in Settings). These protect the library and
settings but not unlimited viewing history, because snapshots must stay small.
A reminder appears in the library after 14 days without a portable export.

Backups are versioned. Older v1 exports import normally, and unknown fields
from newer exports are preserved where possible so future data survives a
round trip.

## Privacy

- The ledger and unlimited viewing history never leave Chrome. History is
  stored in IndexedDB on the extension origin.
- Cover art and canonical-title lookup is optional (Settings → **Cover art &
  matching**) and sends only the show title to AniList, or Bangumi for CJK
  titles. No account data, browsing history or library contents are sent.
- Without a metadata match, artwork falls back to locally generated gradients.
- Permissions: `storage` for the ledger and snapshots, `unlimitedStorage` for
  durable history, `contextMenus` for the right-click action, `activeTab` to
  read the current page only after you interact with the extension, `alarms` for
  the daily snapshot schedule, and host access for `graphql.anilist.co` /
  `api.bgm.tv` used by cover lookup.
- The content script runs only on the supported streaming domains listed in
  `manifest.json`.

## Development

The extension is plain HTML/CSS/ES modules — no build step.

```bash
npm install
npm test          # detection, matching, seasons, history, import and backup checks
python3 tools/make_icons.py   # regenerate toolbar icons
```

### Files

```
manifest.json        MV3 manifest
background.js        service worker: detection, progress, badge, context menu
content.js           isolated right-edge launcher + panel (shadow DOM)
popup.html/css/js    quick watchlist, current-tab tracking, manual add
dashboard.html/...   library, stats, settings, edit modal
lib/sites.js         streaming-site registry, title/episode heuristics
lib/store.js         local storage schema, matching, import/export
lib/backups.js       rolling snapshots, backup metadata and reminders
lib/history.js       unlimited timestamped viewing history (IndexedDB)
lib/episode.js       manual episode history logging
lib/metadata.js      AniList/Bangumi lookup, aliases, cover art
lib/enrich.js        metadata-aware show creation
lib/ui.js            shared rendering helpers
tools/               icon generator and smoke tests
```

Progress updates and the storage store are serialized with the Web Locks API,
so the popup, dashboard and service worker can write safely at the same time.

## Known limitations

- Auto-progress works on sites where the player is in the top document.
  Players embedded in cross-origin iframes won't report position (you can
  still log episodes from the popup).
- Site title formats change; if a show is detected with an odd name, edit it
  in the dashboard. The tracked page keeps matching by URL, aliases and saved
  site rules.
- Cover art lookup only accepts exact canonical title matches. Non-anime shows
  without an AniList/Bangumi record keep their generated gradient artwork.
- Auto-logging needs an episode number in the page title or a supported player
  label. If none is available, use `+1 episode`.
- When several seasons exist and the page does not name the season, the launcher
  asks once. Until a season is selected, the session is not auto-logged.
- Metadata fills the episode total for the matched season. Additional seasons can
  be added in the show editor with their own totals; automatic discovery of every
  related season is not included yet.
- Local snapshots restore library/settings only; portable JSON backups include
  the full viewing history.
- Chrome sync is intentionally not used. Export/import moves data between
  profiles.
