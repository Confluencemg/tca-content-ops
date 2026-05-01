# Routine: daily-blog-publish

**Trigger:** Cron, daily at 2:30am Eastern (`30 2 * * *`)
**Runtime budget:** ~30 minutes
**Output:** One scheduled WordPress post live at 8:00am, 2–3 images generated, social hooks extracted, Notion log row

**Why 2:30am?** Off-peak for Thomas (he's not actively using Claude), reducing concurrent usage strain. Plenty of buffer before the 8am publish window.

---

## Mission

Produce one publish-ready blog post per day for thomasclarkadvisor.com. The post is scheduled in WordPress (not published immediately) so it goes live at 8:00am regardless of what happens after the routine finishes.

The post must sound like Thomas Clark, follow the compliance posture in `CLAUDE.md`, include 2–3 brand-aligned images, and pass the compliance check before scheduling.

The post must also be **engagement-worthy on its own** — not just a vehicle for a click. Strong opener, useful payoff, distinct point of view. Posts that read like generic financial-advisor content fail this routine even if they pass compliance.

---

## Step-by-step

### 0. Load TCA skills (mandatory first step)

Before doing anything else, load:
- **`tca-content-writer`** — the primary skill. Defines voice, structure, pillar logic, hypothetical example construction, and the article workflow. This skill internally loads `tca-compliance-check` as a mandatory final pass, so you don't need to load compliance separately.
- `gpt-image-2-director` — for image generation prompts in Step 5
- `seo-audit` — only for SEO-relevant pieces, optional

The `tca-content-writer` skill is authoritative for everything blog-content-related. This routine file orchestrates the steps; the skill defines how each step is actually done. If they conflict, the skill wins for matters within its scope.

If `tca-content-writer` won't load: log to Notion as Critical Tech task, **stop**. Do not attempt to write a blog post without the writer skill — voice and compliance drift will result.

### 1. Determine today's topic

- Read `content-calendar.md`. If today has a planned topic, use it.
- If not, default to the day-of-week category from `CLAUDE.md` and pick a fresh angle on that category.
- Check the last 14 days of published posts (query WordPress: `GET /wp-json/wp/v2/posts?per_page=14&orderby=date`) to avoid repeating an angle that ran recently.

### 2. Research

- For Market Insights or Trading topics: web search current market conditions, relevant news from the last 24–48 hours, any major economic releases that day. Use specific data, not generalities.
- For Social Security topics: check current SSA rules and recent updates. Social Security guidance changes — never publish stale data without verifying.
- For Retirement Planning / Bucket Planning: pull from Thomas's existing bucket planning content on the site to maintain consistency. Don't reinvent the philosophy each time.
- For Financial Education: pick a concrete angle, not a survey.

### 3. Draft the post

**Length:** 800–1,400 words. Quality over length.

**Structure:**
- Opening hook: 1–3 short sentences. State the thing, then add nuance.
- The main argument or teaching, broken into 3–5 logical sections with H2 headings (no H1)
- A specific concrete example or hypothetical (clearly labeled as hypothetical when used)
- A close that lands the point — never trails off into generic advice
- Footer: standard disclaimer block from `assets/disclaimers/standard-blog-footer.md` and author bio from `assets/author-bio.md`

**Engagement test (ask before moving on):**
- Could a reader walk away from just the headline + opener and have learned something? If no, the opening is too soft.
- Is there a moment in the post where Thomas takes a clear stance most other advisors wouldn't? If no, the post is too safe to be memorable.
- Is the close a payoff or a wind-down? If wind-down, rewrite the close.

**Voice check:**
- Read the draft. Does it sound like Thomas (per `CLAUDE.md` voice profile)?
- Did you slip into generic financial-advisor voice? Fix it.
- Are there any anti-patterns from `voice/anti-patterns.md`? Remove them.

**Compliance check:**
- Run all 8 hard rules from `CLAUDE.md`. Every rule must pass.
- If any fails, fix it. If it can't be fixed without losing the point, scrap the post and pick a different angle.

### 4. Extract social hooks (critical handoff to social routine)

Write to `logs/<date>-social-hooks.md`:

```yaml
date: <YYYY-MM-DD>
post_title: <title>
post_url: <URL>
category: <category>
audience_segments: [<segments>]

# Lead insight — the single strongest standalone takeaway
# Used as the opener for FB and LinkedIn posts
lead_insight: |
  <1-3 sentences delivering value even if reader never clicks>

# Secondary insights — for IG carousel slides and standalone posts
secondary_insights:
  - <insight 1, ~1 sentence>
  - <insight 2, ~1 sentence>
  - <insight 3, ~1 sentence>

# Contrarian angle (if any) — Thomas's stance against conventional wisdom
contrarian_angle: |
  <Thomas's specific take that pushes back on what most advisors say, or null if none>

# Carousel arc — 8-10 slide beats for IG carousel (research-backed sweet spot)
carousel_arc:
  hook_slide: <slide 1: stop-the-scroll opener, ~8-12 words for visual>
  slide_2: <build curiosity, ~15-25 words>
  slide_3: <set up the problem or context, ~20-30 words>
  slide_4: <key insight #1, ~20-30 words>
  slide_5: <key insight #2 or supporting point, ~20-30 words>
  slide_6: <example or specific application, ~25-35 words>
  slide_7: <so what / why it matters, ~20-30 words>
  slide_8: <action or reframe, ~15-25 words>
  slide_9_optional: <additional supporting beat if topic warrants 9 slides>
  cta_slide: <"save this for later" or "link in bio for full breakdown" + brand mark>

# Pinterest pin angles — for the social routine to generate distinct pins
pinterest_angles:
  - <pin angle 1: distinct headline framing>
  - <pin angle 2: different audience hook>
  - <pin angle 3: different value proposition>
  - <pin angle 4: question-led hook>
  - <pin angle 5: list-led hook>
```

This file is the single source of truth the social routine consumes. Don't make the social routine re-read the full blog post — it's slower, costs more tokens, and risks drift between blog and social.

### 4.5. Commit the social hooks file to main

The next-day `daily-social-distribute` routine clones a fresh copy of the repo at run time and reads `logs/<date>-social-hooks.md` from main. If the file isn't on main, social-distribute won't see it.

```bash
git add logs/<date>-social-hooks.md
git commit -m "logs: <date> social-distribute handoff (post <wp_post_id>)"
git push origin main
```

**Push directly to main.** Do not create a feature branch, do not open a PR. Per `CLAUDE.md` → Routine architecture, every routine that writes back to the repo follows this pattern: `git add` → `git commit` → `git push origin main`. The "Allow unrestricted git push" permission on this routine grants direct-push access.

If git push fails: retry once. If still failing, log Critical Notion task with the file path and error — the social handoff is downstream-blocking. Do not skip this step.

### 5. Generate images

**First: check `inbox/blog-images/` for pre-made assets matching today's post.**

For each image slot (featured, in-body 1, in-body 2):
1. Read all sidecar JSON files in `inbox/blog-images/`
2. Find matches by either: (a) `use_with_post_slug` matches today's post slug, or (b) `topic`/`category` matches today's post topic
3. If match found: use the file. Move it to `inbox/used/<YYYY-MM>/` after upload. Update sidecar with `used_in: <post URL>`.
4. If no match: generate via OpenAI Images API (`gpt-image-2`)

**For generation (when inbox doesn't have a match):**

- **Featured image** (1200×630): editorial photography aesthetic, no text overlay, brand palette
- **In-body image 1** (1200×800): supporting concept image
- **Optional in-body image 2** (1200×800): only if 1,200+ words

Use `assets/image-prompts/gpt-image-2-style-guide.md` for prompt patterns. Upload to WordPress media library via REST API.

### 6. Assemble the WordPress post payload

```
POST /wp-json/wp/v2/posts
{
  "title": "<post title>",
  "content": "<full HTML with image tags inserted at appropriate breaks>",
  "status": "future",
  "date": "<today at 8:00am Eastern in ISO 8601>",
  "categories": [<category ID>],
  "tags": [<tag IDs>],
  "featured_media": <featured image media ID>,
  "excerpt": "<2-sentence excerpt>",
  "meta": {
    "_yoast_wpseo_metadesc": "<155-char SEO meta description>",
    "_yoast_wpseo_focuskw": "<focus keyword>"
  }
}
```

**Status MUST be `future` with a `date` in the future.** This is the autopilot guarantee.

### 7. Log to Notion

Create a page in the Operations Hub:
- **Task:** `Blog post published: <title>`
- **Project:** `TCA`
- **Status:** `Done`
- **Type:** `Content`
- **Owner:** `Claude`
- **Priority:** `Medium`
- **Notes:** Post URL, scheduled time, category, image URLs, word count, lead insight, carousel arc summary
- **Source Chat:** `daily-blog-publish routine — <date>`

**Note:** This routine does NOT trigger an email broadcast. The TCA newsletter sends once per week (Sundays via `weekly-newsletter-digest` routine), not per-post. Daily blog emails would create unsubscribe pressure on a multi-generational audience.

### 8. Send the daily summary

To Slack `#tca-content-ops` (or email if not connected):

```
✅ Blog routine complete — <date>

Title: <post title>
Category: <category>
Scheduled: 8:00am ET
URL: <post URL>
Words: <count>
Images: <count> generated, <count> uploaded
Notion log: <Notion page URL>

Social hooks file: logs/<date>-social-hooks.md
  Lead insight: <1-line>
  Carousel: <5-7 beats>

Issues: <none | list>
```

---

## Failure modes and how to handle each

| What fails | What to do |
|---|---|
| Web search returns nothing useful | Pivot to evergreen angle on same category. Don't skip the day. |
| OpenAI image API fails (incl. 403 org-not-verified, rate limit, 5xx) | Fall back to OpenRouter `openai/gpt-5.4-image-2`. If OpenRouter also fails, publish without images and log a Critical Notion task. Never block the publish. |
| WordPress media upload fails | Retry once. If fails, publish without images and log. |
| WordPress post creation fails | Stop. Log Critical Notion task with full draft. |
| Compliance check returns FAIL_BLOCK | Stop. Log Critical Notion task with full draft + findings. |
| Notion logging fails | Send summary anyway. Email Thomas separately about Notion. |
| Voice/compliance fails twice | Stop. Critical Notion task with draft + reason. Don't publish. |
| Engagement test fails twice | Don't publish weak content. Log Blocked, alert Thomas, skip day. |
| Social hooks file write fails | Critical — social routine depends on this. Retry, then log Critical task. |

---

## What this routine never does

- Publish immediately (always scheduled for 8am)
- Skip the compliance check
- Skip the engagement test
- Make specific investment recommendations
- Use advisory CTAs
- Email Thomas asking permission to publish — passes all checks or logs Blocked
- Run if previous day's run is still in progress
