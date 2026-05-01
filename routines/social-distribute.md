# Routine: daily-social-distribute

**Trigger:** Cron, daily at 8:30am Eastern (`30 8 * * *`) — 30 minutes after blog goes live
**Runtime budget:** ~30 minutes
**Output:** Platform-native social posts published or scheduled across Facebook, Instagram (feed + carousel), LinkedIn, and Pinterest. Engagement-first content calibrated per platform's algorithm reality.

---

## Mission

Take today's blog content and produce **engagement-worthy native posts** for each platform. Each platform has different rules — what works on LinkedIn fails on Instagram and vice versa. The routine respects this.

The routine reads `logs/<date>-social-hooks.md` (produced by the blog routine) rather than re-analyzing the full post. This keeps blog and social tightly aligned and saves tokens.

---

## Step 0 — Load TCA skills (mandatory)

Before generating any content, load:
- **`tca-social-media-standards`** — defines per-platform standards, content types, frequencies, visual rules, copy rules, hashtag rules, quality gates. Replaces the older `social-media-content-standards` skill. Brand palette + Pinterest section + per-platform link strategy all live here.
- **`tca-compliance-check`** — for any social post touching financial topics. Load now, invoke per platform post in Step 8 before scheduling.
- `marketing-psychology` — engagement framing
- `copywriting` — headlines and CTAs
- `gpt-image-2-director` — for all image and carousel slide prompts

If `tca-social-media-standards` or `tca-compliance-check` fails to load: log Critical, stop the routine. Do not publish without standards or compliance check.
If other skills fail: log High-priority, proceed with built-in defaults from this file.

---

## The per-platform link/no-link strategy (research-backed)

Different platforms penalize external links very differently. The routine respects this rather than applying one rule everywhere.

### Facebook
- **Default: link in 2 posts per week, standalone 3 per week**
- Reach is already low (~1.65% of followers). Links don't get aggressively penalized but don't get pushed either.
- The smart play: use standalone days to build the algorithm signal that this Page is worth showing, so when link days fire they reach a warmer audience.
- Every link post must deliver value in-feed before the link — first 2 lines must hook before the "see more" cutoff
- Standalone days lean into shareability — content people want to send to a friend
- Suggested cadence: link Mon + Wed, standalone Tue + Thu + Fri (adjust if `content-calendar.md` overrides)

### LinkedIn
- **Default: link in 1–2 posts per week, standalone 3–4 per week**
- LinkedIn applies an estimated ~60% reach penalty to posts with external links. Putting links in the first comment is also detected.
- Most days, deliver the full insight natively in the post. Standalone is the default mode.
- Link days are reserved for posts where the blog genuinely adds depth the LinkedIn post can't (long charts, downloadable resources, full case studies)
- For link days: include the link in body, not as comment

### Instagram (feed)
- **Default: every day works — link/no-link distinction is mostly irrelevant**
- Links live in bio, not in posts, so there's no platform penalty either way
- What matters: shareability (DM shares weight 3-5x higher than likes), watch time, save-worthiness
- Caption strategy: deliver the value, mention "link in bio" if the blog deepens the topic, otherwise just let the post stand alone

### Instagram (carousel)
- **Default: every weekday gets a carousel** — IG carousels are the highest-engagement format on the platform (1.92% vs 1.74% Reels vs 1.26% photo)
- 8–10 slides is the sweet spot (peak reach observed around slide 13 for longer sequences)
- Aspect ratio: 1080×1350 (4:5 portrait) — NOT 1080×1080 square. Portrait dominates mobile screen real estate.
- Last slide is the CTA: "save this for later" or "link in bio for the full breakdown"

### Pinterest
- **Default: every pin links — that's the entire point of Pinterest**
- Pinterest is a search engine, not a social network. Pin = traffic vehicle.
- Generate 5 distinct pin designs per blog post (top 1% of pins drive 50%+ of traffic — design variety is how you increase odds of one going viral)
- Aspect ratio: 1000×1500 (2:3) — Pinterest's algorithm penalizes other ratios
- **Critical rule:** wait at least 24 hours between pins to the same URL. Never publish all 5 pins to the same URL on the same day — triggers spam filters.
- The routine schedules one pin today, then queues the other 4 across the next 4 days

