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

### 2.5 Idempotency check (mandatory — runs before any other work)

This routine must be safe to invoke more than once per day. The cron fires once at 8:30 AM ET, but other paths can re-trigger it: a manual re-run during a Cowork or chat session, an Anthropic environment retry after a perceived failure, or a one-off Cowork task that uses the same prompt instead of read-only verification calls. Every re-trigger creates a parallel set of FB/IG/LI posts and a parallel Pinterest queue against the same source — confirmed observed on 2026-05-02 (one autonomous run at 8:53 AM ET, second run at ~11:00 AM ET when a Cowork verification task fired late after the user's Mac came online).

The check is the FIRST API call this routine makes — before locating yesterday's artifacts, before image gen, before any copy work.

**Compute yesterday's date in ET** (same convention as Step 3) to know what hooks file would be the source for today's distribution. Use `logs/{yesterday}-social-hooks.md` and read just the `post_url` field from the YAML front matter (do not load the full file yet).

**Query GHL for posts created in the last 24 hours by this routine:**

```
POST /social-media-posting/{locationId}/posts/list
Body:
{
  "type": "scheduled",
  "limit": "100",
  "skip": "0",
  "fromDate": "{now_utc - 24h}",
  "toDate": "{now_utc + 14d}"
}
```

Also pull the `published` feed for the same window so we don't miss posts that have already fired today:

```
POST /social-media-posting/{locationId}/posts/list
Body: same as above but "type": "published" and "fromDate": "{today 00:00 UTC}"
```

**Decision:**

For each post in the combined results, check whether ANY of the following match yesterday's source:

1. `pinterestPostDetails.link` contains yesterday's blog slug (Pinterest pins always link back to the blog URL with UTM params)
2. `summary` contains the yesterday's blog title (case-insensitive) OR
3. `summary` contains the yesterday's blog title's first 6 words (catches truncated/reworded copy that came from the same hooks file)

If ANY matching post exists AND its `createdBy` equals `GHL_USER_ID` from the vault → **exit clean.** Log a single Medium task to Operations Hub:

- **Title:** `daily-social-distribute {YYYY-MM-DD} — no-op (already distributed)`
- **Project:** TCA
- **Status:** Done
- **Priority:** Medium
- **Type:** Marketing
- **Owner:** Claude
- **Notes:**
  - Existing distribution detected for source: `{yesterday's blog title}`
  - Earliest matching post created at `{createdAt}` (post ID `{_id}`)
  - This run was a re-trigger (manual, Cowork late-fire, or environment retry) — exited without scheduling new posts
  - Full match list: `{count} matching posts across {N} platforms`

If NO matching posts exist → proceed normally to Step 3.

**Why the 24-hour window:** Today's autonomous cron run will create posts within seconds of starting. A second run hours later (e.g., a delayed Cowork task) will see those posts in the scheduled feed and exit. A run on a fresh day will find no matches and proceed. The window must be at least 24h to cover delayed Cowork-style fires.

**Why match by source URL/title, not by today's date:** A re-trigger could happen seconds after the original run with both posts having the same `createdAt`. Matching by source content is more reliable than matching by timestamp.

**Why Pinterest is the most reliable signal:** Pinterest pins always include `pinterestPostDetails.link` with the destination URL — easy unambiguous match. FB/IG/LI summaries are platform-adapted copy and may not contain the exact blog title verbatim. Check Pinterest first; fall back to summary text matching if no Pinterest pins are found.

**Failure handling on the idempotency query itself:** If the GHL list call returns 5xx or times out, log High to Operations Hub and **exit without scheduling.** Do not assume "no matches found" if we couldn't actually check. Better to skip a day than to double-post.

### 3. Locate yesterday's artifacts

Compute yesterday's date in ET (not UTC — the blog's filename convention is local-day-based).

Required files:
- `logs/{YYYY-MM-DD}-social-hooks.md` — must exist and be non-empty
- The corresponding WP post URL, title, excerpt, and featured image URL — fetched via WP REST API filtering by date

**Fail closed:**
- If the social hooks file is missing → exit clean, log Low/Admin task to Operations Hub noting that yesterday's blog publish probably failed (which has its own task already).
- If the WP post can't be located → exit clean, same logging pattern.

Do **not** distribute without both artifacts.

