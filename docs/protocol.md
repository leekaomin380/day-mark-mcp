# Day Mark MCP — tool reference

All tools return JSON. Business errors come back as a tool result with `isError: true` and text `"<code>: <message>"`.

## Value formats

| Type | Format | Example |
|---|---|---|
| Date | `YYYY-MM-DD`, the user's **local** calendar day | `2026-10-14` |
| Instant | ISO 8601 with local offset | `2026-10-14T11:29:30+08:00` |
| ID | UUID string | `00000000-0000-0000-0000-0000DA7DA7C1` |

Nullable fields are always present with value `null`.

### Task

```json
{ "id": "…", "title": "…", "note": "", "list": { "id": "…", "name": "Tasks" },
  "starred": false, "createdAt": "…", "completedAt": null, "plannedDate": "2026-10-15",
  "leadTimeSeconds": null, "source": "app" }
```

`plannedDate` is the day the user intends to do it — not a deadline. `source` is who created the task.

### Note (journal)

```json
{ "id": "…", "date": "2026-10-14", "text": "…", "createdAt": "…", "updatedAt": null, "source": "agent" }
```

## Tools

### Read

**`daymark_list_lists`** → `{ "lists": [{ "id", "name", "isDefault" }] }`

**`daymark_list_tasks`** — `view`: `open` (default) | `my_day` | `starred` | `list`; `day` (for `my_day`, default today); `listId` (for `list`); `includeCompleted` (default false); `query` (title substring) → `{ "tasks": [Task] }`

**`daymark_search`** — `query` (required, ≤ 200 chars); `from`, `to` (optional inclusive date range); `limit` (1–200, default 50) → `{ "query", "tasks": [Task + "matchedIn"], "notes": [Note], "truncated" }`
Case-insensitive **substring** match over task titles and notes (open and completed) and journal notes. Use specific keywords.

**`daymark_get_work_record`** — `from`, `to` (required, inclusive, ≤ 366 days) →

```json
{ "from", "to", "days": [{ "date", "completedCount",
  "completions": [{ "taskId", "title", "list", "createdAt", "completedAt", "leadTimeSeconds", "source" }],
  "notes": [Note] }] }
```

Only days with activity are returned. A task completed and un-completed on the same day counts by its last event that day.

### Write

**`daymark_add_task`** — `title` (required, ≤ 500); `note` (≤ 10 000); `listId` (default list if omitted); `plannedDate`; `starred` → `{ "task": Task }`

**`daymark_update_task`** — `id` (required) plus any of `title`, `note`, `listId`, `plannedDate`, `starred`. Omit a field to leave it unchanged; `null` clears `note` or `plannedDate`. All arguments are validated before anything is written. → `{ "task": Task }`

**`daymark_add_day_note`** — `text` (required, ≤ 10 000); `date` (default today) → `{ "note": Note }`. Shown to the user as written by an agent.

**`daymark_complete_task`** / **`daymark_uncomplete_task`** — `id` → `{ "task": Task, "changed": Bool }`. Idempotent: `changed: false` when already in that state. Completion history is append-only and tagged with its source.

**`daymark_delete_task`** / **`daymark_delete_day_note`** — `id` → `{ "deleted": true, "id" }`. Soft delete. **Only items created by an agent**; anything else returns `forbidden`.

## Error codes

| Code | Meaning |
|---|---|
| `invalid_argument` | Wrong type, format or range; the message names the field |
| `not_found` | No such id, or it was deleted |
| `forbidden` | The item exists but this caller may not change it |
| `store_error` | Storage failure inside the app |

## Not available to agents

Deleting what the user created · editing journal notes · reminders.
