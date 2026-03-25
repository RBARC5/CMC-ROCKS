# CMC Rocks Crew Portal

A live, multi-device crew management web app for CMC Rocks QLD festival operations.

**Live site:** https://rbarc5.github.io/CMC-ROCKS/

---

## Overview

The portal runs entirely in the browser with no backend server. Data is stored in `data.json` in this repository. All reads and writes go through a Cloudflare Worker proxy (`cmc-rocks-proxy.8gonula.workers.dev`) which hits the GitHub API directly — bypassing CDN caching so all devices see changes within ~8 seconds.

Crew members register once on their own device. Their identity is saved locally so they don't need to log in again.

---

## Tabs

### 2-Way Radios
### Staff Shifts

---

## 2-Way Radios

Tracks the sign-out and return of 2-way radios across the site. Visible and synced in real time across all devices.

### Signing Out a Radio

1. Tap **+ Sign Out Radio**
2. Select a radio from the dropdown — previously used radios appear as `Site ID / Dept ID` pairs
   - If a radio has a logged fault, it shows **⚠ FAULT** in the dropdown
3. Site ID and Department ID are auto-filled when a known radio is selected
   - Alternatively, type a Site ID manually — if it's been used before, Dept ID auto-fills
   - For a brand new radio, type both fields manually
4. Set the condition and add any notes (optional)
5. Tap **Confirm Sign Out**

**Fault warning:** If the last logged action for that radio was a fault, a red warning popup appears showing:
- The radio ID
- Who reported the fault and when
- The fault message they left

The user must either cancel or tap **I Understand — Sign Out Anyway** to proceed.

### Returning a Radio

1. Tap **Return Radio**
2. Select the radio from the list of currently signed-out radios
3. Set the condition on return
4. If there are any issues to report, enter them in the notes field
   - If issues are noted OR condition is set to "Faulty / Not working", the return is logged as a **FAULT** entry
5. Tap **Confirm Return**

### Currently Signed Out

The main view shows all radios currently signed out with:
- Site ID and Department ID
- Who signed it out and when
- Condition at sign-out
- A fault indicator if issues were noted at sign-out

### Full History

A full log of every sign-out, return, and fault entry — newest first. Columns: Action, Radio, By, Time, Condition, Notes.

### Download History

Tap **↓ Download History** to export the full radio log as a CSV file, dated with today's date.

### Site ID / Department ID Pairing

Site ID and Department ID are treated as a fixed pair — if Site ID `S01` always maps to Dept ID `Traffic-1`, the app remembers this from history and auto-fills it every time. You only need to type both fields once for a new radio.

---

## Data Structure

All data is stored in `data.json`:

```json
{
  "crew": [...],
  "radioLog": [...],
  "shifts": {...}
}
```

### radioLog entry fields

| Field | Description |
|---|---|
| `action` | `out` (sign-out), `in` (return), `fault` (return with issues) |
| `siteId` | Site ID of the radio |
| `deptId` | Department ID of the radio |
| `by` | Name of the crew member |
| `time` | Unix timestamp (ms) |
| `condition` | Condition at time of action |
| `notes` | Any issues noted |
| `id` | Unique entry ID (same as timestamp) |

---

## Infrastructure

| Component | Details |
|---|---|
| Frontend | Single `index.html` on GitHub Pages |
| Data store | `data.json` in this repo (GitHub API) |
| Proxy | Cloudflare Worker — handles reads and writes with CORS |
| Polling | Every 8 seconds via proxy (no CDN lag) |
| Auth | None — identity stored in browser `localStorage` |
