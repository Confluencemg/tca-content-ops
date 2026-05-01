# daily-social-distribute

**Schedule:** Daily 8:30 AM ET (cron `30 12 * * *` UTC during EDT, `30 13 * * *` UTC during EST)
**Environment:** TCA Daily Task (`env_01UiKj8r6fjAb2fUusvoLKk1`) — full network access required
**Connectors:** Notion MCP only. GHL is hit via raw HTTP (services.leadconnectorhq.com).
**Routine ID:** TBD (set on creation)

## Purpose

Adapt yesterday's published blog post into platform-native social copy for Facebook, Instagram, LinkedIn, and Pinterest, then schedule the posts via the GHL Social Planner API. Pinterest pin #1 publishes today; pins #2–5 get queued across the next 4 days against the same blog URL with varied copy and creative.

This routine **only schedules**. It does not publish immediately. Every API call sets `status: "scheduled"` and a future `scheduleDate`. Publishing is GHL's job, on the schedule we set.

## Why this routine exists separately from `daily-blog-publish`

Blog publishes at 8 AM ET. Distribution runs at 8:30 AM ET so:
- The blog post is already live (in case any social copy references the URL or excerpt)
- The social hooks file from yesterday (`logs/{YYYY-MM-DD}-social-hooks.md`) is committed
- Failures in distribution don't block the next morning's blog post

If yesterday's blog publish failed, this routine exits clean — there is no content to distribute.

---

## Step-by-step

### 0. Load TCA skills (mandatory first step)

Before doing anything else, load:
- **`tca-social-media-standards`** — authoritative for per-platform content types, copy register, hashtag rules, visual standards. This skill internally loads `tca-compliance-check` for any post touching financial topics.
- `gpt-image-2-director` — for Pinterest pin image generation (only if pin images need to be generated fresh; if reusing the blog hero image, skip)

If `tca-social-media-standards` won't load: log to Notion as Critical Tech task, **stop**. Do not attempt distribution without the skill — voice and platform-fit drift will result.

### 1. Vault fetch (credentials)

Fetch from Notion vault page `Routine Vault — Credentials` (page ID `3533d750-f142-8149-8daf-fea08f6969a2`):

- `GHL_API_KEY` — Private Integration Token
- `GHL_LOCATION_ID` — `v8Ywxi0TTOF3T77DAD3c` (TCA sub-account)
- `GHL_USER_ID` — required on every create call
- `GHL_FB_ACCOUNT_ID`
- `GHL_IG_ACCOUNT_ID`
- `GHL_LI_ACCOUNT_ID`
- `GHL_PIN_ACCOUNT_ID`
- `GHL_PIN_OAUTH_ID` — first segment of the Pinterest account ID, used as map key in `boardIds`
- `GHL_PIN_BOARD_ID_SOCIAL_SECURITY` — Social Security Strategy board
- `GHL_PIN_BOARD_ID_RETIREMENT` — Retirement Planning Essentials board
- `GHL_PIN_BOARD_ID_MARKETS` — Money Mindset & Markets board
- `GHL_PIN_BOARD_ID_FAMILY_FINANCIAL` — Family Financial Organization board (default fallback)
- `GHL_PIN_BOARD_ID_TAX_SMART_RETIREMENT` — Tax-Smart Retirement board

If any required credential is missing, exit clean and log Critical to Operations Hub. Do not proceed with partial credentials.

**Never log token values.** Token only appears in the `Authorization: Bearer` header.

### 2. Repo clone

Clone or pull latest `main` from `github.com/Confluencemg/tca-content-ops` to a working directory.

### 3. Locate yesterday's artifacts

Compute yesterday's date in ET (not UTC — the blog's filename convention is local-day-based).

Required files:
- `logs/{YYYY-MM-DD}-social-hooks.md` — must exist and be non-empty
- The corresponding WP post URL, title, excerpt, and featured image URL — fetched via WP REST API filtering by date

**Fail closed:**
- If the social hooks file is missing → exit clean, log Low/Admin task to Operations Hub noting that yesterday's blog publish probably failed (which has its own task already).
- If the WP post can't be located → exit clean, same logging pattern.

Do **not** distribute without both artifacts.

### 4. Adapt to platform-native copy

For each platform, generate copy per the `tca-social-media-standards` skill. The social hooks file from yesterday already contains:
- Lead insight
- Secondary insights
- Contrarian angle
- 8-slide carousel arc
- 5 Pinterest pin angles

Use those as input. Do not invent new angles — the hooks file is the source of truth.

**Per-platform output specs:**

#### Facebook
- Format: feed post with link preview to blog URL
- Body: 1–3 sentence hook from the social hooks file's lead insight, then a CTA driving to the blog
- Character target: ≤500 (well under the 62,000 ceiling)
- Follow-up comment: a one-line engagement question tied to the post's topic

