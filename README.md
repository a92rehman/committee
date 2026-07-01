# Committee Manager

A mobile-first Progressive Web App (PWA) to manage a Pakistani **committee** (ROSCA — Rotating Savings and Credit Association). Tracks members, monthly payments, payouts, and reminders — all synced in real time via Firebase.

---

## Features

### Members
- Add members with name, phone number, and monthly contribution amount
- Support for **group/shared committee slots** — multiple people split one month's payout proportionally based on their contribution share
- Members can hold both a group slot and their own individual slot
- Send WhatsApp introduction messages to members

### Home (Monthly View)
- Navigate month by month with previous/next arrows
- Live collection progress bar showing paid vs. total pool
- Deadline warning banner (ok / warn / late states)
- Pending/partial payment list per month — individual cards and grouped cards for shared-committee members
- One-tap "Remind All" to send WhatsApp reminders to everyone who hasn't paid
- Introduction message panel

### Payments Tab
- Select any month from a sorted dropdown
- Full payment history per member with amount and date received
- Partial payment support — record multiple instalments
- "Mark Full Paid" shortcut
- Undo last payment / clear all payments
- Every payment entry records the **date the payment was received** (user-selected) and the exact **date + time it was entered** into the system (`recordedAt` — ISO 8601)
- Paid completion timestamp stored as full ISO datetime

### Timeline Tab
- Chronological view of all committee months, sorted by actual calendar date
- Current month highlighted, past months dimmed
- Per-slot payout breakdown with exact amounts for group members
- Mark payout as sent (✓) with full datetime stamp
- **Reverse** a payout mark with the ↩ undo button
- WhatsApp "Mubarak" message for payout recipients
- Per-month notes

### Settings
- Configure start year/month (auto-names all months in order)
- Payment deadline day
- Bank account details (injected into reminder messages)
- Customisable WhatsApp message templates (reminder, partial, mubarak)
- Export data as JSON backup
- Import/restore from backup file

### Sync & Offline
- Real-time sync via Firebase Realtime Database (SSE stream)
- All 4 tabs re-render automatically on any remote change
- Retry logic with exponential backoff (2s → 4s → 8s) on push failure
- SSE reconnect with backoff (1s → 64s) on connection drop
- `localStorage` as offline fallback with rolling safety snapshot
- Sync status indicator in the header (green / amber / red dot)

---

## Data Model

All data lives under a single Firebase node (`/committee`) and mirrors `localStorage`.

| Key | Type | Description |
|-----|------|-------------|
| `members` | `Member[]` | `{id, name, monthly, phone}` |
| `schedule` | `Month[]` | `{month, slots}` — always sorted by calendar date |
| `payments` | `Record<key, PayInfo>` | key = `"monthIndex-memberId"` → `{history:[{amount, date, recordedAt}], totalPaid, status, completedDate}` |
| `reminders` | `Record<key, RemInfo>` | key = `"monthIndex-memberId"` → `{count, lastDate}` |
| `payouts` | `Record<key, PayoutInfo>` | key = `"monthIndex-memberId"` → `{sent, date}` — `date` is full ISO datetime |
| `introSent` | `Record<memberId, true>` | Tracks which members received the intro message |
| `notes` | `Record<monthIndex, string>` | Free-text notes per month |
| `appSettings` | `Settings` | `{startYear, startMonth, deadlineDay, bankDetails, templates}` |

**Slot types inside `schedule[i].slots`:**
- Single slot: `{type: "single", mid: "memberId"}`
- Group slot: `{type: "group", parts: [{mid: "memberId", share: 25000}, ...]}`

---

## Tech Stack

- **Single file** — everything in `index.html` (HTML + CSS + JS, no build step)
- **Vanilla JS** — no framework or dependencies
- **Firebase Realtime Database** — REST API + Server-Sent Events for live sync
- **PWA** — installable on iOS and Android
- **GitHub Pages** — deployed directly from the `main` branch

---

## Local Development

No build step needed. Just open `index.html` in a browser:

```bash
open index.html
# or serve locally:
npx serve .
```

---

## Deployment

Push `index.html` to the `main` branch. GitHub Pages serves it automatically at `https://a92rehman.github.io/committee/`.

---

## Firebase Setup

The Firebase Realtime Database URL is set in `index.html` line 168:

```js
var FB = 'https://<your-project>.firebaseio.com/committee';
```

To use your own Firebase project:
1. Create a Realtime Database in the [Firebase Console](https://console.firebase.google.com/)
2. Set database rules to allow read/write
3. Replace the `FB` value with your database URL
