---
name: linkedin-icp
description: Define or refine the Ideal Customer Profile used by LinkedIn outreach. Use when the user wants to describe who they want to talk to, edit targeting criteria, or push back on profiles that don't fit ("not this one, this one"). Writes the canonical ICP to data/icp.md.
---

# LinkedIn ICP

The ICP is the source of truth for every outreach decision. It lives at `data/icp.md` and is read by the `linkedin-outreach` and `linkedin-followup` skills.

## When to invoke

- First-time setup ("here's who I want to talk to")
- Iteration after seeing prospect lists ("no, not founders at agencies, founders at vertical SaaS")
- Adding anti-signals ("skip anyone with 'consultant' in their headline")

## How to run

1. Read `data/icp.md` if it exists. Treat it as the current draft.
2. Ask the user only for what's missing or unclear. Do not re-ask things already in the file.
3. Update the file using the structure below. Keep it tight — this is a prompt, not a doc.
4. After every edit, show the user the diff and confirm before saving.

## ICP structure

```
# ICP

## One-liner
<who they are in one sentence>

## Targets
- Titles: <comma list>
- Seniority: <Founder / VP / Director / IC>
- Company stage: <Pre-seed / Seed / Series A-B / public>
- Company size: <headcount range>
- Industries / verticals: <list>
- Geo: <list or "any">

## Signals (must have ≥1)
- <recent funding, hiring for X role, posted about Y, etc.>

## Anti-signals (auto-skip)
- <agency, consultant, recruiter, etc.>

## Why this person should care
<one sentence on the hook from their POV>
```

## Refinement loop

When the user rejects a profile:
1. Add the specific reason to **Anti-signals** if it's a hard rule.
2. Add a counter-example to **Signals** if it's the inverse.
3. Re-run the rejected profile against the updated ICP and confirm it would now be filtered.
