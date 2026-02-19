# Changelog

All notable changes to the TagDrop app are documented here.

---

## [Current] — 2025

### Added
- **Watched / Unwatched status** — `watched` boolean on every resource (defaults to `false`). Toggle via 👁 button in the right column of each list row. Watched items dim to 50% opacity. On watch, item drops to a **"WATCHED"** divider group at the bottom of the list. "👁 Unwatched" sidebar filter shows only the backlog. Survives export/import.
- **YouTube thumbnails** — 16:9 thumbnail shown in the right column of each list row for YouTube resources. Free, no API key required (fetched from `img.youtube.com`). Hidden gracefully via `onerror` if the video is private or deleted.
- **NotebookLM export** — 📖 NotebookLM button in the sort bar exports YouTube URLs from the **current filtered view** as a plain `.txt` file (one URL per line) for bulk import into Google NotebookLM.
- **Creator tag collision fix** — tags that are a YouTube channel name (creator) are now always placed in the Creators section only, never simultaneously in Technologies, even if the channel name matches the technology naming heuristic (e.g. "Google AI"). Fixed via a two-pass algorithm in `renderSidebar()`: Pass 1 collects all creator keys from context resources; Pass 2 routes any matching tag to `creator` before checking any other heuristic.
- **`watched` field in data schema** — all resources now carry `watched: false` by default. The load `.map()` backfills the field on older resources automatically (spread pattern).

### Changed
- **List view right-column layout** — action buttons (♥ 👁 ✎ ✕) moved to the **top** of the right column. YouTube thumbnail sits below the buttons. Category badge and date sit at the bottom. Left column now shows only title, URL, description, tags, and stats — clean and uncluttered.
- **Export scope** — Export collection now exports the **current filtered view** only (not the full library). Filter first, then export a curated subset.
- **Import is merge-only** — the destructive Replace option has been removed. Import always merges: adds resources whose `id` doesn't already exist in the library, skips duplicates.

### Removed
- **Card grid view** (`⊞ Cards`) — list is now the only layout
- **Compact view** (`― Compact`)
- **View switcher buttons** from the sort bar
- `view-mode` localStorage key

---

## [Previous] — 2025

### Added
- **Renamed to TagDrop** — app rebranded from "AI Resources" to "TagDrop" with a 🏷️ tag emoji logo. Gemini prompts updated to describe a general learning content library. localStorage keys migrated automatically (`ai-*` → `tagdrop-*`); existing user data is preserved seamlessly.
- **Hybrid tag filtering** — tag filters now use **OR logic within the same type** (selecting two creators shows resources from either) and **AND logic across types** (selecting a creator + a topic shows only resources matching both). Previously all tags used strict AND logic.
- **Tag cross-filtering (dimming)** — when a tag is selected, incompatible tags from *other* type groups are dimmed in the sidebar. Tags of the same type as the active filter are never dimmed (preserving OR-within-type UX).
- **Code refactoring** — removed ~100 lines of dead code. Deduplicated render logic: `actionButtons()` helper, `cardCtx()` with O(1) category Map lookup, `saveAndRefresh()` replacing 11 scattered triple-calls. Search input debounced (150ms). localStorage reads wrapped in try/catch.
- **RECOMMENDATIONS.md** — feature roadmap, technical improvements, and deployment notes.

### Previous additions
- **Favourites** — ♥ button on every row; toggles `r.favourite`; persisted in localStorage. **⭐ Favourites** sidebar filter.
- **Archive (soft delete)** — ✕ archives instead of hard-deleting; **🗄 Archived** filter with ↩ Restore. No data ever permanently lost.
- **Collapsible sidebar sections** — Data and Categories sections have clickable headers with ▼ chevrons. Collapsed state persisted in `sidebar-sections`.
- **API Keys panel** — moved to a collapsible panel pinned to the bottom of the sidebar. Status dot (🟢/🟡/⚫) in header; per-key `✓ saved` / `— not set` indicators when expanded.
- **Context-aware tag counts** — sidebar tag counts reflect the active filter rather than the full resource list. Sorted highest → lowest by count.
- **Archived pinned last** — 🗄 Archived button always appears at the bottom of the category list.
- **Schema version 2** — export/auto-backup payloads include `favourite` and `archived` boolean fields.
- **Improved import dialog** — shows filename, export date, schema version, resource count (with favourited/archived breakdowns), and category count before proceeding.
- **v1 backup compatibility** — importing a v1 backup automatically backfills `favourite: false` and `archived: false`.

---

## [Earlier] — 2025

### Added
- **Resizable sidebar** — drag the right edge to resize between 160px and 480px; width persisted in localStorage
- **Sort bar** — sort by: date added, date posted, most views, most likes, longest/shortest duration
- **Video duration** — fetched from YouTube `contentDetails` API; displayed as ⏱ `14:33`; sortable
- **Refresh YouTube Stats** — refreshes live stats for all YouTube videos; backfills tags and descriptions via Gemini for resources with missing fields
- **Duplicate detection** — warns before adding a URL that already exists (matches YouTube video ID)
- **Tag counts** — resource count per tag shown in brackets `(n)`
- **Case-insensitive tag grouping** — `Python` and `python` treated as the same tag; first-seen casing is canonical
- **Tag filter sections** — sidebar groups tags into collapsible sections: Creators, Technologies, Topics
- **Multi-select tag filtering** — click multiple tags; hybrid OR-within-type / AND-across-types logic; active filter count shown in page title
- **Clear tag filters** button
- **YouTube stats on cards** — 👁 views · 👍 likes · 💬 comments · 📅 published date
- **YouTube Data API v3 integration** — official title, channel, description, uploader tags, stats
- **Creator tag** — YouTube channel name added as the first tag on AI-added YouTube resources
- **oEmbed fallback** via allorigins CORS proxy
- **Gemini enrichment helper** (`enrichWithGemini`) — reused by `analyseUrl()` and `refreshYouTubeStats()`

### Changed
- Tag type classification: position 0 on YouTube AI-added resource = Creator; uppercase/mixed-digit = Technology; lowercase = Topic
- `fetchYouTubeMetadata()` requests `snippet,statistics,contentDetails` in one API call

---

## [Initial] — 2025

### Added
- Single-file HTML/CSS/JS learning content library
- Gemini AI Auto-Add — paste a URL, Gemini classifies and files it
- Category system with colour-coded badges (Tools, Models, Papers, Tutorials, Prompts, APIs)
- Custom category creation and deletion
- Search across title, description, URL, and tags
- Edit resources via modal
- Export / Import JSON backup
- Auto-backup to localStorage on every save
- Restore from auto-backup

---

## Architecture Notes

- **Single file** — entire app is `index.html`
- **No backend** — all state in `localStorage`
- **No build step** — open directly in browser as `file://` or deploy to GitHub Pages / Netlify
- **External APIs** — Gemini (`generativelanguage.googleapis.com`) and YouTube Data API v3 (`googleapis.com/youtube/v3`)
- **CORS proxy** — `api.allorigins.win` used only for oEmbed fallback (no-key path)
- **Backup schema** — v2 (adds `favourite` + `archived` + `watched`); v1 files auto-migrated on import
