# GHL Social Planner API — Surface Test Findings

Test date: 2026-05-01
Tester: Claude Code (Opus 4.7) on behalf of Thomas Clark
Location ID: `v8Ywxi0TTOF3T77DAD3c` (TCA sub-account)
Auth: Private Integration Token (`pit-…`) — value stored in Notion *Routine Vault — Credentials* under `GHL_API_KEY`

---

## TL;DR for the routine designer

Three findings will shape the routine more than anything else:

1. **`status: "scheduled"` is required.** Without it, the API publishes the post **immediately** on the live channel and ignores `scheduleDate`. This is a footgun that already cost us one accidental Facebook post during this surface test (see *Failure modes* below).
2. **`scheduleDate` is UTC ISO 8601 with a `Z` suffix.** Round-trip confirmed: send UTC, get UTC back. Do all timezone math in the routine; don't try to send local-with-offset.
3. **`media[].type` must be a MIME type, not a category.** `"image"` / `"photo"` / `"image/png"` etc. all surface as `Invalid media format type` until you send the proper MIME (`"image/png"`, presumably `"image/jpeg"`, `"video/mp4"` for video). External URLs from `thomasclarkadvisor.com` work fine once the MIME type is right — no GHL media upload required.

---

## Endpoints confirmed

| Method | Path | Purpose |
|---|---|---|
| GET    | `/social-media-posting/{locationId}/accounts`            | List connected social accounts (FB, IG, LI, Pinterest, YouTube). Pinterest boards NOT included. |
| POST   | `/social-media-posting/{locationId}/posts/list`          | Filter posts by status (`scheduled`, `published`, `draft`, `failed`, `in_progress`, `all`). `limit` / `skip` must be **strings**, not numbers. |
| POST   | `/social-media-posting/{locationId}/posts`               | Create a post (immediate publish OR scheduled — see status flag below). |
| GET    | `/social-media-posting/{locationId}/posts/{id}`          | Fetch a single post by GHL `_id`. |
| DELETE | `/social-media-posting/{locationId}/posts/{id}`          | Delete a post. **See caveat in *Failure modes*** — for already-published posts this only removes the GHL record, not the upstream post on Facebook/etc. |
| GET    | `/social-media-posting/{locationId}/categories`          | Returns `{ categories: [], count: 0 }` — empty for this location, but endpoint exists. |
| GET    | `/social-media-posting/{locationId}/tags`                | Tags endpoint exists (returned 200; content not inspected). |

---

## Authentication

- Bearer token via Private Integration Token: `Authorization: Bearer pit-…`
- Required header: `Version: 2021-07-28`
- Tested versions `2021-04-15`, `2021-07-28`, `2021-09-01` — all behave identically for these endpoints
- Scopes implicitly authorized by the PIT (no scope errors observed during the test)

---

## Connected accounts (account IDs)

All five connected accounts and their full GHL account IDs:

| Platform | Account ID | Notes |
|---|---|---|
| Facebook  | `69f4cad66948039bbb6a0915_v8Ywxi0TTOF3T77DAD3c_1096427023547077_page` | Page "Thomas Clark Advisor" |
| Instagram | `69f4cb0e53baf7a8301724a5_v8Ywxi0TTOF3T77DAD3c_17841432305893309`     | Profile `thomasclarkadvisor` |
| LinkedIn  | `69f4cb2c48c64377c008d188_v8Ywxi0TTOF3T77DAD3c_AHkqb1XPUx_profile`    | Personal profile "Thomas D. Clark" |
| Pinterest | `69f4cb5d48c64377c0090914_v8Ywxi0TTOF3T77DAD3c_1152358760812320385_profile` | Profile "Thomas Clark Advisor" |
| YouTube   | `69f4cb4b25a7c38b2e5920be_v8Ywxi0TTOF3T77DAD3c_UCboEOzPWPeFmrB5_u6n9iJA_profile` | (not in scope for the daily-distribute routine) |

The account ID format is roughly `{oauthId}_{locationId}_{originId}_{type}`. Treat as opaque — do not parse.

The accounts response also includes a top-level `groups` array (empty for this location).

---

## Pinterest boards

**Boards are NOT exposed via the accounts endpoint.** The `accounts` response for the Pinterest entry contains no board references.

**No board-list endpoint was found.** I probed 19 candidate paths spanning multiple URL shapes, version headers, and ID variants. Every attempt returned 404 or 400. Documented attempts include:

