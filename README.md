# TDC Room Booking

A browser-based room booking system for TDC, built on [Supabase](https://supabase.com/) with no build tools required — each version is a single `.html` file that opens straight in a browser.

---

## Versions

### `app.html` — V1 (Original)
The original vanilla JavaScript implementation. Fully functional, but uses `setInterval` to poll Supabase every 30 seconds. Because every poll replaces the entire grid's `innerHTML` with a loading spinner first, the screen flashes briefly on each automatic refresh.

### `app-v2.html` — V2 (React)
A rewrite using React 18 (loaded from CDN via Babel standalone — no build step needed). Attempted to replace the polling model with Supabase Realtime WebSocket subscriptions so only changed slots update. This version has known bugs and is kept here for reference — see [Known Issues](#known-issues-v2) below.

### `app-v3.html` — V3 (Recommended ✅)
The original vanilla JS app with a minimal targeted fix for the screen flash. A `silent` boolean parameter was added to `loadData()`:

- **`silent = false`** (default) — shows the loading spinner. Used on first load and when the user navigates weeks, where a visible refresh is expected.
- **`silent = true`** — fetches fresh data and redraws the grid invisibly, with no spinner and no flash. Used by the background poll and the tab visibility handler.

Only 4 lines changed from V1. Everything else — all logic, styling, modals, and reports — is identical.

---

## Features

- **Weekly booking grid** — Mon–Fri view with AM / PM / Full Day slots per room
- **Book a slot** — contact name, course name, attendees, room layout
- **Booking details** — click any booked slot to view details or cancel
- **Week navigation** — browse forward and back through weeks
- **Utilisation reports** — summary stats, room utilisation bars, charts by time slot and day of week, top courses, layout breakdown
- **Save as PDF** — print the report view to PDF
- **Auto-refresh** — data stays current without manual reloading

---

## Tech Stack

| Layer | Technology |
|---|---|
| Database | [Supabase](https://supabase.com/) (PostgreSQL) |
| Auth / API | Supabase JS client v2 |
| Charts | [Chart.js](https://www.chartjs.org/) v4 |
| UI (V1 & V3) | Vanilla HTML / CSS / JavaScript |
| UI (V2) | React 18 + Babel standalone (CDN) |
| Fonts | Inter (Google Fonts) |

---

## Database Schema

Two tables are required in your Supabase project:

**`rooms`**
| Column | Type |
|---|---|
| `id` | int8 (primary key) |
| `room_name` | text |

**`bookings`**
| Column | Type |
|---|---|
| `id` | int8 (primary key) |
| `room_id` | int8 (foreign key → rooms.id) |
| `booking_date` | date |
| `time_slot` | text (`AM` / `PM` / `Full Day`) |
| `contact_name` | text |
| `course_name` | text |
| `attendees` | int4 |
| `room_layout` | text |

---

## Getting Started

1. Clone the repo
2. Open `app-v3.html` directly in a browser — no server or build step needed
3. The Supabase URL and public key are already configured in the file

---

## Known Issues (V2)

- **Slot doesn't turn red after booking** — the grid only updates if Supabase Realtime is enabled on the `bookings` table in the Supabase dashboard (Database → Replication). If the WebSocket event doesn't fire, the new booking is never added to local state.
- **Type mismatch in `findBooking`** — Realtime payloads return `room_id` as a string, but `room.id` from a normal `select()` is a number. Strict equality (`===`) means newly received bookings never match a room.
- **No fallback if Realtime drops** — V2 removes the polling interval entirely. If the WebSocket disconnects, the grid silently stops updating.
- **Broken week navigation** — the `WeekNavBridge` / `WeekNavContext` pattern used to pass state upward causes an effect loop and the header arrows may not work reliably.
