# Automation Center

A dependency-free static port of the Command Center Automations mini-app. It is plain HTML, CSS, and JavaScript. There is no build step, no npm install, no CDN, no auth, no database, and no server route.

Open `http://localhost:8077/?demo` to see the sample data from `data/EXAMPLES.json`. Without `?demo`, the page reads the production files:

- `data/automations.json`
- `data/teams.json`
- `data/executions.json`

Those production files intentionally start as empty arrays.

**Adding a team with Claude or Codex:** just say "add a team for my newsletter" (or any system). `AGENTS.md` (Codex reads it; `CLAUDE.md` points Claude to it) has the shape and the rules, and the `add-team` skill (`.claude/skills/` and `.agents/skills/`) asks you 4 quick questions and writes the team into `data/teams.json` for you.

## The One Data Rule

Stats are honest or absent. Use a real number when you can back it up. Use `null` when you cannot. The UI renders `null` as `—`; it never turns an unknown value into `0`.

## Automations Shape

`data/automations.json` is an array of automation records:

```json
{
  "id": "content-poster",
  "name": "Content auto-poster",
  "description": "What this automation does.",
  "area": "content",
  "trigger": { "type": "cron", "schedule": "*/20 * * * *" },
  "active": true,
  "stats": {
    "lastRunAt": "2026-09-16T12:20:00-04:00",
    "lastRunStatus": "success",
    "runCount": 248,
    "failureCount": 2,
    "failureRate": 0.008,
    "avgRunMs": 882,
    "extras": [{ "label": "Posted 7d", "value": "31" }]
  },
  "flow": { "nodes": [], "edges": [] }
}
```

`trigger` can be:

- Cron: `{ "type": "cron", "schedule": "0 13 * * *" }`
- Event: `{ "type": "event", "source": "proposal.responded" }`
- Webhook: `{ "type": "webhook", "path": "/api/payhip/webhook" }`

## Flow Shape

The detail page draws a read-only n8n-style canvas from each record’s `flow` block:

```json
"flow": {
  "topologySource": "where this diagram came from",
  "nodes": [
    {
      "id": "cron",
      "x": 0,
      "y": 0,
      "icon": "clock",
      "title": "Cron */20",
      "stat": "due-slot scan",
      "tone": "accent",
      "hint": "optional second line",
      "detail": "Shown when someone clicks the node.",
      "small": false,
      "pill": { "label": "ON", "tone": "accent" }
    }
  ],
  "edges": [
    {
      "id": "e-cron-next",
      "source": "cron",
      "target": "next",
      "label": "dispatch",
      "animated": true,
      "dashed": false
    }
  ]
}
```

Use the same spacing as the source app: normal nodes are 288px wide, small nodes are 214px wide, and the source graph convention is about `300` x-units per column and `110` y-units per row. The renderer scales positions the same way the real canvas does, so do not pack nodes at 200-250px columns.

`animated: true` means that hop is genuinely live. Use `dashed: true` for an off or not-yet-enabled path. Animation is data, not decoration.

## Agent Teams Shape

`data/teams.json` is an array of team records, not a department summary grid:

```json
{
  "id": "upwork-team",
  "name": "Upwork Team",
  "command": "/upwork-team",
  "department": "sales",
  "teamType": "Sales Team",
  "buildStatus": "building",
  "purpose": "What this team does.",
  "notionUrl": "https://app.notion.com/...",
  "sourceOfTruth": "~/.claude/skills/upwork-team/SKILL.md",
  "flowAsOf": "2026-09-10",
  "seats": [{ "name": "Job Scout", "type": "Scout" }],
  "skills": [
    {
      "name": "Upwork Application",
      "command": "/upwork-application",
      "seat": "Application Tracker & Time Keeper",
      "owns": "What this skill owns."
    }
  ],
  "flow": { "nodes": [], "edges": [] }
}
```

The Agent Teams tab renders registry-style rows grouped by department. Clicking a team opens the team’s flow. The `Seats & skills` tab lists seats and skills, and each seat can be clicked for a focused seat detail view.

### A team's system, in phases

When a team's system gets built, give its `flow` a list of `phases`. Each phase is drawn as a coloured, labelled box, its steps stack inside it, and arrows join the steps. You can drag to pan, use + / − / ⤢ (fit) to zoom, and click a step to read its detail. A team with no flow yet shows a "No system mapped yet" card, so a new team can start empty.

```json
"flow": {
  "phases": [
    { "id": "plan",    "label": "Plan",    "color": "violet" },
    { "id": "create",  "label": "Create",  "color": "amber" },
    { "id": "publish", "label": "Publish", "color": "teal" }
  ],
  "nodes": [
    { "id": "pick",   "phase": "plan",    "label": "Pick today's idea", "sub": "One idea, one angle", "icon": "sparkle" },
    { "id": "script", "phase": "create",  "label": "Script + captions", "icon": "log", "detail": "Shown when the step is clicked." },
    { "id": "record", "phase": "create",  "label": "Record", "badge": "you" },
    { "id": "post",   "phase": "publish", "label": "Post + confirm", "icon": "send" }
  ],
  "edges": [
    { "from": "pick", "to": "script" },
    { "from": "script", "to": "record" },
    { "from": "record", "to": "post", "label": "approved" },
    { "from": "post", "to": "pick", "label": "learn", "dashed": true }
  ]
}
```

- `phases` are laid out left to right in the order you list them, wrapping after `perRow` (default 6). Give a phase `x` and `y` (pixels) to pin it somewhere else.
- `color` is one of `teal`, `sky`, `violet`, `amber`, `rose`, `lime`, `slate`, or any hex like `#ff8800`.
- A node's `phase` says which box it sits in; nodes stack in the order you list them. `sub`, `icon`, `badge` and `detail` are optional.
- An edge can point at a step or at a whole phase (`"to": "publish"`). An arrow that goes backwards loops under the boxes, and `dashed: true` marks a path that isn't switched on yet.
- A flow without `phases` still uses the free-form `x`/`y` layout described in Flow Shape above.

See the Content Studio team in `data/EXAMPLES.json` (open `?demo`) for a full example.

## Executions Shape

`data/executions.json` is an array of run rows:

```json
{
  "id": "run-001",
  "automationId": "content-poster",
  "automationName": "Content auto-poster",
  "status": "success",
  "startedAt": "2026-09-16T12:20:00-04:00",
  "runMs": 882,
  "detail": "3 targets claimed · 3 posted"
}
```

`status` is `success`, `error`, `skipped`, or `running`. `runMs` may be `null`; it renders as `—`.

## Routes

The static page uses hash routes so it works on any static host:

- `#/automations`
- `#/automations/:id`
- `#/automations/:id/executions`
- `#/executions`
- `#/agent-teams`
- `#/agent-teams/:id`
- `#/agent-teams/:id/seats`
- `#/agent-teams/:id/members/:seat`

## Deploying

Serve the folder as static files. The included `vercel.json` disables caching for `/data/*` so JSON edits show up immediately.