- `/social-media-posting/{loc}/pinterest-boards/{accountId}`
- `/social-media-posting/{loc}/accounts/{accountId}/pinterest-boards`
- `/social-media-posting/{loc}/accounts/{accountId}/boards`
- `/social-media-posting/oauth/{accountId}/boards`
- `/social-media-posting/oauth/{accountId}/data?type=boards` (per Stoplight community references)
- `/social-media-posting/{loc}/pinterest/boards`
- `/social-media-posting/{loc}/oauth/{accountId}/boards`
- … and 12 more variations, all 404/400

**Workaround used in this test:** Thomas scheduled a Pinterest post in the GHL UI with one board attached. `POST /posts/list` returned `pinterestPostDetails.boardIds` containing the board ID, which we then used to successfully create a Pinterest post via the API and verified round-trip. Routine onboarding will need to repeat this for each of the five boards (or use Pinterest's own public API for board enumeration).

**Confirmed `boardIds` shape:** map keyed by **`oauthId`** (NOT the full account ID), with the **Pinterest numeric board ID** as value:

```json
"boardIds": { "69f4cb5d48c64377c0090914": "1152358692094561054" }
```

The `oauthId` is the first segment of the Pinterest account ID — `<oauthId>_<locationId>_<originId>_profile`.

**Board IDs captured in this test:**
- `1152358692094561054` — name unknown (one of *Family Financial Organization*, *Money Mindset & Markets*, *Retirement Planning Essentials*, *Social Security Strategy*, *Tax-Smart Retirement*; Thomas knows which one he picked)

The other four board IDs are still uncaptured — see *Open questions*.

---

## Timezone format ⭐ HIGHEST-CONFIDENCE FINDING

**The API stores and returns `scheduleDate` as UTC ISO 8601 with a `Z` suffix** (e.g. `2026-05-03T19:00:00.000Z`).

**Confirmed by two independent observations:**

1. The pre-existing UI-scheduled post `TIMEZONE TEST DO NOT POST` was scheduled in the GHL composer for **Sunday 2026-05-03, 3:00 PM US/Eastern (EDT, UTC−04:00)**. The API returns it as:
   ```json
   "scheduleDate": "2026-05-03T19:00:00.000Z",
   "displayDate":  "2026-05-03T19:00:00.000Z"
   ```
   3 PM EDT = 19:00 UTC. ✓

2. Round-trip on the API-created post `69f4d4ec617efd86feac6fc8`: sent `2026-05-01T16:59:00.000Z`, got back the exact same string on `GET /posts/{id}`.

**Implication for the routine:** Compute the desired local publish time, convert to UTC in code, format as `YYYY-MM-DDTHH:MM:00.000Z`, and send. Do not send local times with `-04:00` / `-05:00` offsets — not tested but unnecessary now that UTC is confirmed. The routine must handle DST cutovers itself when converting "9 AM Eastern" → UTC.

`displayDate` mirrors `scheduleDate` on scheduled posts; on published posts it tracks the publish moment.

---

## Create post — confirmed minimum schema

```json
{
  "accountIds": ["<one or more account IDs>"],
  "summary": "<post body / caption>",
  "scheduleDate": "2026-05-01T20:00:00.000Z",
  "type": "post",
  "status": "scheduled",
  "media": [],
  "userId": "1X3DkPuhZenwGEoRjukr"
}
```

### Field-by-field

- **`accountIds`** (array, required): one or more account IDs from the accounts endpoint. Cross-platform multi-post is implied (untested in this run).
- **`summary`** (string, required): the caption / post body. Max char varies by platform; Pinterest UI shows a 800-char limit on title/description.
- **`scheduleDate`** (string, required): UTC ISO 8601 with `Z` suffix. See *Timezone format*.
- **`type`** (string, required): the only value seen on existing posts is `"post"`. Other values not probed.
- **`status`** (string, **required for scheduling**): set to `"scheduled"`. **Without this, the post publishes immediately and `scheduleDate` is ignored.** This is the single most dangerous field in the schema.
- **`media`** (array, required): must be present. `[]` is acceptable for text-only Facebook and LinkedIn posts. Required content for Instagram and Pinterest. External image URLs (e.g. on `thomasclarkadvisor.com`) work — no GHL upload step needed. Each entry is `{ "url": "<https URL>", "type": "<MIME type>" }`. The `type` must be a MIME type (`"image/png"`, `"image/jpeg"`, etc.), NOT a category like `"image"` or `"photo"`. Wrong category values surface as `Invalid media format type`. Allowed keys on a media object: `url`, `type`, `caption` (others are rejected with `property X should not exist`).
- **`userId`** (string, required): a GHL user ID. The PIT token alone is not sufficient — the API rejects requests without `userId`. Thomas's user ID is `1X3DkPuhZenwGEoRjukr`, recovered from `createdBy` on the timezone test post. The routine should hard-code or vault this value.

### Pagination shape (for `/posts/list`)

```json
{ "type": "scheduled", "limit": "50", "skip": "0",
  "fromDate": "2026-05-01T00:00:00.000Z",
  "toDate":   "2026-05-08T23:59:59.000Z" }
```

`limit` and `skip` **must be strings** — numeric values fail with 422 `"limit must be a number string"`.

---

## Per-platform parameter objects

The post object exposes per-platform sub-objects: `facebookPostDetails`, `instagramPostDetails`, `linkedinPostDetails`, `pinterestPostDetails`, `youtubePostDetails`, `tiktokPostDetails`, `gmbPostDetails`, `communityPostDetails`. Only the ones for accounts in `accountIds` actually matter on a given create.

### Facebook
- Field: `facebookPostDetails.type` (default `"post"`).
- Text-only posts work with `media: []`.
- No additional fields needed for the daily routine.

### Instagram
- **Media is required.** Every IG create attempt without media returns `"Instagram Account, would require image or video"`.
- Field name is `instagramPostDetails.type`, **not** `postType` / `contentType` / `format` (all of those return `"property … should not exist"`).
- **Enum values for `instagramPostDetails.type`:** `"post"`, `"story"`, `"reel"` — confirmed by API error `"Post Type must be one of the following values: - post, story, reel"`.
- "Feed post" is `type: "post"`. There is no `"feed"` value.
- `story` type rejects captions: `"Instagram Story will not support caption for publishing post directly"`.
- Other observed fields on retrieved IG posts: `collaborators` (object), `publishViaPushNotification` (bool), `publisherNote` (string), `showOnFeed` (bool, default true).
- **End-to-end create succeeded** with `accountIds: [<igAcct>]`, `media: [{url, type: "image/png"}]`, `instagramPostDetails: {type: "post"}`. Round-trip preserved.

### LinkedIn
- Text-only posts work (media optional). `media: []` accepted.
- Observed object fields: `postAsPdf` (bool), `pdfTitle` (string), `shortenedLinks` (array).
- See *Follow-up comment field* below — that field is **root-level**, not nested inside `linkedinPostDetails`.

### Pinterest
- **Media is required** (UI confirms an image is mandatory for any pin).
- `pinterestPostDetails.boardIds` is an **object map** keyed by **`oauthId`** (the first segment of the account ID), value is the **Pinterest numeric board ID**:
  ```json
  "boardIds": { "69f4cb5d48c64377c0090914": "1152358692094561054" }
  ```
- Other fields: `link` (string — outbound URL on the pin), `title` (string — pin title).
- **End-to-end create succeeded** with one board ID; round-trip preserved board mapping, media, and scheduleDate exactly.

---

## Follow-up comment field

- **Field name:** `followUpComment` (root-level on the create body).
- **NOT** nested in `linkedinPostDetails`. Returns `"property followUpComment should not exist"` when nested.
- Round-trip confirmed: sent at root, returned at root.
- Other names tested and rejected: `firstComment`, `comment`, `pinnedComment`, `linkedinFirstComment` (all return `"property X should not exist"`).
- Supported on: LinkedIn confirmed in this test. Whether Facebook / Instagram accept the same root-level field was not probed — try and observe.

---

## Failure modes observed

### 1. `status: "scheduled"` is mandatory for scheduled posts

The most important footgun in this API. A `POST /posts` body with a future `scheduleDate` but no `status: "scheduled"` field returns `201 Created` and a successful response — but the post is **immediately published** to the live social channel. `scheduleDate` is silently ignored.

**Reproduced once during this test.** The API-created Facebook test post with `scheduleDate: 2026-05-01T16:38:00.000Z` and no `status` flag was published at `2026-05-01T16:28:49.552Z`, ten minutes before the requested time, on Thomas's live Facebook page. The post was deleted from the GHL planner via `DELETE /posts/{id}` — but **the upstream post on facebook.com/ThomasClarkAdvisor likely still exists** (see #2 below). Thomas should manually delete the rogue FB post during cleanup.

### 2. DELETE on a published post is GHL-only, not upstream

`DELETE /social-media-posting/{loc}/posts/{id}` returns `200 { "success": true, "message": "Deleted Post" }` for a post that already published, but:
- The post record continues to appear in `POST /posts/list` results with `status: "published"` and `deleted: false`. (Either the delete is soft and not filtered, or GHL keeps a permanent audit row.)
- The actual post on Facebook/Instagram/etc. is **not removed** by this endpoint. The DELETE only affects the GHL planner's record.

For the routine: any "publish failed, retry" or "schedule wrong, abort" recovery logic must assume DELETE is best-effort cleanup of the GHL planner only, and the upstream channel may still hold the post. If the post never published (e.g. the delete fires before the scheduled time), DELETE works as expected and removes the planner entry cleanly.

### 3. `media[].type` must be a MIME type, not a category

```
"media.0.Invalid media format type"
```

Returned for every category-style value tested: `image`, `IMAGE`, `photo`, `picture`, `video`, `post`, `feed`, `reel`, `story`, `img`. The fix is to send the **MIME type**: `"image/png"` (and presumably `"image/jpeg"`, `"video/mp4"`, etc.). Once the MIME type is correct, **external URLs from `thomasclarkadvisor.com` are accepted** end-to-end on Facebook, Instagram, and Pinterest (round-trip confirmed). No GHL-internal upload is required.

The error message also confirmed valid keys on a media object are `url`, `type`, and `caption` only. Invalid keys: `mediaUrl`, `link`, `alt`.

### 4. Instagram requires media, no exceptions

All IG create attempts with `media: []` return `"Instagram Account, would require image or video"`, regardless of `instagramPostDetails.type`. With media present and a MIME type, IG creates succeed.

### 5. `userId` is required on creates

Without `userId`, create returns 422 with `"userId must be a string"` and `"userId should not be empty"`. The PIT token does not implicitly carry a user identity for write operations.

### 6. Pagination types are surprising

`limit` and `skip` on `/posts/list` must be sent as **strings** (`"50"`, `"0"`). Sending them as numbers returns 422 `"limit must be a number string"`. (`fromDate` and `toDate` accept ISO strings as expected.)

### 7. `platform` field on the post object is misleading

Both the timezone test post (a Facebook post) and the API-created Facebook post show `"platform": "google"` at the top level. The actual platform is determined by `accountIds[]`, not by this field. Don't trust `post.platform` for routing.

### 8. Create response sometimes omits the post body

The first FB create call returned only `{ success, statusCode, message: "Created Post", traceId }` with no `results` block. After adding `status: "scheduled"`, subsequent creates returned the full `results.post` object including `_id`, `scheduleDate`, etc. Defensive routine design: always re-`list` after create if `results.post._id` is missing.

---

## Open questions remaining

These should be resolved before the daily-social-distribute routine ships, but none are blocking now that media + Pinterest schemas are confirmed:

1. **Pinterest board IDs (4 of 5 still uncaptured).** One board ID was captured via the UI-then-list workaround (`1152358692094561054`); the remaining four need the same treatment. Per the routine's likely usage, Thomas can either schedule a UI post on each remaining board sequentially (each surfaces its `boardIds` mapping in `posts/list`) or use Pinterest's own public API for board enumeration outside GHL. Add captured IDs to the vault as `GHL_PIN_BOARD_<KEY>` entries with descriptive names.

2. **Video MIME types.** Only `image/png` was tested. `image/jpeg` and `video/mp4` should work but were not probed. The daily-distribute routine will likely only deal with images, so deferrable.

3. **`type` field enum at root.** Only `"post"` was observed. The root-level `type` may mirror or be independent of `instagramPostDetails.type`. Untested.

4. **Multi-platform posts.** Can a single `accountIds` array span FB + IG + LI in one call, or must each platform be its own create? Untested. The routine should default to one create per platform until proven otherwise.

5. **Carousel / multi-image posts.** Multiple `media` entries probably work but the limit (2? 10?) and any per-platform constraints are unconfirmed.

6. **Cleanup of `TIMEZONE TEST DO NOT POST`.** Per the original brief, this stays in place — Thomas will delete `TIMEZONE TEST DO NOT POST` (`_id: 69f4d33d15dfd8b2230ebe2e`, scheduled 2026-05-03 19:00:00 UTC) manually after reviewing these findings.

7. **Manual cleanup of accidentally-published Facebook post.** Thomas should check facebook.com/ThomasClarkAdvisor and delete the post timestamped 2026-05-01 12:28 PM EDT containing the text *"API TEST — Facebook UTC scheduling check. Will be deleted in 5 min."*. The GHL planner record persists with `status: published` but `DELETE` calls return 200; the actual post on Facebook itself was never removed by the GHL DELETE endpoint.
