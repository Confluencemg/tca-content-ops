# TCA Content Ops — Routine Instructions

This repo holds the autonomous content pipeline for **Thomas Clark Advisor**, the personal brand and content platform of Thomas Clark, operating under **Confluence Media Group LLC** (a structurally separate publishing entity from Confluence Capital Management, the registered investment advisory firm).

Four Claude Code Routines run from this repo:
- `daily-blog-publish` — writes, illustrates, and schedules one blog post each morning (2:30am ET). Does NOT send email broadcasts (newsletter is weekly, not per-post).
- `daily-social-distribute` — adapts the published post into platform-native social content and schedules it across channels (8:30am ET); fires Pinterest pin #1 immediately and queues pins #2-5 for the next 4 days
- `pinterest-queue-flush` — daily worker that processes queued Pinterest pins from prior days, generates their images, and posts them (8:45am ET); produces the steady-state ~5-pins-per-day-across-rotating-URLs Pinterest cadence
- `weekly-newsletter-digest` — sends one newsletter per week to TCA Newsletter Subscribers (Sundays 7am ET, sends 8am); fresh 200-400 word lead essay from Thomas plus recap of the week's blog posts

All four share the context, voice, and compliance posture defined here. Routine-specific instructions live in `routines/`.

---

## Routine architecture

Conventions all four routines follow.

**Credentials live in the Notion vault.** No credentials in the repo, no env vars on the routine config. Each routine fetches credentials at runtime by searching Notion for a private page titled exactly `Routine Vault — Credentials` and parsing the fenced code block on that page. Hold credentials in memory only — never write them to the repo, `logs/`, run summaries, or other Notion pages. See `SECRETS.md` for which keys each routine needs and how to refresh them.

**Routines commit directly to `main`.** No feature branches, no PRs, no merge steps. Pattern: `git add <files>` → `git commit -m "..."` → `git push origin main`. The "Allow unrestricted git push" toggle on each routine's Permissions tab grants this access. Apply this pattern in every routine that writes back to the repo (social hooks file, Pinterest queue, newsletter archive, etc.).

**Image generation: OpenAI primary, OpenRouter fallback.** Default to OpenAI `gpt-image-2` at `/v1/images/generations` (requires org verification on the OpenAI account — without it the API returns 403 `organization-must-be-verified`). On failure, fall back to OpenRouter using model `openai/gpt-5.4-image-2` against `/api/v1/chat/completions` with `modalities: ["image", "text"]` (verified working 2026-05-01). If both fail: publish without images, log a Critical Notion task, never block the publish.

**Notion Operations Hub is the run log.** Every routine writes a run-summary task to Operations Hub (data source ID `c96ec729-0d1f-4eb3-a88a-bf1e586b4dfe`). Failures and follow-ups also write Critical/High-priority tasks.

**Network egress.** Routines run in the cloud environment whose tier is set to `full` (not `trusted`). The `trusted` tier's restricted allowlist does not have outbound access to `thomasclarkadvisor.com`, `api.openai.com`, or `openrouter.ai`. MCP traffic (Notion) is unaffected by tier.

---

## CRITICAL: Load TCA skills before doing anything

Before drafting any content, every routine MUST load Thomas's TCA-specific skills. These skills encode hard-won decisions about voice, compliance, and platform strategy that override any general best practices the model might default to.

**Primary skills (load by routine):**

For `daily-blog-publish`:
- **`tca-content-writer`** — the authoritative skill for blog content. Defines voice, structure, pillars, hypothetical example construction, article workflow. Internally loads `tca-compliance-check`.
- `gpt-image-2-director` — for image generation prompts
- `seo-audit` — for SEO-relevant pieces (optional)

For `daily-social-distribute`:
- **`tca-social-media-standards`** — defines per-platform standards (content types, frequencies, visual rules, copy rules, hashtag rules, quality gates per platform). Brand palette + Pinterest standards + per-platform link strategy all defined here.
- **`tca-compliance-check`** — for any social post touching financial topics
- `gpt-image-2-director` — for carousel slide prompts and image generation
- `marketing-psychology` — for engagement framing
- `copywriting` — for headlines and CTAs

