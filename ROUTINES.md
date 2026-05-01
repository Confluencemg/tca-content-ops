# TCA Content Ops — Routines Specification

This file is the authoritative inventory of every routine that runs against this repo. When a new agent connects, this is the file it reads to understand what's running, what each routine does, and what the architectural conventions are.

**Do not infer routines from the contents of `routines/`. Use this file.** If a routine file exists in `routines/` but isn't listed here, treat it as draft/deprecated until added here. If a routine is listed here but the file doesn't exist, the file needs to be built before the routine can run.

---

## Architectural conventions

These apply to every routine in the system. New routines must follow these.

**1. Credentials live in the Notion vault.** No credentials in the repo, no credentials in routine env vars. Every routine starts by fetching credentials from the Notion page titled exactly `Routine Vault — Credentials`, parsing the fenced code block. See `CLAUDE.md` for the fetch protocol.

**2. Routines push directly to `main`.** The "Allow unrestricted git push" toggle on each routine's Permissions tab grants this access. Pattern is: `git add` → `git commit -m "<routine-name> run YYYY-MM-DD"` → `git push origin main`. No feature branches, no PRs, no merge step. The audit trail of what each routine did lives in WordPress, Notion Operations Hub, and the Git commit log on `main` — not in PR history.

**3. Notion Operations Hub is the run log.** Every routine writes a run-summary task to Operations Hub (data source ID `c96ec729-0d1f-4eb3-a88a-bf1e586b4dfe`). Failures and follow-ups also write Critical/High-priority tasks. The "Source Chat" property is set to the routine name and run timestamp.

**4. Network egress requires the TCA Daily Task environment.** All TCA routines run in the cloud environment named `TCA Daily Task` (network access: full). The Default environment's restricted allowlist does not have what these routines need.

**5. Image generation: OpenAI primary, OpenRouter fallback.** Default to OpenAI's `gpt-image-2` (requires org verification on the OpenAI account). On failure, fall back to OpenRouter using model `openai/gpt-5.4-image-2`. If both fail: publish without images, log a Critical task, never block the publish.

**6. Compliance check before publishing.** Every routine that produces user-facing content runs the `tca-compliance-check` skill from `skills/` before publishing. Tier 2 warnings get auto-fixed where possible; Tier 1 critical failures stop the routine and log a Critical task.

**7. Voice: read `voice/tca-voice-examples.md` and `voice/anti-patterns.md` before drafting.** Don't draft from generic memory of "financial advisor voice." The voice files are the source of truth.

---

## Routine inventory

### 1. `daily-blog-publish`

| | |
|---|---|
| **Status** | Live (first run 2026-05-01 ✅) |
| **Schedule** | Daily, 2:30 AM ET (cron `30 6 * * *` UTC during EDT) |
| **Routine ID** | `trig_014R7GNEmkfgQu9KvYWbyhwL` |
| **Source file** | `routines/blog-publish.md` |
| **Connectors** | Notion only |
| **Allowed tools** | Bash, Read, Write, Edit, Glob, Grep, WebFetch |

**What it does:** Picks today's blog topic per `content-calendar.md`, drafts a 1,200–1,800 word post in TCA voice using `skills/tca-content-writer/SKILL.md`, generates featured + in-body images, runs the 8-rule compliance check, schedules to WordPress for 8:00 AM ET, writes a social hooks file to `logs/YYYY-MM-DD-social-hooks.md` for the next-morning social-distribute routine to consume, and logs the run to Notion Operations Hub.

**Inputs:** `content-calendar.md`, `voice/`, `assets/`, `skills/`, recent WP posts (freshness check via WP REST API).
**Outputs:** Scheduled WP post, social hooks file in repo, Notion run log, optional Notion follow-up tasks.

---

### 2. `daily-social-distribute`

| | |
|---|---|
| **Status** | Spec committed — routine not yet created (awaiting Phase 2 of build) |
| **Schedule** | Daily, 8:30 AM ET (cron `30 12 * * *` UTC during EDT) — NOT YET ENABLED. Schedule activates after first manual test run succeeds. |
| **Routine ID** | TBD |
| **Source file** | `routines/social-distribute.md` |
| **Connectors** | Notion only (GHL Social Planner calls go via HTTPS to `services.leadconnectorhq.com` with vault credentials) |
| **Allowed tools** | Bash, Read, Write, Edit, Glob, Grep, WebFetch |

