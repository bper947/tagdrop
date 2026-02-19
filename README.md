# TagDrop

A single-file web application for building a personal learning content library. Paste any link — YouTube video, tutorial, article, tool, or documentation — and Gemini automatically classifies it, extracts tags, and files it into the right category. Think of it as a smart "watch later" dump for learning materials. YouTube videos additionally get live stats, duration, and publication date pulled from the YouTube Data API.

---

## Quick Start

1. Open `index.html` directly in any modern browser (no server needed)
2. Expand **API Keys** at the bottom of the sidebar and save your **Gemini API key**
3. Optionally save your **YouTube API key** for richer YouTube metadata
4. Paste a URL into the **AI Auto-Add** bar and click **Analyse**

---

## Features

### AI Auto-Add
Paste any URL and Gemini classifies it into a category, writes a description, and extracts technology and topic tags. For YouTube videos it uses the YouTube Data API (if a key is saved) to get the official title, channel name, published date, view/like/comment counts, and video duration. Falls back to oEmbed (via allorigins proxy) if no YouTube key is set.

**Duplicate detection** — if the same URL (or YouTube video ID) already exists in the library, a warning is shown before adding again.

### Resource List

Each resource is displayed as a list row with:

- **Left column** — title, URL, description, tags, YouTube stats
- **Right column** — action buttons (top), YouTube thumbnail (middle, if available), category badge and date added (bottom)

Action buttons on each row:
- ♥ **Favourite** — star the resource; turns amber when active
- 👁 **Watch** — toggle watched/unwatched; turns blue when watched
- ✎ **Edit** — open the edit modal
- ✕ **Archive** — soft-delete (recoverable)

Tags are colour-coded: **purple** = Creator · **green** = Technology · **grey** = Topic

YouTube stats row: 👁 views · 👍 likes · 💬 comments · ⏱ duration · 📅 published date

### Watched / Unwatched

Track your backlog with the 👁 button on each resource:

- Click 👁 to mark a resource **watched** — the row dims to 50% opacity and drops to a **"WATCHED"** group at the bottom of the list
- Click 👁 again to mark it **unwatched** — it returns to the main list
- Use the **"👁 Unwatched"** filter in the sidebar to see only your remaining backlog
- The watched state is included in exports and survives import

### YouTube Thumbnails

YouTube resources show a 16:9 thumbnail in the right column — free, no API key required. The thumbnail is fetched directly from YouTube's image CDN. If a video is private or deleted the thumbnail is hidden gracefully.

### Favourites
Click ♥ on any row to favourite it. Favourited resources appear under the **⭐ Favourites** filter in the sidebar.

### Archive
Click ✕ on a row to **archive** it (soft delete). Archived resources:
- Disappear from all normal views including Favourites
- Are accessible via the **🗄 Archived** filter (always at the bottom of the sidebar)
- Can be restored with the ↩ button — no data is ever permanently lost

### Tag Filtering
The sidebar groups all tags into three collapsible sections, sorted by frequency (most-used first):
- **👤 Creators** — YouTube channel names (first tag on AI-added YouTube resources). A creator tag is never shown in Technologies even if its name would otherwise match the technology heuristic.
- **⚙️ Technologies** — tools, frameworks, models (tags with uppercase letters or mixed digits)
- **🏷️ Topics** — lowercase keyword tags

Click tags to filter — **within the same type, tags use OR logic** (selecting two creators shows resources from either); **across different types, tags use AND logic** (selecting a creator + a topic shows only resources matching both). The **✕ Clear filters** button resets all active tag filters.

### Sort Bar
Sort resources by:
- Date added (newest / oldest)
- Date posted (newest / oldest) — YouTube `publishedAt`
- Most views · Most likes
- Longest / Shortest first — by video duration

