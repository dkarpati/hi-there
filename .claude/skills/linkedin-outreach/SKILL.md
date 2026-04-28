---
name: linkedin-outreach
description: Run a daily LinkedIn cold-outreach pass. Use when the user wants to find new prospects matching their ICP, send connection requests, and write a personal note in their voice. Reads data/icp.md and data/voice.md, navigates LinkedIn via Claude Cowork (browser), writes to data/prospects.jsonl. Respects the daily cap in data/config.yaml.
---

# LinkedIn outreach

One pass = (1) source N new prospects, (2) filter against ICP, (3) connect with a personal note, (4) log.

## Inputs

- `data/icp.md` — required. If missing, stop and tell the user to run `/linkedin-icp` first.
- `data/voice.md` — required. If missing, run the **Voice capture** step below before sending anything.
- `data/config.yaml` — daily cap, approval mode, search URLs.
- `data/prospects.jsonl` — append-only log of everyone touched. De-dupe against this on every pass.

## Run loop

1. Read config. Stop if today's count in `prospects.jsonl` already hits `daily_cap`.
2. Read ICP and voice.
3. For each search URL in config:
   - Open in Claude Cowork.
   - Walk results top-to-bottom. For each profile:
     - Skip if already in `prospects.jsonl` (any status).
     - Open the profile. Pull: name, headline, current company, location, recent activity (posts/comments in last 30d), mutual context.
     - Score against ICP. Reject on any anti-signal. Require ≥1 signal.
     - If accept: draft a connection note (rules below).
     - In `approval_mode: manual`, show the user the note + profile summary and wait. In `approval_mode: auto`, send.
     - Click Connect → Add a note → paste → send.
     - Append to `prospects.jsonl`.
4. Stop when `daily_cap` reached or search exhausted. Print a one-line summary.

## Note rules

- Voice first. Re-read `data/voice.md` before every draft. The note must read like the user typed it on their phone.
- ≤ 280 chars (LinkedIn's limit on connection notes is 300; leave room).
- One specific hook per note — pulled from the prospect's recent activity, role change, or post. No "I came across your profile."
- No pitch. The goal is a connection, not a meeting. The pitch comes after they accept.
- Never use: "synergy", "circle back", "quick chat", "picking your brain", em-dashes used like commas, or any phrase the user wouldn't say out loud.
- If there's no real hook, skip the prospect. Don't manufacture one.

## Voice capture (first run only)

If `data/voice.md` is missing or thin:
1. Ask the user for 3-5 examples of messages they actually sent on LinkedIn / email / Slack.
2. Extract: sentence length, contractions, hedges they use, hedges they never use, opener patterns, sign-off, capitalization habits (e.g. always lowercase i?), emoji usage.
3. Write `data/voice.md` and show it to the user. They edit it; you don't argue.

## Prospect log format (`prospects.jsonl`)

One JSON object per line:
```json
{"id":"<linkedin-slug>","name":"<full name>","title":"<headline>","company":"<co>","url":"<profile url>","sourced_at":"<iso ts>","note_sent":"<message>","status":"connect_sent","last_event_at":"<iso ts>"}
```

`status` values: `connect_sent`, `accepted`, `replied`, `meeting_booked`, `closed_lost`, `skipped`.

## Daily cap & ramp

Default cap is 10 to start. Once the user confirms quality looks right, raise to 30. Never auto-raise — the user controls this in `data/config.yaml`.

## Failure handling

- If LinkedIn shows the weekly invite limit warning, stop the run and log it. Don't retry today.
- If a profile fails to load twice, skip and log as `status: skipped, reason: load_failed`.
- If Cowork loses the session, ask the user to re-auth. Do not attempt to re-auth automatically.
