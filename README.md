# tca-content-ops

Autonomous daily content pipeline for **Thomas Clark Advisor**.

Four Claude Code Routines run from this repo on cloud infrastructure (no local machine required):

- **`daily-blog-publish`** — fires 2:30am ET daily, produces one publish-ready post scheduled for 8:00am. Does NOT trigger email broadcasts.
- **`daily-social-distribute`** — fires 8:30am ET daily, adapts the morning's post into platform-native social content across FB, IG (feed + carousel), LinkedIn, and Pinterest; queues 4 future Pinterest pins per URL
- **`pinterest-queue-flush`** — fires 8:45am ET daily, processes queued Pinterest pins from prior days (generates images, posts via Pinterest API); produces the steady-state ~5 pins/day across rotating URLs
- **`weekly-newsletter-digest`** — fires Sundays 7:00am ET, sends one newsletter at 8:00am ET to TCA Newsletter Subscribers; lead essay + recap of week's posts

## Quick orientation

- **`CLAUDE.md`** — shared context for all four routines: brand identity, voice, planning philosophy, compliance posture, tools available, inbox folder system
- **`routines/blog-publish.md`** — the blog routine's full step-by-step
- **`routines/social-distribute.md`** — the social routine's full step-by-step
- **`routines/pinterest-queue-flush.md`** — the Pinterest queue worker
- **`routines/weekly-newsletter-digest.md`** — the weekly newsletter routine
- **`inbox/`** — drop pre-made assets here (images, carousels, pins) for routines to pick up automatically
- **`voice/`** — voice samples and anti-patterns the routines reference
- **`content-calendar.md`** — topic rotation
- **`assets/`** — disclaimers, author bio, image prompt patterns

## Setup checklist

1. Push this repo to GitHub (private)
2. Create the Code Routines in Claude Code Desktop → Routines → New Routine → Remote
3. Configure environment variables / secrets for: WordPress app password, OpenAI API key, GHL API key + webhook URLs, Meta Graph token, LinkedIn token, Pinterest token, Notion MCP credentials
4. Run each routine manually once to verify end-to-end before activating the schedule
5. Watch the first week — check Notion logs after each run, tighten prompts based on what shows up

## When to edit

- **Voice drifts** → edit `voice/tca-voice-examples.md` and `voice/anti-patterns.md`
- **Compliance posture changes** → edit `CLAUDE.md` § Compliance posture
- **New category or new audience segment** → edit `CLAUDE.md` § Content categories
- **Routine misbehaves** → edit the relevant `routines/*.md` directly
- **New platform added (e.g., TikTok, Threads)** → add a section to `routines/social-distribute.md`

The routines re-read these files on every run. Changes take effect on the next scheduled trigger.
