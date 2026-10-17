---
name: add-team
description: Add a new agent team to this Command Center. Use when the user says "add a team", "new team", "add my ___ team", or describes a team or system they want on the dashboard.
---

# Add a team

You are adding ONE team to `data/teams.json`. Read `AGENTS.md` first: it has the exact shape and the rules.

## 1. Ask these 4 questions (one message, plain words)
1. What's the team called?
2. What does it do, in one sentence?
3. What are its steps, in order? (Rough is fine. "find ideas, write, check, send" works.)
4. Who approves the work before it goes out? (You, someone else, or nobody.)

If the user already answered some of these, don't ask again.

## 2. Build the team
- Group the steps into 3 to 6 phases, left to right, with 2 to 4 steps each. Name phases with one plain word (Plan, Write, Review, Send).
- Give each phase a different colour from: sky, violet, amber, rose, teal, lime, slate.
- If someone approves, add a step for it with `"badge": "gate"` and put it right before the step that sends or publishes.
- Draw an arrow (edge) from each step to the next one.
- Seats: one seat per job a person or agent does (Writer, Editor...).
- Skills and `sourceOfTruth`: only list a skill if its folder really exists (look in `.claude/skills/` and `~/.claude/skills/`). The example in AGENTS.md is a shape, not a real skill. If none exist yet, use `"skills": []` and `"sourceOfTruth": null`.
- `buildStatus` is `planning` unless the user says it already runs. Set `flowAsOf` to today's date.
- Never invent numbers.

## 3. Write it
- Open `data/teams.json` (a list). Add the new team object at the end. Don't change the other teams.
- Never edit `assets/app.js` or `index.html`.
- Check the file: `python3 -m json.tool data/teams.json > /dev/null && echo ok`.

## 4. Tell the user
In 2 or 3 lines: the team was added, how many phases and steps it has, and where to see it: open the Command Center, go to Agent Teams, and click the team.
