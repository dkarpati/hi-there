# LinkedIn cold outreach, run by Claude

A small set of Claude Code skills that book exploration calls without manual prospecting.

## The system

Three skills, one shared data folder.

| Skill | What it does | When to invoke |
| --- | --- | --- |
| `linkedin-icp` | Defines and refines the ICP. Iterative — push back when profiles don't fit. | Once at setup, then whenever the wrong people are showing up. |
| `linkedin-outreach` | Sources prospects, filters against the ICP, sends a connection request with a personal note in your voice. | Daily. Start at 10/day, raise to 30 once quality is good. |
| `linkedin-followup` | Nudges threads that have gone quiet, in your voice. Stops on soft/hard no. | Every 3 days. |

## Files

```
.claude/skills/
  linkedin-icp/SKILL.md
  linkedin-outreach/SKILL.md
  linkedin-followup/SKILL.md
data/
  icp.md            # the source of truth for who we're targeting
  voice.md          # how you actually write — keep this honest
  config.yaml       # daily cap, approval mode, search URLs
  prospects.jsonl   # append-only log of everyone touched
```

## Getting started

1. Run `linkedin-icp` and answer the prompts. Reject profiles until the ICP is sharp.
2. Paste 3-5 messages you've actually sent into the `linkedin-outreach` skill. It writes `data/voice.md`. You edit it.
3. Add LinkedIn search URLs to `data/config.yaml` under `outreach.search_urls`.
4. Leave `approval_mode: manual`. Run `linkedin-outreach`. Read every note before it goes out.
5. Once the voice is right, flip `approval_mode: auto` and raise `daily_cap` to 30.
6. Schedule `linkedin-followup` to run every 3 days.

## Notes on trust

The whole system is built around a trust ramp. Manual approval first, auto only after the voice is yours. The config controls this — Claude doesn't flip it for you.
