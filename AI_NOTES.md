# AI notes (shared by ChatGPT and Claude)

Last updated by: Claude, 2026-10-20. Refers to `index.html` version: `v10-wip` on the `wip` branch (not released yet: the owner tries it first). `main` is `v9`.

Isaac (the owner) keeps his `vN_features_to_implement.md` planning documents on his own device. This file lives in the repo so every AI reads the same rules.

## Rules
- Newest entries go at the top of each section. Sign every entry with your name and the date, like **[Claude]** 2026-10-20.
- Never delete or rewrite the other AI's entries. Reply underneath them.
- Ask the owner before anything that could change or remove saved user data.
- Always say what was tested, and what was not.
- Order of work in a round: quality fixes, then tweaks, then features.
- Version line: `var VERSION="vN-wip";` on its own line, matching the round; `var BUILD="20 Oct 10:50";` on the next line, updated every build.
- Work on `wip`; never commit straight to `main`. Files handed to the owner are named after the version (`v8-wip.html`, `v8.html`).
- Release: copy the wip file with `-wip` removed from VERSION, open a PR `wip` → `main`, merge with a merge commit (`gh pr merge --merge`), tag (`git tag vN && git push origin vN`).
  The owner said on 2026-10-20: one release per round, merged as soon as all the work is done and every check passes.
- Keep the repo to the app: `index.html` and this file. Tests, screenshots and tooling stay outside the repo (Render serves it publicly).
- Never put a `service_role` or `sb_secret` key in the page. The Supabase publishable key and `FDC_KEY` are fine.
- Past days are read-only; only today and the previous 7 days can be opened (Calendar and arrows). Older records stay stored and synced. Logging always goes to today.

