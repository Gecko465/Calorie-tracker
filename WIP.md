# WIP board

Last updated: 2026-10-16 by [Claude] (added version numbers)

## Current owner
Nobody (free).
Whoever starts a session writes their name here (e.g. "[ChatGPT], started 2026-10-17") and sets it back to "Nobody (free)" in their handoff note.

## Versions
- Released (`index.html`): v1, 2026-10-16
- In progress (`wip/index.html`): v2-wip, adds Supabase accounts and cloud sync

Version rules:
- Every release gets the next whole number: v1, v2, v3 and so on. Never skip one, never reuse one.
- The version is the single line `var VERSION="vN";` at the top of the script. It is shown at the bottom of every page, so the user can see which version they are looking at.
- While a change is being tested, `wip/index.html` shows the next number with "-wip" (for example `v2-wip`). At release it becomes plain `v2` in both `wip/index.html` and `index.html`.
- Every release also saves the old file as `archive/index-vN.html` and adds a line to the Release log.

## Now (approved work)
- D-006: Supabase accounts + cloud sync, hosted as a Render static site (approved by the user 2026-10-16). Status: code written in `wip/index.html`, waiting for the user to set up Supabase and test.

## Test checklist for the current change
Fill this in for each change. Use: ✔ passed, ✘ failed, – not tested. Say who tested it and how.

| Check | Result | Tested by | Notes |
|---|---|---|---|
| JavaScript syntax check | ✔ | [Claude] | Ran with node, no errors |
| Page works with no Supabase keys pasted (Settings says sync not set up, everything else as v1) | – | | Needs phone/browser test |
| Create account and sign in | – | | Needs real Supabase project |
| Add a food on phone, it appears on laptop after Sync now | – | | |
| Sign out, data stays on the device | – | | |
| New device: sign in with an empty log, cloud data loads and is NOT overwritten by empty data | – | | Most important test |
| Existing GitHub Pages data: Export there, Import here | – | | |

## Standing regression checklist (must pass before every release)
- [ ] Page loads with no errors (AI: syntax check; user: on phone at /wip/)
- [ ] Adding a food in Daily log updates the calorie ring and macro rings
- [ ] Existing saved data still loads (goal, days, targets, saved ingredients)
- [ ] Home cooking: add ingredients, see totals, add one portion to the log
- [ ] Ingredients: save one, it appears in the Home cooking list, delete works
- [ ] Settings: goal and macro on/off and targets save; export then import works
- [ ] Barcode lookup by typed number works (user, phone)
- [ ] Camera scan works (user, phone)
- [ ] Light and dark mode are both readable

## Release log (newest first)
- v1, 2026-10-16, [Claude]: first self-hosted release. Daily log with calorie ring and macro rings, Home cooking builder, Ingredients page with barcode scan and Open Food Facts lookup, Settings with goal, macro targets, export/import.

## Handoff log (newest first)

### 2026-10-16, [Claude]
**Did:** Added Supabase email+password accounts and cloud sync to `wip/index.html` (v2-wip). The whole saved log is stored as one JSON row per user in a table `user_data` with row level security. The newest save wins (compared by a timestamp `updated` that `save()` sets). Added an "Account and sync" card at the top of Settings. Plan is to host on a Render Static Site (not a web service, which sleeps after 15 minutes).
**Files touched:** `wip/index.html` (new), `WIP.md`, `DISCUSSION.md`.
**Setup the user must do:** run the SQL from the chat in Supabase, paste the project URL and publishable key into `SUPABASE_URL` and `SUPABASE_KEY` at the top of `wip/index.html`, create the Render static site from this repo.
**Tested:** syntax only. Nothing that touches Supabase has been run, because Claude cannot reach the network.
**Not tested:** everything in the checklist marked –.
**Known issues / warnings:**
- Free Supabase projects pause after 7 days of no activity; the data survives but the project must be restored from the dashboard. Take regular exports.
- Last-write-wins uses device clocks. Two devices editing offline at the same time can overwrite each other.
- The startup migration deliberately does NOT stamp `updated`. Do not change that: a fresh device would otherwise overwrite the cloud copy with empty data.
- Never put the service_role key in the page.
**Next AI should:** wait for the user's test results, fix anything that failed, then ask the user whether to release as v2 (archive v1 first).
**Owner now:** Nobody (free).

### 2026-10-16, [Claude]
**Did:** Built v1 and set up this WIP/Discussion system.
**Files touched:** `index.html` (v1). Nothing in `wip/` yet.
**Tested:** JavaScript syntax check passed. The user confirmed on their phone that the page loads and barcode scanning works. Everything else was checked by reading the code only, not run.
**Not tested:** Camera scan on different phones/browsers, import of a backup file, large saved lists.
**Known limits:**
- Built-in foods use typical per-100g averages. The supermarket dropdown is saved with each ingredient but does not change the numbers.
- Saved ingredients are per 100g only. Millilitres are treated as grams.
- Saved ingredients cannot be edited, only deleted and re-added.
- Data lives in the browser's localStorage under `calorie-tracker-v1`. Never rename that key or change the shape of saved data without a migration (see how `mig1` and `targets` are handled in the JS).
- Data from the earlier hosted preview page did not carry over.
**Code notes:** one self-contained `index.html`, one IIFE, plain JS (`var`, `function`), no build step. Key functions: `render`, `add`, `tab`, `buildFoods`, `drawIngs`, `showProd`, `lookup`, `drawT`, `drawRings`. Page sections are `pg-log`, `pg-cook`, `pg-ing`, `pg-set`.
**Next AI should:** read DISCUSSION.md, ask the user which idea (if any) to approve, then copy `index.html` to `wip/index.html` if that has not been done.
**Owner now:** Nobody (free).