For `pinterest-queue-flush`:
- `gpt-image-2-director` — for pin image prompt construction
- `copywriting` — for any pin headline tweaks

For `weekly-newsletter-digest`:
- **`tca-content-writer`** — for the lead essay
- **`tca-email-campaigns`** — for assembly, audience targeting, GHL send
- `tca-compliance-check` — loaded internally by both above skills

For ad-hoc product/lead magnet copy:
- **`tca-sales-copy`** — for sales-adjacent content (product pages, lead magnet pitches, community CTAs)
- `tca-compliance-check` — final pass

**Skills are authoritative for their domain.** If a skill instruction conflicts with this CLAUDE.md, the skill wins for matters within its scope. CLAUDE.md is authoritative for compliance posture and brand identity.

**If a skill won't load:**
- For `tca-content-writer` failure: log Critical Notion task, stop the routine. Do not attempt to write without it.
- For `tca-compliance-check` failure: log Critical, stop. No content publishes without compliance check.
- For other skills: log High-priority, proceed with built-in defaults from CLAUDE.md.

---

## Brand identity (read first, every run)

**Thomas Clark Advisor** is a creator-economy publishing brand. Thomas is a Series 65 Investment Advisor Representative whose credential informs content credibility — not a financial advisor whose content is a lead funnel for advisory services. Products and content are standalone publishing assets with their own commercial logic.

This framing matters because it determines compliance posture, voice, and CTAs. Treat every piece of content as standalone publishing by an author who happens to be a financial professional.

**Multi-generational target audiences:**
- Pre-retirees aged 55–65 planning their retirement transition
- Retirees managing income and distribution
- Younger investors learning money habits and wealth-building (20s–40s)
- Active traders learning markets
- Families seeking organizational and educational tools

---

## Voice profile

Thomas Clark sounds like:
- **Conversational but authoritative** — explains complex topics in plain language
- **Direct and confident** — strong opinions about retirement planning, especially around annuities and guaranteed income
- **Empathetic** — readers are real people planning real financial lives
- **Not corporate or stiff** — this is a personal brand, not a big firm
- **Educational first** — teach before promoting any product
- **Specific, not generic** — always tie back to bucket planning when retirement is the topic, or trading background when markets is the topic

For full voice samples and anti-patterns, see `voice/tca-voice-examples.md` and `voice/anti-patterns.md`.

---

## Planning philosophy (the differentiator)

Thomas's retirement planning approach centers on **bucket planning** — Now, Soon, and Later buckets:

- **Now bucket** — current liquidity and living expenses
- **Soon bucket** — guaranteed income floor using Social Security, pensions, and income-focused Fixed Index Annuities
- **Later bucket** — market growth, aligned with risk tolerance and legacy goals

**Strong opinions Thomas holds:**
- Variable Annuities are counterproductive for retirement income planning
- Fixed Index Annuities are for income, not growth
- Social Security claiming strategy is one of the highest-leverage decisions a pre-retiree makes
- A guaranteed income floor changes everything about how you experience retirement
- The transition from accumulation to distribution requires a fundamentally different mindset

When markets/trading is the topic, lean on Thomas's actual trading background — pattern recognition, risk management, position sizing, the psychological side.

---

## Compliance posture (apply on every piece)

Confluence Media Group LLC operates as a structurally separate publishing entity from Confluence Capital Management. Content is standalone publishing by an author who happens to be a financial professional.

Altitude Capital Management (CCO: Charles Johnson) has reviewed the existing thomasclarkadvisor.com content. New posts following established patterns do not require per-piece review. Thomas makes his own compliance calls.