## Saved data
localStorage key `calorie-tracker-v1` (never rename):
```
{goal, days:{"YYYY-MM-DD":[{id,name,kcal,m?:{p,c,f,fi?,fip?},at?,s?}]},
 meals:[{name,kcal,p,c,f,fi?,fip?,t,q?:{v,g,p?},n?,i?:[{k,n,a,u,w}]}],
 ings:[{id,name,sn?,code?,k,p,c,f,fi?,e?,scanned,src?,rating?}],
 targets:{p,c,f,fi:{on,g}, rate?}, scores:{"YYYY-MM-DD":{v,f,t}},
 folders?:{id:{k:"i"|"m",name,t,gone?}}, fileIn?:{"i:<ingredient key>"|"m:<meal name>":{f:folder id|null,t}},
 weights?:{"YYYY-MM-DD":{kg,t}}, del:{id:ms}, sUpdated, updated, mig1}
```
- Per device, not synced: `calorie-tracker-theme` (theme), `calorie-tracker-units` (`"kg"` or `"st"`, weight display unit), `calorie-tracker-meal-keep` (saved meals the user chose to keep after an ingredient changed: `{meal name: values signature}`), `calorie-tracker-cal-done` (date today's Calendar ring last celebrated).
- Entry ids are `Date.now()+Math.random()` (the time it was logged).
- Unknown fibre is never stored as 0. Startup code must not call `save()`, stamp `updated`, or add keys (`weights` is created on the first weight save).
- v10 additions (approved by the owner 2026-10-20): `folders` (ingredient and meal folders kept apart by `k`; a removed folder stays as `gone:true`), `fileIn` (which folder each item is in; no entry or a removed folder = Unfiled; the ingredient and meal records aren't touched), and `del["m:<meal name>"]` when a saved meal is removed. All created on first use.
- v9 additions (approved by the owner 2026-10-20): `days[].at` (time logged, ms); `meals[].n` (portions the recipe makes, missing = 1; kcal/macros stay whole-recipe totals) and `meals[].i` (ingredients: `k` key, `n` name, `a` amount typed, `u` `g`/`ml`/`lb`/`e`, `w` grams or ml); `ings[].sn` (source/search name) and `ings[].code` (barcode). `days[].s` (v8 slot) is kept in data but no longer shown or written.
- Ingredient key: `c:<barcode>` if it has one, else `n:<source name or name, lower case>`; basic foods `b:<name, lower case>`. Without a barcode the source name can't be edited (meals find the ingredient by it).
- v8 additions: `days[].s` (meal slot `b`/`l`/`d`/`s`), `weights` (kg always; `kg:null` marks a removed reading), `targets.rate` (kg per week for the adaptive target, one of -1, -0.75, -0.5, -0.25, 0, 0.25, 0.5).

## Sync and merge
`sync()` reads the row, runs `merge()`, and writes only if `updated_at` is unchanged (compare-and-retry).
`merge()` builds its result from known keys only, so **any new top-level key is dropped on the next sync** unless it gets a merge rule.
Every new piece of data needs: a merge rule, deletion markers where it can be deleted, a check in `validBackup()`, and a test showing it survives a two-device merge.
- Entries by id (union minus `del`). The slot `s` travels with the entry.
- Meals by lower-cased name (newest `t`); a meal with a deletion marker `m:<name>` at or after its `t` is dropped (so a meal saved again later is kept).
- Folders and fileIn: newest `t` wins per folder id / item. Ingredients by key (barcode, else source name, else name), highest id wins. Editing an ingredient gives it a new id and marks the old id deleted.
- Goal and targets (including `targets.rate`) by newest `sUpdated`.
- Scores by date (frozen beats live).
- Weights by date, newest `t` wins; a removal is `{kg:null,t}` so it syncs like a reading.
- Known risk: a device still running v7 or older that syncs drops `weights` from the cloud row. Any v8 device puts them back on its next sync (its local copy is merged in), so nothing is lost for good.

## Behaviour to preserve
- Daily score: calories weight 60, each tracked macro weight 15. Past days are frozen once and never recalculated.
- Rings: canvas, one painter (`paintProgress`). Background redraws must not cancel a running animation. Confetti and pulse fire once and respect reduced motion.
- Day slide: arrows, date and Calendar stay put; reduced motion changes days instantly.
- Ingredient picker: one live-search combobox (`makePicker`), used by Home cooking and the Daily log's Food option. Amounts 0.1 to 100,000.
- Import asks Merge / Replace / Cancel. Themes: Green, Ocean, Berry, Slate, Sand, Custom; new UI must look right in all of them, light and dark.
- v10: Daily log chart "How the day built up": cumulative calories vs goal and tracked macros as % of target, stepped at each item's logged time; same minute = one step; missing macros count as 0g (said under the chart); untimed items listed in the note and table, not on the chart; a table under "Show as a table".
- v10: search lists for ingredients and saved meals open on folders (user folders, Unfiled, then the basic groups for ingredients); choosing a folder opens it; typing shows matches inside their folders; clearing collapses them again (`makeCombo`).
- v10: saved ingredients (Ingredients page) and saved meals (bottom of Home cooking) are listed in folders with Edit, a folder menu and Remove. Removing anything (item or folder) asks in an accessible dialog (`ask()`); folders never delete their items. Removing an ingredient keeps the totals of meals that use it and lists them under "Saved meals to check" with the missing ingredient named.
- v10: Home cooking: "Portions made" sits just above the totals; meals are edited from the saved-meals list.
- v10: entry times outside 2000–2100 are ignored for display and rejected on import.
- v9: no meal slots. Each entry shows the time it was logged (`at`, or the id for older entries: every version since v1 made the id from `Date.now()`); ids that aren't times show "Time not recorded". Newest first.
- v9: units: "each" is the default when the food has an item weight; lb for foods measured in grams (not ml, no density guessed); "each (set g/ml)" asks for the item weight, and a saved ingredient can remember it (`e`).
- v9: saved meals log one portion by default; the Daily log has a Portions box. Home cooking shows per-portion and whole-recipe totals and can load a saved meal (v9 meals only) to edit.
- v9: when an ingredient changes, saved meals using it whose recalculated totals differ are listed in Home cooking ("Saved meals to check"); nothing changes until the user picks Update (new `t`, so it syncs) or Keep (remembered per device for those values). Log entries never change.
- v9: Calendar rings are one colour from the day's score (red, yellow, green). Rings that fill get a small bulge and shake once (Calendar: today's ring, once a day per device); none with reduced motion.
- v9: a rate-limited lookup (HTTP 429, e.g. USDA DEMO_KEY) is reported as busy, not as not found.
- v8: recent foods are worked out from the last 28 days of the log (no stored list). Copies from past days get new ids; the past day isn't changed.
- v8: weight trend is an EWMA (10% a day, gaps count as several days), calculated, never stored.
- v8: adaptive target uses the last 28 days, logged days only (today excluded), needs 14 logged days and weigh-ins 14+ days apart; burn = average intake − trend change × 7,700 kcal/kg. It only suggests; the goal changes after the owner confirms (sets `sUpdated`). Suggestions never go below 1,200 kcal.
- v8: charts on canvas in theme colours with a text summary under each; 4 weeks / 3 months by day, 1 year by week. If a custom accent is too close to the over-goal colour, charts use ink for "over".

## Open questions for the owner
From ChatGPT's reviews (v9 and v10 planning docs), not approved yet:
- #1 Keep accounts' local data separate when a different account signs in (currently merges into that account).
- #2 "Replace my data" is local only; a sync can bring cloud-only data back. Label it, or add a confirmed cloud replace with a backup first.
- #3 Deletion markers expire after 90 days, so a device offline longer than that could bring a deleted item back. Keep them longer?
- #7 Conflicts trust device clocks (`sUpdated`, `t`, ids); server revisions would need a Supabase change.
- Edit or delete past weigh-ins (today only at the moment, to keep past days read-only).
Done: adaptive-target wording and USDA rate limit (v9); meals with a removed ingredient stay in review, impossible times rejected (v10).

## Handoff log (newest first)

### [Claude] 2026-10-20, v10-wip
**Did:** from v10_features_to_implement.md: ChatGPT #4 (meals using a removed ingredient stay in review, totals kept) and #5 (impossible entry times rejected and never break the page); tweaks: saved-meals list at the bottom of Home cooking with Edit and confirmed Remove, confirmed Remove for ingredients (meals using it named), Portions made above the totals; features: "How the day built up" chart with summary and table, folders for ingredients and saved meals (create, rename, remove with move-to, per-item folder menu, folder browsing in the search lists, Daily log meal search with folders).
**Tested:** headless Edge (Playwright, `file://`, network blocked, Supabase and product databases mocked), 172 checks (all v9 checks adapted plus v10): chart values, same-minute steps, untimed items, 0g note, updates on add/remove, empty and untimed days; folder create/rename/remove with validation, keyboard and click browsing, search narrowing and clearing, ingredient and meal folders kept apart; dialogs (focus on Cancel, Tab trapped, Escape, focus returned); removals with deletion markers and log untouched; two-device merge of folders, assignments and removed meals; export/import and bad-backup rejection; no overflow at 320/375px; no unexpected console errors. Phone screenshots in light, dark and Berry checked by eye.
**Not tested:** a real phone, real Supabase, real product databases. **Not released:** the owner asked to try v10-wip before it goes to `main`.

### [Claude] 2026-10-20, v9
**Did:** from v9_features_to_implement.md: ChatGPT #5 (adaptive target labelled a rough estimate for adults, not medical advice) and #7 (USDA rate limit reported as busy); tweaks: saved meals flagged for review when an ingredient changes (Update / Keep, nothing silent), meal slots replaced by the time each entry was logged, a small bulge-and-shake when a ring fills; features: portions made for meals (log one portion, choose more), one-colour Calendar rings, "each" as default unit plus lb and a typed item weight, source/search name vs display name for ingredients with barcode matching, ingredient editing, saved-meal editing.
**Tested:** headless Edge (Playwright, `file://`, network blocked, Supabase and product databases mocked), 103 checks: v8 data loads byte-for-byte unchanged; each feature and edge case (zero portions, missing item weight, ml foods with no lb, same display name with different barcodes, keep/update review, reduced motion, celebration only once, rate-limited lookup); two-device merge of the new fields; export/import and rejection of bad v9 fields; no overflow at 320/375px on every page; no console errors apart from the deliberate mocked 404/429 lookups. Phone screenshots in light, dark, Berry and Sand checked by eye.
**Not tested:** a real phone, real Supabase, real product databases.

### [Claude] 2026-10-20, v8
**Did:** quality fixes (removing an entry after a sync no longer overwrites the day with a stale list; the app moves to the new day when left open past midnight; goal capped at 500–10,000; stray global removed), then the six v8 features: single food and Quick add on the Daily log (shared picker), recent foods with Undo, copying a past day or slot, meal slots with subtotals, weight log with trend, progress charts, adaptive calorie target.
**Tested:** headless Edge (Playwright, `file://`, network blocked, Supabase and product databases mocked), 104 checks: v7 data loads byte-for-byte unchanged and startup doesn't save; each feature and its edge cases (no weight data, gaps in logging, reduced motion, rapid taps, amount limits, unknown fibre); two-device merge through the mocked cloud (weights, removals, slots, rate); export and import (Merge and Replace) and rejection of bad backups; no overflow at 320 and 375px on every page; no console errors. Screenshots at phone width in light and dark and in every theme, checked by eye.
**Not tested:** a real phone (camera scanning, iOS Safari canvas drawing and `roundRect`, touch feel of the new controls), real Supabase, and real product databases.
