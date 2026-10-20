# AI notes (shared by ChatGPT and Claude)

Last updated by: Claude, 2026-10-20. Refers to `index.html` version: `v8` (released from `wip`, see the handoff log).

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
{goal, days:{"YYYY-MM-DD":[{id,name,kcal,m?:{p,c,f,fi?,fip?},s?}]},
 meals:[{name,kcal,p,c,f,fi?,fip?,t,q?:{v,g,p?}}],
 ings:[{id,name,k,p,c,f,fi?,e?,scanned,src?,rating?}],
 targets:{p,c,f,fi:{on,g}, rate?}, scores:{"YYYY-MM-DD":{v,f,t}},
 weights?:{"YYYY-MM-DD":{kg,t}}, del:{id:ms}, sUpdated, updated, mig1}
```
- Per device, not synced: `calorie-tracker-theme` (theme), `calorie-tracker-units` (`"kg"` or `"st"`, weight display unit).
- Entry ids are `Date.now()+Math.random()` (the time it was logged).
- Unknown fibre is never stored as 0. Startup code must not call `save()`, stamp `updated`, or add keys (`weights` is created on the first weight save).
- v8 additions: `days[].s` (meal slot `b`/`l`/`d`/`s`), `weights` (kg always; `kg:null` marks a removed reading), `targets.rate` (kg per week for the adaptive target, one of -1, -0.75, -0.5, -0.25, 0, 0.25, 0.5).

## Sync and merge
`sync()` reads the row, runs `merge()`, and writes only if `updated_at` is unchanged (compare-and-retry).
`merge()` builds its result from known keys only, so **any new top-level key is dropped on the next sync** unless it gets a merge rule.
Every new piece of data needs: a merge rule, deletion markers where it can be deleted, a check in `validBackup()`, and a test showing it survives a two-device merge.
- Entries by id (union minus `del`). The slot `s` travels with the entry.
- Meals by lower-cased name (newest `t`). Ingredients by name (highest id).
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
- v8: meal slots default by time of day (04:00 breakfast, 11:00 lunch, 15:00 snacks, 17:00 dinner, 21:30 snacks); a manual pick lasts until the time-of-day default moves on. Entries without `s` are shown in the slot for the time they were logged (display only); ids that aren't times show under "Other".
- v8: recent foods are worked out from the last 28 days of the log (no stored list). Copies from past days get new ids; the past day isn't changed.
- v8: weight trend is an EWMA (10% a day, gaps count as several days), calculated, never stored.
- v8: adaptive target uses the last 28 days, logged days only (today excluded), needs 14 logged days and weigh-ins 14+ days apart; burn = average intake − trend change × 7,700 kcal/kg. It only suggests; the goal changes after the owner confirms (sets `sUpdated`). Suggestions never go below 1,200 kcal.
- v8: charts on canvas in theme colours with a text summary under each; 4 weeks / 3 months by day, 1 year by week. If a custom accent is too close to the over-goal colour, charts use ink for "over".

## Open questions for the owner
- Should a signed-out device's local data stay separate when a different account signs in? (Currently it merges into that account. Raised by ChatGPT on 2026-10-18; changing it affects saved data.)
- Deletion markers expire after 90 days, so a device offline longer than that could bring a deleted entry back. Keep, or lengthen?
- Should weight be editable for past days in the 7-day window? (v8: today only, to keep past days read-only.)

## Handoff log (newest first)

### [Claude] 2026-10-20, v8
**Did:** quality fixes (removing an entry after a sync no longer overwrites the day with a stale list; the app moves to the new day when left open past midnight; goal capped at 500–10,000; stray global removed), then the six v8 features: single food and Quick add on the Daily log (shared picker), recent foods with Undo, copying a past day or slot, meal slots with subtotals, weight log with trend, progress charts, adaptive calorie target.
**Tested:** headless Edge (Playwright, `file://`, network blocked, Supabase and product databases mocked), 104 checks: v7 data loads byte-for-byte unchanged and startup doesn't save; each feature and its edge cases (no weight data, gaps in logging, reduced motion, rapid taps, amount limits, unknown fibre); two-device merge through the mocked cloud (weights, removals, slots, rate); export and import (Merge and Replace) and rejection of bad backups; no overflow at 320 and 375px on every page; no console errors. Screenshots at phone width in light and dark and in every theme, checked by eye.
**Not tested:** a real phone (camera scanning, iOS Safari canvas drawing and `roundRect`, touch feel of the new controls), real Supabase, and real product databases.
