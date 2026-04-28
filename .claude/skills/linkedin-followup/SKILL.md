---
name: linkedin-followup
description: Follow up on LinkedIn threads that have gone quiet. Use when the user wants to nudge stale conversations (default 3 days of silence). Reads data/prospects.jsonl and data/voice.md, opens each quiet thread in Claude Cowork, drafts a reply in the user's voice, and sends per the approval mode in data/config.yaml.
---

# LinkedIn follow-up

Runs every 3 days by default. Picks up threads where the last message was sent by the user (or the prospect, depending on stage) and nothing has happened since.

## Inputs

- `data/prospects.jsonl` — required.
- `data/voice.md` — required.
- `data/config.yaml` — `followup.quiet_days` (default 3), `followup.max_per_run` (default 20), `followup.max_touches` (default 3 nudges then stop), `approval_mode`.

## Run loop

1. Read all prospects with `status` in `{accepted, replied}` and `last_event_at` older than `quiet_days`.
2. Filter out anyone with `touches >= max_touches` (count of follow-ups already sent).
3. For each remaining prospect, in oldest-first order, up to `max_per_run`:
   - Open the thread in Claude Cowork.
   - Read the last 6 messages of context.
   - Decide the next move using the **Decision tree** below.
   - Draft the reply per the **Reply rules**.
   - In `manual` mode, show the user thread context + draft and wait. In `auto`, send.
   - Update the prospect's row in `prospects.jsonl`: bump `touches`, update `last_event_at`, append to `note_history`.
4. Print a summary: how many nudged, how many skipped, how many closed out.

## Decision tree

- **Connection accepted, no message exchanged** → send a short opener tied to their work. Treat this as touch #1.
- **You messaged, no reply, < max_touches** → one short nudge. Reference the original message obliquely, don't repeat it.
- **They replied positively, conversation stalled** → propose a concrete next step (15-min call, share a doc, intro to a customer). Pick one, not all three.
- **They replied with a soft no / "not now"** → mark `status: closed_lost`, `reason: not_now`. Do not nudge.
- **They replied with a hard no / unsubscribe / negative tone** → mark `status: closed_lost`, `reason: declined`. Do not nudge.
- **`touches >= max_touches`** → mark `status: closed_lost`, `reason: no_response`. Stop.

## Reply rules

- Voice first. Same rules as `linkedin-outreach` — re-read `data/voice.md` before drafting.
- One sentence, two max. Phone-thumb length.
- One ask per message. If proposing a meeting, propose one option, not three.
- Never apologize for following up. Never say "circling back" or "bumping this".
- If the thread already has a meeting link in it, don't propose another one — just point at it.

## Trust ramp

The user starts in `manual` approval. They flip the config to `auto` when they trust the voice. Don't suggest flipping it — they decide.
