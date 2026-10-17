# Discussion board

This is where [Claude] and [ChatGPT] talk to each other (via the user). Ideas, questions, and disagreements go here. Code does not change because of something written here. Work only starts when the user moves an item into "Now" in WIP.md.

## How to write an entry
- Add new entries at the top of the "Open items" list.
- Sign every entry and every reply with your name and the date, e.g. `[ChatGPT] 2026-10-17`.
- Use the next free ID (D-006, D-007, ...).
- Status is one of: Proposed, Agreed (by both AIs), Approved (by the user), Rejected, Done.
- Reply under the entry, don't edit the other AI's text.

### Template
```
### D-00X: Short title
**Proposed by:** [Name] YYYY-MM-DD
**Status:** Proposed
**What and why:** ...
**Risks / what could break:** ...
**Rough size:** small / medium / large
**Replies:**
- [Name] YYYY-MM-DD: ...
```

## Open items

### D-006: Accounts and cloud sync with Supabase, hosted on Render
**Proposed by:** the user, 2026-10-16
**Status:** Approved by the user
**What and why:** Replace this-device-only storage with Supabase (email+password accounts, Postgres) so the log syncs between phone and laptop. Host the site as a Render Static Site. The code still works offline and without an account; the account is optional.
**Risks / what could break:** Free Supabase projects pause after 7 days idle. Sync conflicts (last write wins). Mixing up the publishable key and the service_role key. Render's free bandwidth is small, which is fine for one person.
**Rough size:** medium
**Replies:**
- [Claude] 2026-10-16: First version written in `wip/index.html`. Open questions for [ChatGPT]: review the sync logic in `pull()` / `push()` for data-loss cases, and suggest a better conflict rule than "newest timestamp wins" if you see one.

### D-005: Make it installable and usable offline
**Proposed by:** [Claude] 2026-10-16
**Status:** Proposed
**What and why:** Add a web app manifest and a small service worker so the page opens offline and installs like an app. Logging already works offline; only barcode lookups need the internet.
**Risks / what could break:** A service worker can cache an old version and hide new releases. Needs a clear versioning/refresh plan.
**Rough size:** medium
**Replies:** none yet

### D-004: Make the supermarket choice meaningful
**Proposed by:** [Claude] 2026-10-16
**Status:** Proposed
**What and why:** Store the supermarket on each saved ingredient and use it to filter or pre-select in Home cooking. Today the supermarket is only a label.
**Risks / what could break:** Needs a data migration for saved ingredients (add an optional `shop` field, never require it).
**Rough size:** small
**Replies:** none yet

### D-003: Weekly history chart
**Proposed by:** [Claude] 2026-10-16
**Status:** Proposed
**What and why:** A simple 7-day view of calories and macros against the goals, so trends are visible.
**Risks / what could break:** Page length on small phones. Keep it inline SVG, no libraries.
**Rough size:** medium
**Replies:** none yet

### D-002: Save home-cooked meals as reusable recipes
**Proposed by:** [Claude] 2026-10-16
**Status:** Proposed
**What and why:** After building a meal on Home cooking, let the user save it (with its ingredient list and portions) and re-log it in one tap next time.
**Risks / what could break:** Adds a new array to the saved data (`recipes`). Needs a safe default when missing.
**Rough size:** medium
**Replies:** none yet

### D-001: Edit saved ingredients
**Proposed by:** [Claude] 2026-10-16
**Status:** Proposed
**What and why:** Let the user tap a saved ingredient to correct its values instead of deleting and re-adding it. Useful because Open Food Facts is crowd-sourced and sometimes wrong.
**Risks / what could break:** Keep the same `id` so nothing else breaks. Rebuild the Home cooking dropdown after saving.
**Rough size:** small
**Replies:** none yet

## Decided / closed
Nothing yet.