## Format rotation by weekday

The routine rotates LinkedIn and Instagram formats across the work week to (1) avoid algorithmic penalties from same-format-every-day patterns, (2) capture the engagement lift from carousel formats on Tue/Thu, and (3) match what high-performing accounts actually do — formats vary, theme stays consistent.

Determine the weekday in ET (not UTC — the routine fires at 8:30 AM ET, after the date has rolled over locally).

### LinkedIn rotation

| Weekday | Format | Image required |
|---|---|---|
| Mon | Text + image (1.91:1 horizontal graphic) | Yes |
| Tue | PDF carousel (8 slides — see step 6b below) | Yes — generates PDF |
| Wed | Text-only (no image) | No |
| Thu | PDF carousel (8 slides) | Yes — generates PDF |
| Fri | Text + image (1.91:1) — use chart/data graphic if blog has stats, otherwise text-only | Optional |

If the routine fires on a non-weekday (Sat/Sun), default to text-only LinkedIn — weekend distribution is lower-priority and the carousel pipeline cost isn't worth it.

### Instagram rotation

| Weekday | Format |
|---|---|
| Mon | Single image (square 1080×1080) |
| Tue | Carousel (8 slides, square 1080×1080 each) |
| Wed | Single image |
| Thu | Carousel |
| Fri | Single image |

Sat/Sun default to single image.

### Carousel days are coordinated

Tue and Thu are "carousel days" on both LinkedIn and Instagram. The carousel arc from yesterday's social hooks file (`logs/{YYYY-MM-DD}-social-hooks.md` — already contains an 8-slide arc) is the source for both. Generate the slide IMAGES once (8× 1080×1080 PNGs) and use them two ways:
- LinkedIn: stitch into a single PDF document upload
- Instagram: post as multi-image carousel (8 separate media entries in the GHL post body)

This produces visual consistency across platforms and saves on image gen API calls.

### Visual consistency on carousel slides

All carousel slides for a given day share template constraints enforced via the GPT Image 2 system prompt:
- TCA brand colors (navy `#1a2947`, warm cream `#f7f1e3`, aged brass `#9c7a3c`)
- Same header treatment across all 8 slides
- Same footer credit (small "thomasclarkadvisor.com" mark)
- Slide number indicator (1/8, 2/8, etc.) bottom-right
- One central concept per slide, scannable in 3 seconds
- Slide 1 = hook, slides 2–7 = supporting points, slide 8 = recap + CTA

Reference the `gpt-image-2-director` skill for prompt structure but pass an explicit `system_prompt` enforcing the template constraints above.

---

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

For each platform that needs an image (FB, IG, Pinterest, LinkedIn on image days):

1. If reusing WP featured image: fetch the image URL directly. No upload needed — GHL accepts external URLs.
2. **If generating fresh, use the primary→fallback chain (matches `routines/blog-publish.md`):**
   - **Primary: OpenAI Images API** (`POST https://api.openai.com/v1/images/generations`) with model `gpt-image-2`. Auth via `OPENAI_API_KEY` from the vault. Org verification was completed 2026-05-01, so this path is now live and returns correct aspect ratios when `size` is passed explicitly.
   - **Fallback: OpenRouter** (`POST https://openrouter.ai/api/v1/chat/completions`) with model `openai/gpt-5.4-image-2`, `modalities: ["image", "text"]`. Auth via `OPENROUTER_API_KEY`. Trigger this path on any OpenAI failure (rate limit, 5xx, network error, 403). The response shape differs — handle both.
   - **Final fallback:** if both providers fail, skip the image and post text-only on platforms that allow it (FB, LinkedIn). For platforms that require media (IG, Pinterest), skip that platform's post and log a per-platform warning. Do not block the whole distribution.

   Save generated images to a public-accessible URL (WP media library or a CDN). Capture the MIME type from the generation step.