**What it does:** Reads yesterday's blog post (now live on thomasclarkadvisor.com) and the corresponding `logs/YYYY-MM-DD-social-hooks.md` file. Adapts the post into platform-native content per `skills/tca-social-media-standards/SKILL.md`: Facebook post + image, Instagram single feed image, LinkedIn long-form native text post, Pinterest pin #1 + queued pins #2–5 across the next 4 days. Schedules every post via the GHL Social Planner API (`POST /social-media-posting/{locationId}/posts` with `status: "scheduled"`). Pre-flight assertions enforce `status: "scheduled"`, UTC ISO 8601 with Z suffix, ≥5 min lead time, `userId`, and non-empty `accountIds` before any POST. Runs `tca-compliance-check` against all 4 primary posts and all 5 pins. Partial success is acceptable (one platform fails, others continue); rollback is not.

**Inputs:** Yesterday's published WP post, `logs/YYYY-MM-DD-social-hooks.md`, voice/assets/skills, GHL credentials from Notion vault.
**Outputs:** 4 platform posts + 5 Pinterest pins scheduled in GHL Social Planner, Notion Operations Hub run log. No repo writes.

**Credentials in vault (already populated):** `GHL_API_KEY`, `GHL_LOCATION_ID`, `GHL_USER_ID`, `GHL_FB_ACCOUNT_ID`, `GHL_IG_ACCOUNT_ID`, `GHL_LI_ACCOUNT_ID`, `GHL_PIN_ACCOUNT_ID`, `GHL_PIN_OAUTH_ID`, `GHL_PIN_BOARD_ID_*` (5 boards).

**Note:** This routine replaces the previously planned `daily-pinterest-queue-flush` routine. The 5-pin Pinterest queue is folded into this routine's single scheduled-API-calls flow rather than being a separate next-day flush.

---

### 3. `weekly-newsletter-digest`

| | |
|---|---|
| **Status** | Not yet built |
| **Schedule** | Sundays, 7:00 AM ET (cron `0 11 * * 0` UTC during EDT). Sends at 8 AM ET. |
| **Routine ID** | TBD |
| **Source file** | `routines/weekly-newsletter-digest.md` |
| **Connectors** | Notion only (GHL calls go via HTTPS API with vault credentials) |
| **Allowed tools** | Bash, Read, Write, Edit, Glob, Grep, WebFetch |

**What it does:** Drafts a fresh 200–400 word lead essay in TCA voice (no rehashed blog content), pulls the week's published blog posts (Mon–Sat) into a recap section, picks one rotating soft CTA (newsletter / Just in Case Binder / community), runs compliance check, sends as a GoHighLevel broadcast to the TCA Newsletter Subscriber tag in the TCA sub-account.

**Inputs:** Past 7 days of WP posts, voice/assets/skills, rotating CTA list.
**Outputs:** Sent GHL broadcast, archive of the issue committed to `logs/newsletters/YYYY-MM-DD.md`, Notion run log.

**New credentials needed in vault:** `GHL_API_KEY`, `GHL_LOCATION_ID` (TCA sub-account).

---

## Future routines (planned, not yet specified)

These are on the roadmap but not yet designed. When ready to build, add a new section above following the same template, then build the source file in `routines/`.

- **`tca-video-production`** — daily YouTube/Shorts content. Deferred until Higgsfield workflow is locked in.
- **`weekly-content-audit`** — Sundays, audits past week's published content for engagement signals and flags underperforming posts for refresh. Pure analytics + Notion logging, no publishing.
- **`monthly-pinterest-board-curation`** — first of the month, reviews Pinterest analytics, prunes underperforming pins, adjusts board strategy. Read-only analysis + Notion task generation.
- **`quarterly-skills-refresh`** — quarterly check that voice files, anti-patterns, content calendar, and brand assets are current with what's actually publishing well. Generates a refresh-recommendation Notion page.
- **`community-builder`** — TBD, Thomas's planned next skill. Will define when prompt is provided.

---

## Maintenance

When a routine is created, edited, or deprecated, update this file in the same commit. The "Status" line tells future agents whether to trust the routine.

When the architecture conventions section changes (new credential added, new environment, new branch policy), all routines need to be reviewed for impact. Conventions are deliberately small in number — additions are a Thomas decision.
