# Routine: pinterest-queue-flush

**Trigger:** Cron, daily at 8:45am Eastern (`45 8 * * *`) — fires 15 minutes before pin #1 of the day from social-distribute, processes everything else queued for today
**Runtime budget:** ~15 minutes
**Output:** All Pinterest pins scheduled for today (across all queue files) generated, posted via Pinterest API, and marked as fired

**Why it exists:** The `daily-social-distribute` routine queues 4 future pins per URL but only fires the first one immediately. Without this worker, queued pins #2-5 for each URL would never publish. This routine reads all `pin-queue-*.md` files, finds pins scheduled for today, generates their images, and posts them.

**Why 8:45am:** Pinterest pin #1 from social-distribute fires at 9:00am. This worker runs first so all of today's pins (1 fresh + ~4 from prior days' queues) are ready to fire by 9am.

---

## Mission

At steady state (day 5+), this routine fires ~4 queued pins per day across 4 different URLs (yesterday's URL pin #2, day-2's URL pin #3, day-3's URL pin #4, day-4's URL pin #5). Combined with social-distribute's pin #1 of today's URL, that's the 5-pins-per-day-across-5-URLs steady state.

---

## Step 0 — Load TCA skills (mandatory)

Before generating any pin images, load:
- `gpt-image-2-director` — for pin image prompt construction
- `copywriting` — for any pin headline or description tweaks

If a skill won't load: log High-priority Notion task, proceed with built-in defaults.

---

## Step-by-step

### 1. Find all queue files

List all files matching `logs/pin-queue-*.md` in the repo. Each file represents one blog post URL with its pins #2–5 queued.

If zero queue files found: this is normal during the first 4 days of operation. Log a low-priority Notion task ("queue empty, no pins to flush"), exit cleanly.

### 2. Identify pins scheduled for today

For each queue file:
1. Read the pin entries (each entry has a scheduled date)
2. Filter for entries where `scheduled_date == today` AND `status != "fired"`
3. If no pins for today in this file, skip it
4. If pins for today exist, add them to the day's processing list

### 3. For each pin scheduled today

Process in order. For each pin entry:

**Check inbox first:**
- Read the pin's source post slug from the queue file
- Check `inbox/pinterest-pins/` for any sidecar JSON with matching `topic`/`category` from the source post, OR matching `use_with_post_slug`
- If a matching inbox pin exists AND hasn't been used: use it instead of generating
  - Move file + sidecar to `inbox/used/<YYYY-MM>/`
  - Update sidecar with `used_in: <pin URL after posting>`

**Otherwise generate:**
1. Read the pin's queued data (angle, headline draft, GPT Image 2 prompt, description)
2. Generate the pin image at 1000×1500 (2:3) via OpenAI Images API using the queued prompt
3. Verify image dimensions before proceeding (Pinterest penalizes wrong ratios)

**Post to Pinterest:**
1. Upload image and pin metadata via Pinterest API
   - Title: from queued headline
   - Description: from queued description  
   - Link: source post URL with UTM tracking `?utm_source=pinterest&utm_medium=social&utm_campaign=<post-slug>&utm_content=pin-<N>`
   - Board: based on category — see board mapping below
2. Capture the returned pin URL

**Mark as fired in the queue file:**
- Update the pin entry: `status: fired`, `fired_at: <timestamp>`, `pin_url: <URL>`
- Save the queue file back

### 4. Board mapping

Each pin posts to the board matching the source post's category:

| Category | Pinterest Board |
|---|---|
| Market Insights | "Market Commentary" |
| Social Security | "Social Security Strategy" |
| Retirement Planning | "Retirement Planning" |
| Bucket Planning | "Bucket Planning Method" |
| Financial Education | "Personal Finance" |
| Trading & Markets | "Trading & Markets" |

If a board doesn't exist yet, log a Medium-priority Notion task ("Pinterest board missing: <name>") and post to the default "Thomas Clark Advisor" board as fallback.

### 5. Cleanup completed queue files

If all pins in a queue file are now `status: fired`, move the file from `logs/pin-queue-*.md` to `logs/pin-queue-archive/<YYYY-MM>/`. This keeps the active `logs/` directory clean.

### 6. Log to Notion

Create one page in the Operations Hub:
- **Task:** `Pinterest queue flush — <date>`
- **Project:** `TCA`
- **Status:** `Done`
- **Type:** `Marketing`
- **Owner:** `Claude`
- **Priority:** `Low` (this is a routine maintenance task, not noteworthy unless something fails)
- **Notes:**
  - Number of queue files processed
  - Number of pins fired today
  - Per-pin: source post slug, pin URL, board, whether from inbox or generated
  - Any pins that failed to fire and why
  - Queue files archived after completion
- **Source Chat:** `pinterest-queue-flush routine — <date>`

### 7. Send the daily summary

To Slack `#tca-content-ops` (or email):

```
✅ Pinterest queue flush complete — <date>

Pins fired today: <N>
Across URLs: <N>

Sources:
  - <post-slug-1> (pin <X> of 5)
  - <post-slug-2> (pin <X> of 5)
  - <post-slug-3> (pin <X> of 5)
  - <post-slug-4> (pin <X> of 5)

Inbox pins used: <N>
Generated fresh: <N>

Issues: <none | list>
Notion log: <URL>
```

Combined with social-distribute's pin #1 for today's new URL, total Pinterest activity today is `<N+1>` pins.

---

## Failure modes

| What fails | What to do |
|---|---|
| No queue files found | Normal during first 4 days. Log Low-priority "queue empty." Exit cleanly. |
| Queue file malformed (can't parse) | Skip that file. Log High-priority Notion task with file path so Thomas can inspect. |
| Skill loading fails | Use built-in defaults. Log High-priority. |
| Pinterest API rate limit hit | Pause 60 seconds, retry once. If still rate-limited, push remaining pins to fallback scheduler (Metricool). Log High-priority. |
| Image generation fails for one pin | Skip that pin, mark as `status: failed` in queue with error message. Continue with other pins. Log Medium-priority Notion task. |
| Image generation fails for entire batch | OpenAI API likely down. Skip all pins for today, leave queue intact, log Critical Notion task. They'll retry tomorrow. |
| Posting fails for one pin | Mark as `status: failed`, log Medium-priority. Continue with other pins. |
| Board doesn't exist | Post to fallback board, log Medium-priority to create the missing board. |
| Notion logging fails | Send Slack/email summary anyway. Email Thomas about Notion failure separately. |

---

## What this routine never does

- Generate pin metadata from scratch (titles, descriptions) — that's social-distribute's job. This routine only fires what's already queued.
- Fire pins scheduled for any date other than today
- Re-fire pins already marked `status: fired`
- Skip the inbox check before generating
- Post to a single board for everything (always category-matched)
- Delete queue files (only archives completed ones)

---

## Pin queue file format (reference)

For context, the `pin-queue-*.md` files written by social-distribute look like this:

```yaml
# pin-queue-the-real-cost-of-claiming-social-security-at-62.md

source_post:
  slug: the-real-cost-of-claiming-social-security-at-62
  url: https://thomasclarkadvisor.com/the-real-cost-of-claiming-social-security-at-62
  category: Social Security
  published_date: 2026-04-30

pins:
  - pin_number: 2
    angle: "different audience hook — adult kids helping parents"
    headline: "How to Help Your Parents Make the Right Social Security Choice"
    gpt_image_2_prompt: "<full prompt ready to fire>"
    description: "<200-500 char Pinterest description>"
    scheduled_date: 2026-05-01
    scheduled_time: "09:00 ET"
    status: queued
    fired_at: null
    pin_url: null

  - pin_number: 3
    angle: "different value proposition — survivor benefits angle"
    headline: "Why Surviving Spouses Lose 30% — And How to Plan Around It"
    gpt_image_2_prompt: "<full prompt>"
    description: "<description>"
    scheduled_date: 2026-05-02
    scheduled_time: "09:00 ET"
    status: queued
    fired_at: null
    pin_url: null

  # ... pins 4 and 5 follow same structure
```

This routine reads the file, processes today's entries, updates `status`/`fired_at`/`pin_url`, and saves back.