3. **Verify MIME type**: GHL requires `media[].type` to be a MIME type (`image/png`, `image/jpeg`), NOT a category like `image`. Default to `image/png` for GPT Image 2 output, `image/jpeg` for WP photo images.
4. **Verify size limits**: FB 10MB, IG 8MB, LinkedIn 8MB, Pinterest 10MB.
5. **Pass an explicit `size` parameter on every image-generation API call.** Per-platform sizes:
   - **Pinterest pin (vertical 2:3):** `size: "1024x1536"`
   - **Instagram square / carousel slide (1:1):** `size: "1024x1024"`
   - **LinkedIn / Facebook horizontal (1.91:1):** `size: "1536x1024"`

   > **Critical:** Pass `size` as an explicit API parameter on every image-gen call (both OpenAI's `gpt-image-2` `/v1/images/generations` body and OpenRouter's chat-completions body). Do NOT rely on prompt language to specify aspect ratio — the model ignores it. Confirmed during 2026-05-01 run: prompts asking for vertical 2:3 returned 1024×1024 squares because the size param was unset. If a provider rejects the size value (some only accept a fixed enum), use the closest supported value and document it in the run log.

If image prep fails for one platform, post text-only on platforms that allow it (FB, LinkedIn) and skip IG/Pinterest with logged warnings. Do not block the whole distribution on one platform's image issue.

### 6b. LinkedIn PDF carousel pipeline (Tue/Thu only)

On Tue/Thu, after generating the 8 carousel slide images:

1. **Stitch slides into a single PDF** using `img2pdf` (Python): one PDF, 8 pages, slides in order, no compression that loses readability.
2. **Upload PDF to GHL media** via `POST /medias/upload-file` with `type: "application/pdf"`. Capture returned media ID and public URL.
3. **Create LinkedIn post** with the PDF as the media reference. Best-guess body shape (verify against findings doc):

```json
{
  "accountIds": ["{GHL_LI_ACCOUNT_ID}"],
  "userId": "{GHL_USER_ID}",
  "summary": "{LinkedIn carousel caption — 1,300-1,600 chars per the spec}",
  "scheduleDate": "{utc_iso_8601_z}",
  "status": "scheduled",
  "type": "post",
  "media": [{"url": "{pdf_url}", "type": "application/pdf"}],
  "followUpComment": "Read the full breakdown: {blog_url}"
}
```

#### Fallback chain (LinkedIn carousel days only)

If the PDF carousel path fails at any step, fall through:

1. **PDF upload to GHL fails** → fall to step 2
2. **PDF post create rejects (e.g., 422 on `application/pdf` MIME)** → fall to step 2
3. **Step 2: Multi-image post** — try posting the 8 slide images as a multi-image LinkedIn post (`media: [...]` with all 8 image entries). LinkedIn still supports multi-image posts even though native carousels were deprecated.
4. **Multi-image fails** → fall to step 3
5. **Step 3: Single image post** — use slide 1 as a teaser image with a "8-slide breakdown coming — what would you want to see covered?" caption.
6. **Single image fails** → fall to step 4
7. **Step 4: Text-only post** — post just the LinkedIn copy from yesterday's hooks file with no media.

Log every fallback step to Operations Hub with severity Medium so we can track which path actually fires over time. After 2–3 weeks of data we'll know which path GHL+LinkedIn supports reliably and can simplify the chain.

#### Pre-flight PDF test (one-time, before first Tuesday run)

The first Tuesday after schedule activation, the routine should run an **internal pre-flight check** before the main pipeline:

1. Generate a 3-slide dummy PDF (slides labeled "PRE-FLIGHT TEST 1/3" etc.)
2. Upload to GHL media
3. Attempt a LinkedIn POST with `status: "scheduled"`, scheduled 1 hour out
4. If POST returns success → immediately DELETE the test post, log success to Ops Hub, proceed with real pipeline
5. If POST returns 422 or rejects PDF MIME → log Critical to Ops Hub with the exact error, fall straight to step 2 (multi-image) for the day's actual carousel, and update this spec section based on the error before next Tuesday

Track the pre-flight outcome in a flag file in the repo: `state/linkedin-pdf-supported.json` with `{"supported": true|false, "tested_at": "..."}`. Subsequent Tuesdays read this flag and skip the pre-flight if already verified. If the flag says unsupported, skip PDF entirely and go straight to multi-image fallback.

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

**Per-platform retry behavior on 4xx errors (mandatory — prevents duplicate-post storms):**

If a single platform's POST returns a 4xx error (422, 400, etc.):

