# WAY Attendance Compliance Dashboard

Internal dashboard for auditing greytHR swipe exports against the CS roster.
Hosted on GitHub Pages, no backend. Everything runs in the browser; swipe
files never leave the user's machine.

**Live URL:** https://kiranfrnndz.github.io/way-attendance-dashboard/

---

## Files in this repo

| File | What it is | How often it changes |
|---|---|---|
| `index.html` | The dashboard app (HTML + CSS + JS in one file) | Rarely — only for feature changes / bugfixes |
| `roster.json` | Agents, shifts, roles, aliases, exceptions | ~Every 3 months as roster rotates |
| `README.md` | This file | As needed |

On page load, `index.html` fetches `roster.json`. If the fetch fails (e.g.
opened as a local file), the dashboard falls back to a built-in seed roster
so it still works.

---

## How to update the roster

The whole point of this setup: **you never edit the HTML to update people.**
All roster changes happen in `roster.json`, which you can edit directly in
the GitHub browser UI.

### Step-by-step

1. Go to the repo: https://github.com/kiranfrnndz/way-attendance-dashboard
2. Click **`roster.json`**
3. Click the **✏ pencil icon** (top-right of the file view)
4. Make your edits (see examples below)
5. Scroll to the bottom → commit message like `Roster update Jan 2027` → **Commit changes**
6. Wait ~30 seconds for GitHub Pages to redeploy
7. Users refresh the live URL → they see the new roster

No file sharing. No code touched. Same URL as always.

### What you can change in roster.json

#### Add a new agent
Add a new object inside the `"agents": [...]` array:
```json
{ "name": "New Joiner Name", "shift": "S2", "weekoff": "SAT-SUN", "team": "CS English" }
```
Valid `weekoff` values use the day codes `SUN MON TUE WED THURS FRI SAT`
joined with a dash, two days: `SUN-MON`, `WED-THURS`, etc.

#### Remove an agent (resignation / transfer out)
Delete their object from the `"agents"` array. Mind the commas — JSON doesn't allow a trailing comma.

#### Change an agent's shift permanently
Edit their `"shift"` value in the `"agents"` array.

#### Change an agent's shift temporarily (e.g. 2-month cover)
Add a dated exception in `"exceptions": [...]`:
```json
{ "person": "Agent Name", "from": "2027-01-15", "to": "2027-03-15", "type": "shift", "shift": "S3", "note": "Covering night shift" }
```

#### Add a new shift code (e.g. a 5th main shift)
Add to the `"shifts": [...]` array:
```json
{ "code": "S9", "istStart": "10:00", "istEnd": "19:00", "label": "New Shift", "istLabel": "10:00 AM – 07:00 PM IST", "color": "#60a5fa" }
```
Pick a unique code (S9, S10, …) and a hex color. The dashboard automatically
picks it up: shift pills, legend rail, and the exceptions modal dropdown
all update themselves.

#### Promote someone to Team Lead
Add their exact name to the Team Lead list in `"roles"`:
```json
"roles": {
  "Leadership": ["Joyson Fernandez","Anju Mareeta Lean"],
  "Team Lead":  [..., "New TL Name"]
}
```

#### Handle a name mismatch between greytHR and roster
If greytHR exports a name that doesn't match the roster, add an alias:
```json
"aliases": {
  "mdjaweedk": "Javed Khan"
}
```
Key = greytHR's rendering (lowercase, no spaces, letters only — the dashboard normalizes before matching).
Value = exact roster name.

### If you break the JSON
GitHub's web editor shows red-underline errors on invalid JSON before you
commit. Easy ones:
- Missing comma between objects
- Trailing comma after the last item
- Mismatched braces or brackets
- Quotes not closed

If a bad commit lands, the dashboard will silently fall back to the seed
roster baked into the HTML — not great, but not broken. Open the previous
good commit of `roster.json` on GitHub and copy-paste it over.

---

## Current roster snapshot (as of 7 Oct 2026)

- **67 agents** across 8 shift codes
- **58 CS English + 9 CS Spanish**
- **2 Leadership** (Joyson, Anju) · **9 Team Leads**
- **4 main shifts** (S1–S4) + **4 Spanish-team shifts** (S5–S8)
- 1 historical override (Jithin S, July 2026)
- 2 current temp shift exceptions (Aswini, Rashmika from 7 Oct)
- 7 Spanish night rotation blocks (Sattar / Uppara / Javed / Siddarth, 2-week cycles through 5 Jan 2027)

---

## Rules enforced by the dashboard

- **On-time login:** ≥ 1 min late → `LATE` flag
- **Break:** ≤ 60 min allowed; break calculation now clips to the shift
  window so pre-shift step-out doesn't eat the budget
- **Office hours:** ≥ 8 hrs required
- **TL / Leadership break waiver:** if they put in 8+ hrs in office, break
  over 60 min is waived (`BREAK_WAIVED`)
- **OT:** if someone stays ≥ 45 min past shift end, `STAYED OT` info flag
- **Weekoff work:** working on a rostered week-off day flags `WEEK-OFF WORK`
- Exceptions in `roster.json` (and user-added via the UI) suppress the
  corresponding flag with an "approved" tag

---

## Deployment

Pushed via GitHub Pages from the `main` branch, root folder.
No build step — HTML is served as-is.
First deploy was 7 Oct 2026.