#### Instagram
- Format: single feed image (NOT carousel — carousel is a separate future routine)
- Image: ALWAYS generate a fresh 1:1 (1080×1080) image via GPT Image 2 using the blog topic and hook as the prompt input. Do not attempt to crop the WP featured image — most blog heroes are 16:9 or 3:2 and forced cropping loses the focal point. Image gen is one API call per day; that cost is acceptable.
- Caption: hook + value bullet line + CTA + hashtags
- Hashtags: 5–8 relevant tags from the standards skill's TCA hashtag list
- Character target: ≤2,200 (Instagram's hard limit)
- Follow-up comment: "Link in bio for the full breakdown: {blog URL}" (Instagram strips clickable links from captions — this directs hand-typing users)

#### LinkedIn
- Format: long-form native text post (NOT a link post — LinkedIn algorithm penalizes link posts)
- **Target length: 1,300–1,600 characters** (≈220–280 words). This is the empirically-validated engagement sweet spot for 2026 — long enough to deliver substance, short enough to fit the 30–45 second dwell time the algorithm rewards. Do not exceed 2,000 chars; completion rate falls off above that. Hard ceiling: 3,000 chars (LinkedIn limit).
- **Mandatory structure:**
  1. **Hook (first ~200 chars)** — must contain the insight or contradiction. The first 200 chars are the only thing visible before "see more" truncation. No throat-clearing, no "I want to share..." preamble.
  2. **Body (3–5 short paragraphs)** — 1–2 sentences each, with white space between every paragraph. No walls of text. Single-line paragraphs for emphasis are fine.
  3. **Insight payoff** — deliver the takeaway explicitly. Don't bury it.
  4. **Closing question (last 100–200 chars)** — specific and answerable. Drives comments, which drive algorithmic distribution more than likes or shares.
- No URL in the body
- Follow-up comment: "Read the full breakdown: {blog URL}" (this is the standard LinkedIn pattern — link goes in the first comment, not the post body)

#### Pinterest pin #1 (today)
- Image: vertical 2:3 (1000×1500) — either the blog hero re-cropped or a fresh pin generated via GPT Image 2
- Title: ≤100 chars, SEO-keyword-led, distinct from the blog title
- Description: ≤500 chars, hook + value statement + soft keyword inclusion
- Destination URL: the blog post URL
- Board: select per pillar mapping in step 4a below

### 4a. Pinterest board mapping

Pillar → board (vault keys, captured 2026-05-01):

| Pillar | Vault Key | Board Name |
|---|---|---|
| Social Security | `GHL_PIN_BOARD_ID_SOCIAL_SECURITY` | Social Security Strategy |
| Retirement Planning | `GHL_PIN_BOARD_ID_RETIREMENT` | Retirement Planning Essentials |
| Bucket Planning | `GHL_PIN_BOARD_ID_RETIREMENT` | (shares the Retirement board) |
| Market Insights | `GHL_PIN_BOARD_ID_MARKETS` | Money Mindset & Markets |
| Trading & Markets | `GHL_PIN_BOARD_ID_MARKETS` | (shares the Markets board) |
| Financial Education | `GHL_PIN_BOARD_ID_FAMILY_FINANCIAL` | Family Financial Organization |
| Tax-related content | `GHL_PIN_BOARD_ID_TAX_SMART_RETIREMENT` | Tax-Smart Retirement |

If the post's pillar doesn't map cleanly to one of these (e.g., a content piece spans multiple pillars), default to `GHL_PIN_BOARD_ID_FAMILY_FINANCIAL` as the most general fit.

For the 5-pin queue (pin #1 today + pins #2–5 across the next 4 days), pin #1 goes to the primary pillar board. Pins #2–5 should rotate across **2–3 different boards** where the topic genuinely fits, not just the same board 5 times. This prevents Pinterest from flagging duplicate-board behavior and improves discoverability across multiple board contexts. Example: a Social Security post might pin to Social Security Strategy (pin #1), Retirement Planning Essentials (pin #2), Tax-Smart Retirement (pin #3), Family Financial Organization (pin #4), Social Security Strategy again with different creative (pin #5).

### 5. Compliance pass

Run `tca-compliance-check` against all four primary posts and all five Pinterest pins (pin #1 plus #2–5).

- Tier 1 issues (e.g., performance promises, implied advisory relationship, missing required disclosure on a triggered post) → **fail closed**, do not schedule, log Critical task.
- Tier 2 issues → auto-fix and continue.
- Pass → continue.

Note: routine social posts do **not** require the standard footer disclaimer per the standards skill. Profile-level disclosure suffices. Inline disclosure is only required for testimonials, performance claims, third-party ratings, or product comparisons — and the compliance check is what flags those.

### 6. Image preparation

For each platform that needs an image (FB, IG, Pinterest):

1. If reusing WP featured image: fetch the image URL directly. No upload needed — GHL accepts external URLs.
2. If generating fresh: use GPT Image 2 via `gpt-image-2-director` skill, save to a public-accessible URL (WP media library or a CDN). Capture the MIME type from the generation step.
3. **Verify MIME type**: GHL requires `media[].type` to be a MIME type (`image/png`, `image/jpeg`), NOT a category like `image`. Default to `image/png` for GPT Image 2 output, `image/jpeg` for WP photo images.
4. **Verify size limits**: FB 10MB, IG 8MB, LinkedIn 8MB, Pinterest 10MB.

If image prep fails for one platform, post text-only on platforms that allow it (FB, LinkedIn) and skip IG/Pinterest with logged warnings. Do not block the whole distribution on one platform's image issue.

### 7. Schedule posts via GHL API

For each platform, `POST /social-media-posting/{locationId}/posts` with the platform-specific request body.

**Hard requirements on every call (safety interlocks):**

```javascript
// Pre-flight validation — refuse to call API if any of these fail
assert(body.status === "scheduled");
assert(body.scheduleDate.endsWith("Z")); // UTC ISO 8601
assert(new Date(body.scheduleDate) > new Date(Date.now() + 5 * 60 * 1000)); // ≥5 min out
assert(body.userId);
assert(Array.isArray(body.accountIds) && body.accountIds.length > 0);
```

Any assertion fails → log Critical, exit. Do not POST. The `status: "scheduled"` check is non-negotiable — without it, the post publishes immediately and creates the same kind of rogue post we hit during the API surface test.

**Schedule times (all converted to UTC ISO 8601 with Z suffix before sending):**

| Platform | Local time (ET) | Notes |
|---|---|---|
| LinkedIn | 7:30 AM | Highest engagement window for B2B finance content |
| Facebook | 9:00 AM | After commute, before mid-morning lull |
| Instagram | 11:00 AM | Late-morning scroll window |
| Pinterest pin #1 | 8:00 PM | Pinterest evening pattern |
| Pinterest pin #2 | tomorrow 8:00 PM | |
| Pinterest pin #3 | +2 days 8:00 PM | |
| Pinterest pin #4 | +3 days 8:00 PM | |
| Pinterest pin #5 | +4 days 8:00 PM | |

**Per-platform body shape (confirmed via API surface test 2026-05-01 — see `docs/ghl-api-findings.md` for full schema):**

#### Facebook
```json
{
  "accountIds": ["{GHL_FB_ACCOUNT_ID}"],
  "userId": "{GHL_USER_ID}",
  "summary": "{facebook copy}",
  "scheduleDate": "{utc_iso_8601_z}",
  "status": "scheduled",
  "type": "post",
  "media": [{"url": "{image_url}", "type": "image/jpeg"}],
  "followUpComment": "{engagement question}"
}
```

#### Instagram
```json
{
  "accountIds": ["{GHL_IG_ACCOUNT_ID}"],
  "userId": "{GHL_USER_ID}",
  "summary": "{instagram caption with hashtags}",
  "scheduleDate": "{utc_iso_8601_z}",
  "status": "scheduled",
  "type": "post",
  "media": [{"url": "{square_image_url}", "type": "image/jpeg"}],
  "followUpComment": "Link in bio for the full breakdown: {blog_url}"
}
```

Note: Instagram `type` enum is `post|story|reel` (NOT `feed`). For feed posts, use `"type": "post"`.

#### LinkedIn
```json
{
  "accountIds": ["{GHL_LI_ACCOUNT_ID}"],
  "userId": "{GHL_USER_ID}",
  "summary": "{long-form native text post, no URL}",
  "scheduleDate": "{utc_iso_8601_z}",
  "status": "scheduled",
  "type": "post",
  "followUpComment": "Read the full breakdown: {blog_url}"
}
```

LinkedIn does not require media. Text-only is the highest-performing format for finance content.

#### Pinterest (each pin)
```json
{
  "accountIds": ["{GHL_PIN_ACCOUNT_ID}"],
  "userId": "{GHL_USER_ID}",
  "summary": "{pin description, ≤500 chars}",
  "scheduleDate": "{utc_iso_8601_z}",
  "status": "scheduled",
  "type": "post",
  "media": [{"url": "{vertical_image_url}", "type": "image/png"}],
  "pinterestPostDetails": {
    "boardIds": {"{GHL_PIN_OAUTH_ID}": "{numeric_board_id}"},
    "title": "{pin title, ≤100 chars}",
    "link": "{blog_url}"
  }
}
```

Pinterest `boardIds` is a map keyed by `oauthId` (not an array of board IDs). Use the OAuth ID from the vault.

### 8. Capture post IDs and verify

For each successful POST, capture the returned post `_id` (or `id`). Immediately call `GET /social-media-posting/{locationId}/posts/{id}` for each to verify:
- `status` is `"scheduled"`
- `scheduleDate` matches what was sent (round-trip check — confirms timezone behavior)
- `accountIds` includes the right account

If verification fails on any post, log High to Operations Hub with the post ID and what mismatched. Do not retry automatically — flag for human review.

### 9. Failure handling — partial success

If one platform's create call fails (e.g., IG rejects an image, LinkedIn returns 422 on a field), do **not** roll back the successful platforms. Log per-platform status to Operations Hub:

- ✅ Facebook scheduled — post ID, scheduled time
- ✅ LinkedIn scheduled — post ID, scheduled time
- ❌ Instagram failed — error message, body sent (with credentials redacted)
- ✅ Pinterest pins 1–5 scheduled

Partial-success logging beats all-or-nothing. The failed platform gets a follow-up High task.

### 10. Notion run log

Write a single summary entry to Operations Hub:

- **Title:** `daily-social-distribute {YYYY-MM-DD} — {N}/4 platforms scheduled`
- **Project:** TCA
- **Status:** Done (if all 4 succeeded), In Progress (if partial), or Blocked (if 0 succeeded)
- **Priority:** Low (full success), Medium (partial), High (full failure)
- **Type:** Marketing
- **Owner:** Claude
- **Notes:**
  - Source post: title, URL
  - Per-platform: scheduled time, post ID
  - Pinterest queue: 5 pins scheduled across {date range}
  - Any failures with error messages
  - Link to the social hooks file in the repo

### 11. Direct push to main

The only repo write is the run log entry (none — Operations Hub absorbs the log). No commits required for this routine. If the run log ever moves into the repo, push direct to main per ROUTINES.md convention #2.

### 12. Done

Routine exits cleanly. No further action.

---

## What this routine does NOT do

- **Comment management.** Replies happen on-platform.
- **Engagement analytics.** Future `weekly-social-analytics` routine handles that.
- **YouTube uploads.** Separate routine for Descript/Higgsfield video pipeline.
- **TikTok / Threads / GBP / X.** Not in scope. Each platform that gets added in the future is a routine update + standards skill update, not a separate routine.
- **Carousel posts.** Single image only for v1. Carousel is a future enhancement once we know what arc length actually performs.
- **Recurring/evergreen posts.** UI-only feature. If recycling becomes valuable, that's a separate manual workflow.

---

## Failure mode reference

| Failure | Behavior |
|---|---|
| GHL token expired | Exit, log Critical, await manual reauth |
| Token has insufficient scope | Exit, log Critical, await scope fix |
| `status: "scheduled"` missing from body | Pre-flight assert fails — refuse to send, log Critical |
| `scheduleDate` not UTC with Z | Pre-flight assert fails — refuse to send, log Critical |
| `scheduleDate` <5 min out | Pre-flight assert fails — refuse to send, log Critical |
| Social hooks file missing | Exit clean, log Low |
| WP post not findable | Exit clean, log Low |
| One platform's image fails | Skip that platform's image, post text where allowed |
| One platform's create fails | Continue with other platforms, log per-platform |
| All 4 platforms fail | Exit, log High to Operations Hub |
| Verification round-trip mismatch | Continue (post is created), log High for review |
| Pinterest board ID missing for pillar | Fall back to `GHL_PIN_BOARD_ID_FAMILY_FINANCIAL`, log Low |

---

## DST switchover

This routine's cron schedule is in UTC. When DST changes:
- **Nov 1 2026** (EDT → EST): change cron from `30 12 * * *` to `30 13 * * *` to keep 8:30 AM ET execution
- **Mar 8 2027** (EST → EDT): change back to `30 12 * * *`

The schedule times *within* the routine (LinkedIn 7:30 AM ET etc.) are computed in code from "now in ET" + offset, so they auto-adjust with DST. Only the routine's own trigger cron needs manual adjustment.

---

## Reference

- **API findings doc:** `docs/ghl-api-findings.md` — full schema reference, all confirmed field names, all gotchas
- **Skill:** `tca-social-media-standards` — voice, copy structure, hashtag rules, per-platform standards
- **Skill:** `tca-compliance-check` — pre-publish compliance gate
- **Source data:** `logs/{YYYY-MM-DD}-social-hooks.md` — yesterday's blog distillation
- **Sister routine:** `routines/blog-publish.md` — produces the social hooks file this routine consumes