1. **Retry ONLY that platform's call** with the corrective body change. Do NOT re-execute Step 7 from the top. Do NOT re-run image generation. Do NOT re-call any platform that already returned 2xx.
2. Apply the corrective change in-place (e.g., add a missing `media: []`, fix a MIME type, adjust a `type` enum) and re-POST that single platform's body.
3. If the same platform fails again on the second attempt, log High to Operations Hub for that platform only, mark it failed in the per-platform status, and continue with the remaining platforms. Do not retry a third time.
4. **Never re-enter Step 7 in a loop.** A single platform's retry is bounded to one extra attempt. Re-running the whole scheduling block on a 4xx is the documented cause of the 2026-05-02 duplicate FB/IG/LI posts: the LinkedIn 422 (missing `media: []`, since fixed in this spec) cascaded into a full re-execution that re-created posts on the platforms that had already succeeded.

The Step 2.5 idempotency check defends against re-trigger paths originating outside this routine. This rule defends against re-trigger paths originating *inside* this routine.

**Schedule times (all converted to UTC ISO 8601 with Z suffix before sending):**

| Platform | Local time (ET) | Notes |
|---|---|---|
| Facebook | 9:00 AM | After commute, before mid-morning lull |
| LinkedIn | 10:00 AM | B2B finance engagement window AND structurally reachable from the 8:30 AM ET routine fire (the prior 7:30 AM target was always in the past, forcing every run into a same-day reschedule). |
| Instagram | 11:00 AM | Late-morning scroll window |
| Pinterest pin #1 | 8:00 PM | Pinterest evening pattern |
| Pinterest pin #2 | tomorrow 8:00 PM | |
| Pinterest pin #3 | +2 days 8:00 PM | |
| Pinterest pin #4 | +3 days 8:00 PM | |
| Pinterest pin #5 | +4 days 8:00 PM | |

**Per-platform body shape (confirmed via API surface test 2026-05-01 — see `docs/ghl-api-findings.md` for full schema):**

> The shapes below cover the non-carousel days (Mon/Wed/Fri/Sat/Sun). On **carousel days (Tue/Thu)**, LinkedIn uses the PDF body shape from step 6b and Instagram uses a multi-image media array (8 entries instead of 1). Apply the weekday rotation rules from "Format rotation by weekday" before assembling these bodies.

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
  "media": [],
  "followUpComment": "Read the full breakdown: {blog_url}"
}
```

LinkedIn does not require *content* in media, but the `media` field itself is mandatory on every create. Omitting it returns 422 `"media must be an array with media objects or an empty array"` (confirmed during the 2026-05-02 autonomous run). Send `media: []` for text-only posts. Text-only is the highest-performing format for finance content.

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
| Idempotency check finds existing distribution | Exit clean, log Medium "no-op (already distributed)". Do not schedule. |
| Idempotency query itself fails (5xx, timeout) | Exit, log High. Do NOT assume zero matches. Better to skip a day than double-post. |
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
| LinkedIn PDF upload fails (Tue/Thu) | Fall to multi-image, then single image, then text-only. Log Medium per fallback step. |
| Carousel slide generation partial fail | If 6+ slides generated, proceed with what we have. If <6, fall to single image for IG and text-only for LinkedIn. |
| Pre-flight PDF test fails | Log Critical, set state flag to unsupported, route Tuesday/Thursday LinkedIn to multi-image permanently until manual review. |

---

## DST switchover

This routine's cron schedule is in UTC. When DST changes:
- **Nov 1 2026** (EDT → EST): change cron from `30 12 * * *` to `30 13 * * *` to keep 8:30 AM ET execution
- **Mar 8 2027** (EST → EDT): change back to `30 12 * * *`

The schedule times *within* the routine (LinkedIn 10:00 AM ET etc.) are computed in code from "now in ET" + offset, so they auto-adjust with DST. Only the routine's own trigger cron needs manual adjustment.

---

## Reference

- **API findings doc:** `docs/ghl-api-findings.md` — full schema reference, all confirmed field names, all gotchas
- **Skill:** `tca-social-media-standards` — voice, copy structure, hashtag rules, per-platform standards
- **Skill:** `tca-compliance-check` — pre-publish compliance gate
- **Source data:** `logs/{YYYY-MM-DD}-social-hooks.md` — yesterday's blog distillation
- **Sister routine:** `routines/blog-publish.md` — produces the social hooks file this routine consumes