### NotebookLM Export
The **📖 NotebookLM** button in the sort bar exports all YouTube URLs from the **current filtered view** as a plain `.txt` file — one URL per line. Import this into [Google NotebookLM](https://notebooklm.google/) to bulk-add videos as sources for AI analysis, audio summaries, or study guides.

### Categories
Default categories: Tools, Models, Papers, Tutorials, Prompts, APIs. Add custom categories via the sidebar input. Delete a category only after removing all resources from it. Category counts exclude archived resources.

### Refresh YouTube Stats
The **↻ Refresh YouTube Stats** button (in the Data panel) refreshes live stats for all YouTube videos in the library. It also backfills tags and descriptions via Gemini for any resources that are missing them.

### Data Management
All data is stored in `localStorage` — no backend required.

| Button | Action |
|--------|--------|
| ↓ Export collection | Downloads a `.json` file of the **current filtered view** |
| ↑ Import collection | Merges a collection file — adds resources not already in the library (by `id`), skips duplicates |
| ↻ Restore auto-backup | Reverts to the last automatically saved snapshot |
| ↻ Refresh YouTube Stats | Fetches live stats + backfills missing tags/desc |

An auto-backup snapshot is saved to `localStorage` every time data changes.

### Collapsible Sidebar Sections
All sidebar sections are collapsible — click any section header to toggle:
- **Data** — Export / Import / Restore / Refresh buttons
- **Categories** — category filters + add category input
- **👤 Creators / ⚙️ Technologies / 🏷️ Topics** — tag filter sections
- **API Keys** — key configuration (pinned to bottom; always visible)

Collapsed/expanded state for every section persists across page loads.

### API Keys Panel
The **API Keys** panel is pinned to the bottom of the sidebar. It shows a status indicator:
- 🟢 **Green** — both Gemini and YouTube keys saved
- 🟡 **Amber** — one key saved, one missing
- ⚫ **Grey** — no keys configured

### Resizable Sidebar
Drag the right edge of the sidebar to resize it (min 160px, max 480px). Width is persisted across page loads.

---

## API Keys

Both keys are stored in `localStorage` and never sent anywhere except the respective APIs.

### Gemini API Key
Used for AI classification, description generation, and tag extraction.
- Get a key at: https://aistudio.google.com/app/apikey
- Model used: `gemini-2.0-flash`

### YouTube Data API v3 Key
Used to fetch official video metadata (title, channel, stats, duration, description, uploader tags).
- Get a key at: https://console.cloud.google.com/apis/library/youtube.googleapis.com
- Enable **YouTube Data API v3** and create an API key
- Quota cost: 1 unit per video lookup (daily quota: 10,000 units)

---

## Data Storage

All data lives in `localStorage` under these keys:

| Key | Contents |
|-----|----------|
| `tagdrop-resources` | JSON array of resource objects (each has `watched`, `favourite`, `archived` fields) |
| `tagdrop-categories` | JSON array of category objects |
| `tagdrop-auto-backup` | Last auto-snapshot `{ version, savedAt, resources, categories }` |
| `gemini-api-key` | Gemini API key string |
| `youtube-api-key` | YouTube Data API v3 key string |
| `sidebar-width` | Saved sidebar width in pixels |
| `sidebar-sections` | JSON object tracking collapsed state of Data and Categories sections |
| `api-keys-panel-open` | Boolean — whether the API Keys panel is expanded |

See `DATA_SCHEMA.md` for the full resource and backup format schema.

---

## Deployment

TagDrop is a single `index.html` file with no backend or database. Each visitor's data is stored entirely in their own browser — libraries are completely isolated and private. See `RECOMMENDATIONS.md` for full deployment options.

### GitHub Pages (free, recommended)
1. Create a public GitHub repository (e.g. `tagdrop`)
2. Upload `index.html` to the repo root
3. Go to **Settings → Pages → Source → Deploy from branch → main → / (root)**
4. Your app is live at `https://your-username.github.io/tagdrop`

### Sharing libraries between users
Use **Export / Import JSON** — filter first, export the curated subset, send the `.json` file, and the recipient imports it to merge into their library.

---

## Browser Compatibility

Requires a modern browser with support for: `localStorage`, `fetch` API, `crypto.randomUUID()`, CSS Grid and CSS custom properties. Tested in Chrome and Edge. Works as a local `file://` URL.

---

## File Structure

```
todo-app/
├── index.html         # Entire application (HTML + CSS + JS, single file)
├── README.md          # This file
├── CHANGELOG.md       # Feature history
├── DATA_SCHEMA.md     # Resource and category data structures + backup format
└── RECOMMENDATIONS.md # Feature roadmap, technical improvements, deployment notes
```
