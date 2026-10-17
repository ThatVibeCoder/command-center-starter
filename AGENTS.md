# Command Center: notes for Claude / Codex

This is a plain web page. No build step, no packages. Open `index.html` with any static server (`python3 -m http.server`), and add `?demo` to the URL to see sample data.

## The three data files (the only files you should change)
- `data/teams.json`: your agent teams. Each team has seats, skills, and a flow (its system, drawn in phases).
- `data/automations.json`: jobs that run on their own (a schedule, a webhook, an event).
- `data/executions.json`: a log of those jobs' runs.
Each file is a list: `[ {...}, {...} ]`. An empty list `[]` is fine.

## Rules
- To add a team: add ONE object to the list in `data/teams.json`. Never edit `assets/app.js` or `index.html`.
- Only write numbers you really have. If you don't know one, use `null` (it shows as a dash). Never make stats up.
- A flow is phases, left to right. Use 3 to 6 phases, with 2 to 4 steps in each.
- Phase colours: `sky`, `violet`, `amber`, `rose`, `teal`, `lime`, `slate`.
- `id` is short, lowercase, with dashes (`newsletter-team`). Every step `id` is unique inside its team.
- Keep the JSON valid: double quotes, no trailing commas. Check it with `python3 -m json.tool data/teams.json`.
- Need help? The `add-team` skill walks you through it.

## One full team (copy this shape)
```json
{
  "id": "newsletter-team",
  "name": "Newsletter Team",
  "command": "/newsletter-team",
  "department": "marketing",
  "teamType": "Marketing Team",
  "buildStatus": "planning",
  "purpose": "Writes and sends one newsletter a week.",
  "sourceOfTruth": null,
  "flowAsOf": "2026-10-17",
  "seats": [{ "name": "Writer", "type": "Draft & Approve" }],
  "skills": [],
  "flow": {
    "phases": [
      { "id": "plan", "label": "Plan", "color": "violet" },
      { "id": "write", "label": "Write", "color": "amber" },
      { "id": "send", "label": "Send", "color": "teal" }
    ],
    "nodes": [
      { "id": "topic", "phase": "plan", "label": "Pick the topic", "sub": "One idea", "icon": "sparkle" },
      { "id": "draft", "phase": "write", "label": "Write the draft", "icon": "form" },
      { "id": "ok", "phase": "write", "label": "You approve", "badge": "gate", "icon": "check" },
      { "id": "go", "phase": "send", "label": "Send it", "icon": "send" }
    ],
    "edges": [
      { "from": "topic", "to": "draft" },
      { "from": "draft", "to": "ok" },
      { "from": "ok", "to": "go", "label": "approved" }
    ]
  }
}
```
Field notes: `skills` lists only skills that really exist (e.g. a folder in `.claude/skills/`), as `{ "name", "command", "seat", "owns" }`; if there are none yet, use `[]`. `sourceOfTruth` is the file where the team's instructions live, or `null`. Never invent either. `department` is one of `sales`, `marketing`, `operations`, `engineering`, `finance`, `customer_support`, `product`. `buildStatus` is `planning`, `building`, `active` or `archived`. A step's `sub`, `icon`, `badge` and `detail` (text shown on click) are optional. Icons: `terminal`, `search`, `form`, `check`, `send`, `mail`, `clock`, `trend`, `sparkle`, `user`, `shield`, `log`, `db`, `route`, `browser`. If the system isn't built yet, use `"flow": { "phases": [], "nodes": [], "edges": [] }`.
