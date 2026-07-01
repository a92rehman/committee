# CLAUDE.md — Committee Manager

This file is read automatically by Claude Code at the start of every session.

---

## Project Overview

Single-file PWA (`index.html`) for managing a Pakistani committee (ROSCA). Deployed via GitHub Pages. All logic, styles, and markup are in one file — no build step, no framework, no dependencies.

## Architecture

**One file:** `index.html` (~1400 lines). Sections in order:
1. CSS (lines ~10–200)
2. HTML shell — loading screen, header, nav, 4 page divs, modals
3. JS globals and data variables
4. Utility functions
5. Firebase sync functions (`fbPush`, `fbPull`, `fbListen`)
6. Renderer functions: `R.home()`, `R.timeline()`, `R.members()`, `R.payments()`
7. Event delegation — single `click` listener on `document`
8. Modal builders (payment modal, member form, month editor, settings)
9. Init / boot sequence

## Data Variables (global, in-memory)

| Variable | Type | Purpose |
|----------|------|---------|
| `M` | `Member[]` | Members: `{id, name, monthly, phone}` |
| `S` | `Month[]` | Schedule: `{month, slots[]}` — always sorted by calendar date via `sortSchedule()` |
| `PAY` | `object` | Payment records keyed `"mi-mid"` |
| `REM` | `object` | Reminder counts keyed `"mi-mid"` |
| `PAYOUT` | `object` | Payout sent flags keyed `"mi-mid"` |
| `INTRO` | `object` | Intro message sent flags `{mid: true}` |
| `NOTES` | `object` | Month notes `{mi: "text"}` |
| `SET` | `object` | App settings |

## Key Functions

- `tdy()` → `"YYYY-MM-DD"` (for date inputs only)
- `now()` → ISO 8601 string (for all auto-recorded timestamps)
- `fD(s)` → `"DD/MM/YY"` — handles both date strings and ISO timestamps
- `fDT(s)` → `"DD/MM/YY HH:MM"` — full datetime display
- `payKey(mi, mid)` → `"mi-mid"` — universal key format for PAY/REM/PAYOUT
- `sortSchedule()` — re-sorts `S[]` by parsed calendar month; remaps all PAY/REM/PAYOUT/NOTES keys to new indices
- `svAll()` — saves to localStorage + Firebase + re-renders all 4 tabs
- `applyD(d)` — applies downloaded Firebase data; calls `sortSchedule()` after

## Firebase

- URL hardcoded at line ~168: `var FB = 'https://committee-app-123-default-rtdb.firebaseio.com/committee'`
- Uses REST API (`PUT` for push, `GET` for pull, SSE for live updates)
- Retry on push: 3 attempts with 2s / 4s / 8s backoff
- SSE reconnect: exponential backoff 1s → 64s, stored in `_es` / `_esRetry`

## Timestamps — Important

All financial events record full ISO datetime via `now()`:
- `PAY[key].history[i].recordedAt` — when payment was entered into the system
- `PAY[key].completedDate` — when payment was fully completed
- `PAYOUT[key].date` — when payout was marked sent
- `REM[key].lastDate` — when last reminder was sent

The `date` field on payment history entries is the **user-selected** date the payment was received (may differ from `recordedAt`).

## Month Sorting

`S[]` is always kept in calendar order. `sortSchedule()` must be called:
- After `applyD()` (loading from Firebase or localStorage)
- After adding a new month (`doAddMonth`)
- After updating start settings (`saveStart`)

Payment keys are re-mapped during sort so indices in PAY/REM/PAYOUT/NOTES stay consistent with the new S[] positions.

## Group Committee Slots

A slot can be `{type:"group", parts:[{mid, share}]}`. Multiple people share one payout month proportionally by their `share` vs. the group total. The permanent group identity key is `parts[].mid` values sorted and joined with `"|"`. Group members are shown in a purple card across all tabs.

## Event Handling

All UI actions use `data-act="actionName"` attributes. One `click` listener on `document` handles everything via `if/else` on `b.dataset.act`. Common data attributes: `data-mi` (month index), `data-mid` (member ID), `data-idx` (generic index).

## Deployment

```bash
git add index.html
git commit -m "description"
git push origin main
# GitHub Pages auto-deploys from main branch
```

## Coding Conventions

- Minified/compressed JS style inside the file (no unnecessary whitespace in logic)
- No comments unless the WHY is non-obvious
- CSS uses short utility class names: `.card`, `.row`, `.av`, `.bp`, `.bs`, etc.
- CSS variables in `:root` — `--g` (green), `--gd` (gold), `--r` (red), `--o` (orange), `--wa` (WhatsApp green)
- All modals use `openM(html)` / `closeM()` helpers
- Toast notifications via `toast("message")`

## Skills Installed

Located in `.claude/skills/`:
- `frontend-design` — UI/UX improvements
- `firebase-basics` — Firebase workflows
- `web-design-guidelines` — UI compliance review
