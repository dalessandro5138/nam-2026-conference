# Pick several tracks on the schedule's Track filter

## What this changes
On `/schedule` the track pills toggle independently, so an attendee can follow two or three tracks at once. Each pill exposes `aria-pressed`; **All** is lit when none is selected and clears the selection. The selection is `?track=a,b` in the URL and `trackSlug=a,b` to the API. Any track selection still forces list view and blocks the grid.

## Where
- `src/pages/SchedulePage.jsx` — derive `selectedTracks = filters.track.split(',')` (empty when `ALL`); the pill `onClick` adds or removes a slug and writes the joined string via `set('track', ...)` (an empty result deletes the key); the **All** button keeps `set('track', ALL)` and is active when the selection is empty; add `aria-pressed` to every pill. `BLOCKERS`, `activeKeys` and the `useFetch` deps already work on the single string, so they are unchanged (the `Filters (n)` badge counts the track filter once).
- `server/routes/sessions.js` — line 38: split `trackSlug` on `,`, trim, drop blanks, and emit `t.slug IN (?, ?)`; one value gives the same single-placeholder query as today. `speakers.js` reads its own params and is not touched.
- `tests/api/endpoints.test.js` — new case beside `filtered()`/ids using the existing `ids.track`; a second slug from `/bootstrap` tracks.
- `tests/schedule.spec.js` — new test in the Schedule filter describe, using `visit`, `waitForResults`, `API` and the per-test lane helpers (`laneFor`); mobile clicks `Filters` first.

## How it will be proved
| Done when | Check | Layer |
| --- | --- | --- |
| An API test covers `trackSlug=a,b` | `/sessions?trackSlug=a,b` returns the union of `a` and `b` results (count equals sum of the singles; every row's track is a or b) — false today, where the comma value matches nothing | API (`tests/api/endpoints.test.js`) |
| Several tracks can be selected and deselected on desktop and mobile; `result-count` matches the API after adding and after removing | click pill A, pill B (`aria-pressed` true, URL `track=a,b`, `result-count` = API count for `a,b`), then click B again (`result-count` = API count for `a`) — false today, as a second click replaces the first | Spec (`tests/schedule.spec.js`, desktop + mobile projects) |
| A single-track `?track=a` link behaves as before | visit `?track=a&view=list`; pill A pressed, `result-count` = API count for `a` | Spec; existing `endpoints.test.js` cases still pass |
| The grid stays blocked while any track is selected | with two tracks selected `grid-blocked-notice` is visible and `view-grid` is disabled | Spec |

## Decisions
- Clicking **All** while tracks are selected clears every selection, and deselecting the last pill returns to **All** — OK?

## Out of scope
- Grid view for a track selection; the speakers page's own track filter.
