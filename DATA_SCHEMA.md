# Data Schema

## Resource Object

Each saved resource is stored as a JSON object in the `tagdrop-resources` localStorage array.

```json
{
  "id":          "uuid-v4-string",
  "title":       "Video or page title (max 100 chars)",
  "url":         "https://...",
  "desc":        "AI-generated description (max 500 chars)",
  "catId":       "category-id-string",
  "tags":        ["Creator Name", "TechTag", "topic-tag"],
  "createdAt":   "2025-01-15T10:30:00.000Z",
  "publishedAt": "2024-06-01T00:00:00.000Z",
  "ytStats": {
    "views":    1250000,
    "likes":    42000,
    "comments": 1800
  },
  "ytDuration":  874,
  "favourite":   false,
  "archived":    false,
  "watched":     false
}
```

### Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | UUID generated at creation time |
| `title` | `string` | Resource title, max 100 characters |
| `url` | `string` | Full URL including protocol |
| `desc` | `string` | AI-generated description, max 500 characters |
| `catId` | `string` | ID of the assigned category |
| `tags` | `string[]` | Ordered tag array — max 14 tags |
| `createdAt` | `string` | ISO 8601 timestamp when added to library |
| `publishedAt` | `string \| null` | ISO 8601 — YouTube video publish date, or `null` |
| `ytStats` | `object \| null` | YouTube stats object, or `null` if unavailable |
| `ytStats.views` | `number \| null` | View count |
| `ytStats.likes` | `number \| null` | Like count (null if hidden by creator) |
| `ytStats.comments` | `number \| null` | Comment count |
| `ytDuration` | `number \| null` | Video duration in **seconds**, or `null` |
| `favourite` | `boolean` | `true` if the user has starred this resource. Defaults to `false`. |
| `archived` | `boolean` | `true` if the resource has been soft-deleted. Defaults to `false`. |
| `watched` | `boolean` | `true` if the user has marked this resource as watched. Watched resources are visually dimmed and grouped at the bottom of the list. Defaults to `false`. |

### Tag Ordering Convention

Tags are stored in a flat array with a defined order:

```
[creatorTag?, ...techTags, ...topicTags]
```

- **Position 0** — Creator tag: the YouTube channel name (only on AI-added YouTube resources)
- **Technology tags** — framework/model/tool names; identified by containing uppercase letters or digit-letter mixes (e.g. `GPT-4o`, `LangChain`, `n8n`)
- **Topic tags** — lowercase keyword tags (e.g. `rag`, `fine-tuning`, `agents`)

**Creator-priority deduplication:** when building the sidebar tag groups, any tag that is a creator (appears at position 0 on any YouTube resource) is always placed in the Creators section — never in Technologies — even if its name would otherwise match the technology heuristic (e.g. a channel named "Google AI"). This is enforced by a two-pass algorithm in `renderSidebar()`.

### Tag Filter Logic

Tag filtering uses **hybrid OR/AND logic**:
- **Within the same type** (e.g., two creators): **OR** — resource matches if it has at least one of the selected tags
- **Across different types** (e.g., a creator + a topic): **AND** — resource must satisfy all active type groups

---

## Category Object

```json
{
  "id":    "uuid-or-slug-string",
  "name":  "Tutorials",
  "color": "#3b82f6"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | UUID (custom categories) or slug (defaults like `tools`, `models`) |
| `name` | `string` | Display name, max 30 characters |
| `color` | `string` | Hex colour used for the category dot and list badge |

### Default Categories

| ID | Name | Color |
|----|------|-------|
| `tools` | Tools | `#6366f1` |
| `models` | Models | `#10b981` |
| `papers` | Papers | `#f59e0b` |
| `tutorials` | Tutorials | `#3b82f6` |
| `prompts` | Prompts | `#8b5cf6` |
| `apis` | APIs | `#ec4899` |

---

## Backup / Export Format

### Version 2 (current)

Exported and auto-backup files share the same schema. Version 2 introduced `favourite` and `archived`; the `watched` field was added subsequently and is included automatically as all resources carry it.

```json
{
  "version":    2,
  "exportedAt": "2025-01-15T12:00:00.000Z",
  "resources":  [ ...resource objects ],
  "categories": [ ...category objects ]
}
```

> Auto-backup uses `savedAt` instead of `exportedAt`. Both keys are accepted on import.

### Version 1 (legacy)

Version 1 files are fully supported on import. Resources missing `favourite`, `archived`, or `watched` are automatically backfilled with `false` defaults during the import process. No manual migration is required.

```json
{
  "version":    1,
  "exportedAt": "2025-01-01T00:00:00.000Z",
  "resources":  [ ...resource objects without favourite/archived/watched ],
  "categories": [ ...category objects ]
}
```

### Import behaviour

| Action | Result |
|--------|--------|
| **Import (merge)** | Adds only resources and categories whose `id` does not already exist in the library. Duplicates are skipped. |

Import is always a merge — there is no destructive Replace option.

The import confirmation dialog shows: filename, export date, schema version, resource count (with favourited/archived counts), and category count.
