# Recommendations

Feature ideas, naming suggestions, and technical improvements for TagDrop.

---

## App Name

The app is named **TagDrop** — drop links in, tag them, resurface them later.
localStorage keys (`tagdrop-*`), Gemini prompts, and export filenames all use this name.

---

## What's Already Built

| Feature | Status |
|---------|--------|
| AI Auto-Add (Gemini) — title, tags, category, description | ✅ Done |
| Hybrid tag filtering — OR within type, AND across types | ✅ Done |
| Tag cross-filtering — dim incompatible tags from other type groups | ✅ Done |
| Creator tag collision fix — creator-priority deduplication in sidebar | ✅ Done |
| Favourites, Archive, per-category views | ✅ Done |
| List view (single layout mode) | ✅ Done |
| List view right-column layout — actions top, thumbnail, badge+date | ✅ Done |
| Watched / Unwatched status + WATCHED group divider | ✅ Done |
| "👁 Unwatched" sidebar filter | ✅ Done |
| YouTube thumbnails (list view, free, no API key) | ✅ Done |
| Sort by date added, date posted, views, likes, duration | ✅ Done |
| YouTube stats (views, likes, duration, published date) | ✅ Done |
| YouTube stats refresh (batch-able — see technical notes) | ✅ Done |
| Export collection JSON (current filtered view) | ✅ Done |
| Import collection JSON (merge-only, skips duplicates) | ✅ Done |
| NotebookLM export — YouTube URLs as .txt for bulk import | ✅ Done |
| Auto-backup to localStorage + restore | ✅ Done |
| Sidebar resizable | ✅ Done |
| Gemini + YouTube API key management | ✅ Done |

---

## Deployment

TagDrop is a single `index.html` file with no backend or database. **No database is required** for personal use — each visitor's library is stored entirely in their own browser (`localStorage`), completely isolated and private.

### No database required — how data works
- Each person who opens the app has their own private library in their browser
- No server, no database, no auth system needed
- Each user configures their own Gemini + YouTube API keys
- Libraries can be shared between people via **Export / Import JSON**

### GitHub Pages (free, recommended)
The easiest way to publish TagDrop so anyone can open it at a stable URL:

1. Create a free account at [github.com](https://github.com) if you don't have one
2. Create a new public repository (e.g. `tagdrop`)
3. Upload `index.html` to the repository root (drag and drop in the GitHub UI)
4. Go to **Settings → Pages → Source → Deploy from branch → main → / (root)** → Save
5. Your app is live at `https://your-username.github.io/tagdrop`

To update: upload a new `index.html` — the site updates automatically within ~1 minute.

### Alternative: Netlify Drop (instant, no Git needed)
1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag `index.html` onto the page
3. Done — instant public HTTPS URL (e.g. `https://random-name.netlify.app`)
4. Create a free Netlify account to rename the URL and manage future updates

### If you ever want a shared library (future)
If multiple people need to contribute to the **same** shared library, a backend + database would be required:
- **Easiest path:** [Firebase Firestore](https://firebase.google.com/) + Firebase Auth — Google-managed, no server to run, generous free tier
- **Alternatively:** [Supabase](https://supabase.com/) — PostgreSQL, open-source, free tier
- **Scope:** significant rebuild — resources/categories move from localStorage to the DB, authentication added, API keys managed server-side
- **Recommendation:** Start with GitHub Pages. Use Export/Import JSON to share curated collections between users. Only invest in a backend if the shared-library use case becomes essential.

---

## Feature Roadmap

### High Priority

#### Notes per resource
A free-text `notes` field for personal takeaways, timestamps, or reminders.
Show a small 📝 icon on rows that have notes. Editable in the existing edit modal.

#### Quick-add from URL
A faster alternative to AI Auto-Add: paste a URL and save immediately with just the page title (auto-fetched from oEmbed or `<title>`). Skip Gemini entirely. Useful for bulk dumping links to organise later.

#### Sneak Peek — AI audio summary (revisit)
The original implementation used browser Web Speech API which sounded too robotic.
Better options to explore:
- **Gemini TTS** (`gemini-2.5-flash-preview-tts`) — uses the existing key, returns audio, natural-sounding neural voices
- **OpenAI TTS** (`tts-1-hd`, voices: nova, shimmer, alloy) — very natural, requires an OpenAI key
- **ElevenLabs** — best quality, free tier (10k chars/month), requires a new key

Flow: click a button on any YouTube row → transcript fetched → Gemini summarises → spoken aloud.

---

### Medium Priority

#### Bulk operations
Multi-select mode (checkboxes on rows) + a bulk action bar:
- Archive selected
- Move to category
- Add/remove tag
- Mark watched
- Export selected as collection

Essential once the library grows past ~50 items.

#### Collections / Playlists
Categories are flat and single-assignment. Collections allow grouping into themed learning paths
(e.g. "RAG Deep Dive", "Weekend Watch"). A resource can belong to multiple collections.
Useful for sharing curated playlists with others via the JSON export.

#### Personal rating
A 1–5 star rating per resource, separate from YouTube's like count.
Useful for prioritising quality content. Sortable in the sort bar.

#### Content type indicators
Visual icons to distinguish content types at a glance:
- 🎬 Video (YouTube)
- 📄 Article / blog post
- 📝 Paper (arXiv, academic)
- 🛠 Tool / app
- 📚 Course / playlist

Auto-detected from URL patterns or set by Gemini during Auto-Add.

#### Keyboard shortcuts
- `/` — focus search bar
- `Esc` — close modal / clear search
- `j` / `k` — move focus between rows
- `s` — star/unstar focused row
- `w` — toggle watched/unwatched
- `a` — archive focused row
- `e` — open edit modal

#### Undo for archive
Replace the archive action with a toast notification + "Undo" button (5-second window). Much less disruptive than a blocking confirm dialog.

---

### Lower Priority

#### Responsive / mobile layout
Currently no `@media` queries — breaks on screens narrower than ~800px.
Add a collapsible sidebar (hamburger) and single-column layout for tablet/mobile.

#### Statistics dashboard
A dedicated view:
- Total resources / hours of video saved
- Watched vs unwatched breakdown
- Content added over time
- Tag distribution
- Most-saved categories

#### Drag-and-drop reordering
Add a `sortOrder` field. Let users drag rows to reorder within the current view. Useful for building a personal watch queue.

#### Export to other formats
Beyond JSON collection:
- CSV (for spreadsheets)
- Markdown (for Obsidian / Notion)
- OPML (for RSS readers)

#### Browser extension
One-click "Save to TagDrop" from any page in Chrome/Firefox.

#### Spaced repetition / resurfacing
Periodically surface items saved long ago but never watched. A "Forgotten" sidebar section.

---

## Technical Improvements

### Tag type storage
Currently tag types (creator / tech / topic) are inferred at runtime via `getTagType()` heuristics plus the creator-priority deduplication fix in `renderSidebar()`. This is partially mitigated but still fragile — editing a tag's casing can silently break its classification.

Long-term fix: store the type explicitly in the data model:
```json
"tags": [
  { "name": "Andrej Karpathy", "type": "creator" },
  { "name": "GPT-4o", "type": "tech" },
  { "name": "fine-tuning", "type": "topic" }
]
```
Requires a one-time data migration and updates to `getTagType()`, `filterGroups`, and `renderSidebar()`.

### Batch YouTube API calls
Stats refresh currently makes one API call per video with a 200ms stagger.
YouTube Data API v3 supports up to 50 video IDs per request — batching reduces 100 calls to 2.

### CSS custom properties
50+ hard-coded colour values in the stylesheet (`#1a1a2e`, `#6366f1`, etc.).
Centralise into `:root` CSS variables to simplify theming and make a dark/light toggle trivial.

### Virtual scrolling / pagination
All rows are rendered into the DOM at once. For 500+ items this causes noticeable lag.
Options: pagination (50 per page), lazy render on scroll (`IntersectionObserver`), or full virtual scrolling.

### Accessibility
- `role` and `aria-label` on interactive elements
- Focus trapping in the edit modal
- `aria-expanded` on collapsible sections
- Keyboard support for the sidebar resizer
- WCAG AA colour contrast audit

### Event delegation
Replace inline `onclick` handlers in dynamically generated HTML with event delegation on container elements.
Benefits: cleaner HTML strings, better debuggability, fewer event listeners.