**Hard rules (every post must pass):**
1. **No specific investment recommendations.** Educational discussion of categories, strategies, concepts only.
2. **No performance claims or projections.** No "you'll earn X%." Use ranges, hypotheticals clearly labeled, language like "could," "may," "in some cases."
3. **No guarantees outside legitimately guaranteed instruments.** Social Security, pensions, FIA income riders can be discussed as guaranteed. Market returns cannot.
4. **No advisory CTAs.** Never link to advisory services. Never use "schedule a consultation." Acceptable CTAs: read related blog posts, sign up for newsletter, get the free guide, buy a product, join the community.
5. **Author bio uses "Series 65 Investment Advisor Representative."** Never "Fiduciary Advisor," "Wealth Manager," or any title implying an advisory relationship with the reader.
6. **Disclaimer footer on every post.** Pull from `assets/disclaimers/standard-blog-footer.md`.
7. **Hypothetical examples are clearly labeled.** "Consider a hypothetical retiree named Bob who…" not "My client Bob…"
8. **No testimonials in blog posts** without proper SEC Marketing Rule disclosure language. Default: don't include them.

**If a draft fails any of these, fix it before publishing.** Don't publish and ask permission.

---

## Content categories

Every post belongs to exactly one primary category and may have one secondary:

| Category | Description | Audience segment |
|---|---|---|
| Market Insights | Daily/weekly market commentary, economic trends | All segments |
| Social Security | Optimization, claiming strategies, spousal benefits, timing | Pre-retirees, retirees |
| Retirement Planning | Bucket planning, income planning, FIA for income | Pre-retirees, retirees |
| Bucket Planning | The Now/Soon/Later framework — brand differentiator | Pre-retirees, retirees |
| Financial Education | Broader literacy for multiple segments | Younger investors, all |
| Trading & Markets | Active trading, market mechanics, Thomas's trading background | Traders |

Default day-of-week rotation if `content-calendar.md` doesn't override:
- **Monday** — Market Insights (tied to weekend/weekly market events)
- **Tuesday** — Social Security
- **Wednesday** — Retirement Planning or Bucket Planning (alternate weekly)
- **Thursday** — Financial Education
- **Friday** — Trading & Markets

---

## Tools and connectors available to routines

- **WordPress REST API** — `https://thomasclarkadvisor.com/wp-json/wp/v2/` with application password auth (credentials fetched from Notion vault at runtime — see Routine architecture above)
- **OpenAI API** — for GPT Image 2 image generation (`gpt-image-2`), with OpenRouter fallback `openai/gpt-5.4-image-2` per Routine architecture
- **Notion MCP** — Operations Hub database for logging
  - Database ID: `a71e9baf-21ba-4605-bd22-9b2e1f21ce16`
  - Data source ID for create-pages: `c96ec729-0d1f-4eb3-a88a-bf1e586b4dfe`
- **GHL API** — for triggering email broadcasts via webhook
- **Web search** — built-in
- **Pinterest API** — for pin scheduling (or Metricool fallback)
- **Meta Graph API** — for Facebook and Instagram (feed + carousel)
- **LinkedIn API** — for LinkedIn post scheduling

If a connector is unavailable, log the failure to Notion and continue with what you can complete. **Never block the entire pipeline because one channel is down.**

---

## Failure handling (both routines)

1. **Log every failure to Notion.** Operations Hub task with `Status: Blocked`, `Type: Tech`, `Owner: Thomas`, clear `Notes`.
2. **Continue with partial completion.** If image generation fails, publish without images and log it. If LinkedIn fails, publish to FB/IG and log the LinkedIn miss.
3. **Never publish broken content to live channels.** Better to draft and log than to publish something malformed.
4. **Send a daily summary** at end of run: what shipped, what was skipped, what failed. To Slack if connected, otherwise email.

---

## Inbox folder — Thomas-supplied assets

Thomas may pre-make images, carousels, or pins in ChatGPT (or elsewhere) when he has time. The `inbox/` folder is where he drops them. **Both routines check `inbox/` first before generating any visual asset via the API.** If a matching asset exists, use it. If not, generate.

**Folder structure:**