---

## Step-by-step

### 1. Read today's social hooks file

Read `logs/<date>-social-hooks.md` from the repo. Contains:
- Lead insight (strongest standalone takeaway)
- Secondary insights (for IG slides and standalone posts)
- Contrarian angle (if any)
- Carousel arc (8–10 beats — note the updated count)
- Pinterest pin angles (5 distinct framings)

If the file doesn't exist: blog routine likely failed. Log to Notion as Medium priority. Do not invent content. Exit cleanly.

### 2. Determine today's per-platform mix

Use the rules above. Override only if `content-calendar.md` specifies otherwise (e.g., big launch day, force all platforms to link).

### 3. Generate Facebook post

**Length:** 100–250 words.

**Structure:**
- Open with the lead insight from the hooks file. First 2 lines must hook before the "see more" cutoff (~80 chars on mobile).
- 1–2 paragraphs developing the idea
- If link day: end with one line ("Full breakdown here") + URL
- If standalone day: end with engagement prompt — no link
- No hashtags (FB doesn't reward them)

### 4. Generate LinkedIn post

**Length:** 200–500 words. LinkedIn rewards substance and dwell time.

**Structure:**
- Strong opener — ideally a contrarian or specific claim. Use the contrarian angle from the hooks file if present.
- Short paragraphs, 1–3 sentences each (mobile readability)
- Professional tone but recognizably Thomas — direct, opinionated, specific
- 1–3 hashtags max at the end (more than 5 triggers spam signals)
- If link day: include link in body text (not in comment — that workaround is now also penalized)
- If standalone day: end with a question that invites real responses

Schedule for 10:00am Eastern.

### 5. Generate Instagram feed caption

**Length:** 150–300 words.

**Structure:**
- Hook line (first line, before the "more" cutoff at 125 characters)
- Line break
- Develop in short paragraphs, 1–3 sentences each, with line breaks
- Conversational, more personal than FB
- Keyword-rich (IG SEO matters more than hashtags now)
- 5–10 hashtags at end (mix broad + specific) — for categorization, not discovery
- "Link in bio" callout at end — Linktree manages the actual landing

**Image:** 1080×1350 portrait (4:5) for IG feed.

**Check `inbox/ig-feed-images/` first.** If a sidecar matches by `use_with_post_slug` or `topic`/`category`, use that image. Move to `inbox/used/<YYYY-MM>/` after upload. If no match, generate via GPT Image 2.

### 6. Generate the IG carousel (the engagement workhorse)

**Slide count:** 8–10 slides (sweet spot per 2026 IG algorithm data).

**Aspect ratio:** 1080×1350 (4:5 portrait) for every slide.

**Carousel principles:**
- Slide 1 must stop the scroll. Strong visual + ≤12 word headline.
- Each slide delivers one idea. Don't crowd.
- Visual consistency across all slides — same brand palette, typography, layout grid.
- Slides 2 through N-1: build, support, develop the idea
- Final slide is the CTA: "save this for later" + brand mark, or "link in bio for full breakdown" if it's a link day
- Total reading time across all slides: 30–60 seconds

**Generation pipeline for each carousel:**

**First: check `inbox/ig-carousels/` for a pre-made carousel set matching today's post.**

A carousel set is a subfolder (e.g., `inbox/ig-carousels/bucket-planning-101/`) containing slide images named `slide-1.png` through `slide-N.png` plus a `set.json` sidecar describing the set:

```json
{
  "type": "ig_carousel",
  "topic": "bucket planning basics",
  "category": "Bucket Planning",
  "use_with_post_slug": null,
  "slide_count": 9,
  "caption_draft": "<optional pre-written caption>"
}
```

If a matching set exists: use those slides directly. Use `caption_draft` if present, otherwise generate the caption per the rules below. Move the entire subfolder to `inbox/used/<YYYY-MM>/` after publishing.

**If no matching set in inbox, generate fresh:**

For each of the 8–10 slides in the carousel arc from the hooks file:
1. Compose the slide text (matching slide's role — hook, build, insight, example, payoff, action, CTA)
2. Build a structured GPT Image 2 prompt using the carousel template in `assets/image-prompts/gpt-image-2-style-guide.md`
3. Generate the slide image at 1080×1350 with slide text as text overlay (GPT Image 2 handles text rendering well — this is exactly its strength)
4. Save to temp directory: `slide-1.png` through `slide-N.png`

**Brand consistency rules for carousels:**
- Same background treatment across all slides in a single carousel
- Same font family, same text positioning template
- Slide 1 may have a slightly different layout (it's the hook) but slides 2–N must match
- Brand mark (small TCA monogram or wordmark) on every slide in the same position
- No stock-photo-with-text-slapped-on. Designed-feeling, not generated-feeling.

**Generate the carousel caption** (separate from the slides):
- 100–200 words
- Tells the reader what the carousel covers and why it's worth swiping through
- Reinforces the lead insight
- 5–10 hashtags
- Save-worthy framing ("Save this for the next time you…") — saves are a top algorithm signal

**Post the carousel:** Upload all slides via Meta Graph API as a carousel post with the caption. Schedule for ~2:00pm Eastern.

### 7. Generate and schedule Pinterest pins

**Today: 1 pin for today's post. Queue 4 more across days 2–5.**

**Critical context: the pin queue is additive, not replacing.** Every day this routine adds 5 future pins to the queue (1 fires today, 4 are scheduled for the next 4 days). At steady state — starting roughly day 5 — the queue is firing **5 pins per day across 5 different blog post URLs**: today's post (pin 1 of 5), yesterday's post (pin 2 of 5), the day before's (pin 3 of 5), and so on.

This is the right Pinterest pattern. Established accounts (6+ months): 5-10 fresh pins daily. The top 1% of pins drive 50%+ of all traffic, so volume of distinct designs is how you increase your odds of going viral. 5 pins/day rotating across recent URLs lands in the recommended range and maximizes the chance one design hits.

**Why not all 5 to today's URL today:** pinning multiple pins to the same URL on the same day triggers Pinterest's spam filters. The 24+ hour stagger across 5 days for each URL is what keeps the account clean.

**Per-pin generation (for pin #1 fired today):**

**First: check `inbox/pinterest-pins/` for pre-made pins.** Read all sidecar JSONs. Find matches by either `use_with_post_slug` matching today's post, or `topic`/`category` matching. If multiple inbox pins match, use the first one for today's pin #1 and queue the rest for days 2-5 (replacing slots that would otherwise be generated).

If no inbox match for pin #1, generate fresh using the first angle from the hooks file:
1. Compose the pin headline (8–12 words, scannable, keyword-rich, written for Pinterest search)
2. Build the GPT Image 2 prompt using the Pinterest template in `assets/image-prompts/gpt-image-2-style-guide.md`
3. Generate at 1000×1500 (2:3) with the headline as text overlay
4. Compose the pin description: 200–500 chars, keyword-rich, written for Pinterest search not human reading
5. Schedule for 9:00am Eastern today
6. UTM tracking: `?utm_source=pinterest&utm_medium=social&utm_campaign=<post-slug>`

For any inbox pin used: move file + sidecar to `inbox/used/<YYYY-MM>/` after scheduling, update sidecar with `used_in: <pin URL or scheduler ref>`.

**Queue pins #2–5:**

Write to `logs/pin-queue-<post-slug>.md` in YAML format (the `pinterest-queue-flush` routine reads this file daily and processes entries scheduled for today):

```yaml
source_post:
  slug: <post-slug>
  url: <full URL>
  category: <category>
  published_date: <today YYYY-MM-DD>

pins:
  - pin_number: 2
    angle: <angle 2 from hooks file>
    headline: <composed headline>
    gpt_image_2_prompt: <full prompt ready to fire>
    description: <composed description>
    scheduled_date: <today + 1 day>
    scheduled_time: "09:00 ET"
    status: queued
    fired_at: null
    pin_url: null

  - pin_number: 3
    # ... same structure, scheduled_date = today + 2

  - pin_number: 4
    # ... scheduled_date = today + 3

  - pin_number: 5
    # ... scheduled_date = today + 4
```

The companion routine `pinterest-queue-flush.md` (fires daily at 8:45am) reads all `pin-queue-*.md` files, finds entries scheduled for today, generates the images using the queued prompts, posts via Pinterest API, and updates each entry's `status`, `fired_at`, and `pin_url`. **You do not need to handle pins #2-5 in this routine.** Just queue them properly.

### 8. Schedule everything

| Platform | Method | Scheduled time |
|---|---|---|
| Facebook | Meta Graph API | 9:00am ET |
| LinkedIn | LinkedIn API | 10:00am ET |
| Instagram feed | Meta Graph API | 11:00am ET |
| Instagram carousel | Meta Graph API | 2:00pm ET |
| Pinterest pin #1 | Pinterest API | 9:00am ET today |
| Pinterest pins #2-5 | Queued | 9am ET on next 4 days |

If a platform API is unavailable, push to fallback scheduler (Metricool / Buffer / GHL social planner — set in environment) and log which required fallback.

### 9. Log to Notion

Create one page in the Operations Hub:
- **Task:** `Social distribution complete: <post title>`
- **Project:** `TCA`
- **Status:** `Done`
- **Type:** `Marketing`
- **Owner:** `Claude`
- **Priority:** `Medium`
- **Notes:**
  - Source post URL
  - Per platform: scheduled time, post URL, link/standalone choice and why
  - Today's per-platform mix used
  - Carousel slide count and asset URLs
  - Pinterest pin #1 details, queue file location for pins #2-5
  - Any platforms that required fallback or failed
- **Source Chat:** `daily-social-distribute routine — <date>`

### 10. Send the daily summary

To Slack `#tca-content-ops` (or email):

```
✅ Social routine complete — <date>

Source: <blog post title>
URL: <blog post URL>

Distributed today:
  ✅ Facebook (<link|standalone>) — scheduled 9:00am
  ✅ LinkedIn (<link|standalone>) — scheduled 10:00am
  ✅ Instagram feed — scheduled 11:00am
  ✅ Instagram carousel (<N> slides) — scheduled 2:00pm
  ✅ Pinterest — pin #1 of 5 for today's URL fired 9:00am
  📋 Pinterest queue: 4 new pins added for this URL (days 2-5)

Pinterest steady-state today: <N> pins firing across <N> URLs (today's + recent days')

Issues: <none | list>
Notion log: <URL>
```

---

## Failure modes

| What fails | What to do |
|---|---|
| Hooks file missing | Blog routine likely failed. Log Medium-priority Notion task. Exit. Don't invent. |
| Skill loading fails | Log High-priority. Use built-in defaults from this file. |
| Image API fails for one slide | Skip that slide, post carousel with remaining slides if 6+ remain. If <6, skip carousel for the day, log High priority. |
| Image API fails for entire batch | Post text-only versions where supported (FB, LI). Skip IG (image-required). Log High. |
| One platform API fails | Distribute to others. Log High-priority for failed platform. |
| All platform APIs fail | Push everything to fallback scheduler. Log Critical. |
| Carousel upload fails (Meta API) | Save slide images to Notion task, log High — Thomas can manually post. |
| Pinterest pin queue write fails | Pin #1 still scheduled. Log High — pins #2-5 need manual scheduling. |
| Notion logging fails | Send summary. Email Thomas about Notion separately. |

---

## What this routine never does

- Distribute identical copy across platforms
- Apply the same link/no-link rule to every platform
- Invent content when no source post exists
- Make recommendations or claims that weren't in the source post
- Use advisory CTAs
- Publish all 5 Pinterest pins to the same URL on the same day
- Use 1080×1080 square for IG carousels (always 1080×1350 portrait)
- Use anything other than 1000×1500 (2:3) for Pinterest
- Use stock photos for IG carousels (designed slides only)
- Crowd carousel slides with too much text (one idea per slide)
- Use more than 3 hashtags on LinkedIn