```
inbox/
├── blog-images/        ← featured + in-body images (1200×630 featured, 1200×800 body)
├── pinterest-pins/     ← pre-made 1000×1500 pins
├── ig-carousels/       ← carousel sets, one subfolder per set (e.g., bucket-planning-101/)
├── ig-feed-images/     ← 1080×1350 IG feed images
└── README.md           ← naming conventions and sidecar JSON spec
```

**Matching logic — sidecar JSON file:**

Each image in the inbox has a sidecar `.json` file with the same base name describing what it's for. Example: `pinterest-pins/social-security-claiming-2026.png` paired with `pinterest-pins/social-security-claiming-2026.json`:

```json
{
  "type": "pinterest_pin",
  "topic": "social security claiming age",
  "headline": "The Real Cost of Claiming Social Security at 62",
  "category": "Social Security",
  "use_with_post_slug": null,
  "use_by_date": "2026-06-30"
}
```

**Two ways to use the inbox:**

1. **Topic-matched** — `topic` and `category` set, `use_with_post_slug` is `null`. The routine pulls this asset whenever it's working on a post matching the topic/category. Most flexible.

2. **Bound to a specific scheduled post** — `use_with_post_slug` is set (e.g., `"why-bucket-planning-beats-the-3-percent-rule"`). The routine grabs this asset only when generating that specific post.

**After consuming an inbox asset:**

The routine moves the file from `inbox/` to `inbox/used/<YYYY-MM>/` and updates the sidecar JSON with a `used_in` field pointing to the published post URL. This prevents re-use and gives Thomas a record of what landed where.

**If the inbox has more pre-made assets than today's routine needs:**

Leave them in inbox/ — they'll be picked up by future runs as topics match. The routine's content calendar logic should also check inbox/ when picking topics: if Thomas has dropped 5 pre-made pins about Social Security claiming, that's a soft signal to bump Social Security up the calendar to use the inventory.

**Failure modes for inbox:**

- Sidecar JSON malformed → log Medium-priority Notion task with file path, ignore the asset, generate fresh one
- Image dimensions don't match expected ratio → log Low-priority warning, use anyway (Thomas may have intentionally varied)
- `use_by_date` has passed → log Low-priority warning, move to `inbox/expired/`, don't use
- `use_with_post_slug` set but post never publishes → asset stays in inbox forever (acceptable)

---

## File structure

```
tca-content-ops/
├── CLAUDE.md                          ← you're reading it
├── routines/
│   ├── blog-publish.md                ← daily-blog-publish routine (2:30am)
│   ├── social-distribute.md           ← daily-social-distribute routine (8:30am)
│   ├── pinterest-queue-flush.md       ← pinterest-queue-flush worker (8:45am)
│   └── weekly-newsletter-digest.md    ← weekly-newsletter-digest routine (Sundays 7am)
├── voice/
│   ├── tca-voice-examples.md          ← voice samples
│   └── anti-patterns.md               ← what to avoid
├── content-calendar.md                ← topic rotation
├── assets/
│   ├── disclaimers/
│   │   └── standard-blog-footer.md    ← compliance disclaimer block
│   ├── author-bio.md                  ← bio block
│   └── image-prompts/
│       └── gpt-image-2-style-guide.md ← brand-aligned image prompts
├── inbox/                             ← Thomas drops pre-made assets here
│   ├── blog-images/
│   ├── pinterest-pins/
│   ├── ig-carousels/
│   ├── ig-feed-images/
│   ├── used/                          ← consumed assets archived here
│   └── README.md                      ← sidecar JSON spec
├── logs/
│   ├── (routine run summaries — gitignored)
│   ├── pin-queue-*.md                 ← active Pinterest pin queues (one per blog URL)
│   ├── pin-queue-archive/             ← completed queue files archived here by month
│   ├── newsletter-cta-history.md      ← CTA rotation tracking for weekly newsletter
│   └── <date>-social-hooks.md         ← daily handoff from blog routine to social routine
└── README.md
```

When a routine needs context, it reads the files it needs — not the whole repo. Keep files focused and short.
