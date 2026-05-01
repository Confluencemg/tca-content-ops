---
name: tca-social-media-standards
description: Use this skill any time TCA social media content is being created, adapted, or reviewed for Facebook, Instagram, LinkedIn, Pinterest, or YouTube. Defines per-platform content types, frequencies, visual standards, copy standards, hashtag rules, quality gates, and engagement principles. Brand-scoped to TCA only — does NOT apply to Built In The Dark or other CMG properties (those need their own brand-prefixed skill). Trigger this skill from the daily-social-distribute routine, when adapting blog content for social, when planning content calendars, when creating carousels or quote cards or video clips, or when auditing social media output quality. Loads tca-compliance-check internally for any post touching financial topics. Replaces the older social-media-content-standards skill.
---

# TCA Social Media Standards Skill

## Core philosophy

The job is **not** to dump blog links across platforms. The job is to create platform-native content that delivers value in the format each platform's audience expects. Every post must look and feel like it was made specifically for that platform.

The blog is the **source**, not the destination. Every blog post becomes 4-5 distinct platform-native pieces — the same source material, different execution per platform.

---

## Core rules (apply everywhere)

1. **Never just post a link with no context.** Every link share must include a standalone value hook that gives the reader a reason to care before they click.
2. **Adapt, don't copy-paste.** The same blog post becomes a carousel on Instagram, a story-driven text post on Facebook, a thought leadership take on LinkedIn, and 5 distinct pin designs on Pinterest. Never copy-paste copy across platforms.
3. **Brand consistency across platforms.** Use the TCA brand palette below. Thomas's headshot or TCA logo on every branded graphic. thomasclarkadvisor.com linked everywhere.
4. **Compliance check is mandatory.** Any post touching financial topics runs through `tca-compliance-check` before scheduling. Critical findings block publication.
5. **One post at a time per platform.** Never post multiple pieces back-to-back on the same platform. Minimum 3-4 hours between posts.
6. **Disclaimers live at the profile level, not in every post.** Bio/About sections carry the persistent disclosure. Routine educational posts get no inline disclaimer text. Inline disclosures appear ONLY when content includes a specific trigger (testimonial, performance claim, third-party rating, product comparison). The compliance check identifies triggers on a per-post basis.

---

## Brand palette (canonical)

All TCA visual assets use this palette consistently:

| Token | Hex | Use |
|---|---|---|
| Brand Navy | `#1a2947` | Primary anchor color, headlines, dark backgrounds |
| Warm Cream | `#f7f1e3` | Light backgrounds, off-white surfaces |
| Aged Brass | `#9c7a3c` | Accents, highlights, button states, key callouts |
| Ink Brown | `#241e14` | Body text on light backgrounds |

Derived/extended tokens:
- White `#ffffff` for highest-contrast surfaces when warm cream feels too soft
- A muted gray `#d9d4c7` (warm cream darkened) for borders, dividers, secondary surfaces

**Never use:** bright blue, gold (use aged brass instead), pure black for body text (use ink brown), gray-on-gray, or any color outside this palette without explicit approval.

---

## Target audience reminder

- **Primary:** Pre-retirees aged 55-65 — keep it professional, warm, educational. Not a meme-savvy audience. No trendy slang. No excessive emojis. No clickbait.
- **Secondary:** Retirees 65+, active investors 45-55
- **Tertiary:** Younger investors (20s-40s), adult children helping parents
- **Quaternary:** Active traders (only on Trading & Markets content)

Default to writing for primary audience unless the topic specifically targets a different segment.

---

## Voice across all platforms

- Calm, confident, direct
- Educational first, never salesy
- Real examples, not vague hypotheticals
- Plain language, define jargon on first use
- Short sentences. Clear language. Zero hype.
- Occasional dry humor welcome but never forced
- Asks questions to drive engagement when natural ("What's your biggest retirement planning question?")
- Acknowledges complexity without intimidating

For full voice patterns, see `voice/tca-voice-examples.md` in the content-ops repo.

---

## FACEBOOK — Thomas Clark Advisor Page

### Audience behavior
This is where the primary audience (55-65) is most active. They read longer posts, engage with educational content, share things that help their friends. Reach is challenging — Facebook averages ~1.65% organic reach for Pages — so quality over quantity matters more here than anywhere else.

### Posting cadence
**5 posts per week, not 7.** Higher volume on a low-reach platform doesn't help. Better to do fewer, higher-quality posts.

### Per-day mix (Mon–Fri default)

| Day | Type | Link day or standalone |
|---|---|---|
| Mon | Blog feature post | Link |
| Tue | Quick tip / Did you know | Standalone |
| Wed | Blog feature post | Link |
| Thu | Engagement question | Standalone |
| Fri | Personal / Behind the scenes OR Quick tip | Standalone |

This gives 2 link days and 3 standalone days per week — calibrated to Facebook's reach reality.

### Content types

**Type 1: Blog feature post (2x/week, link days)**
- Adapted from a blog post — not just a link drop
- Structure:
  - Hook line (1 sentence, surprising fact or curiosity gap)
  - 2-3 sentences of standalone value from the blog (reader gets something even if they never click)
  - "Read the full breakdown here: [link]"
  - Branded image (featured image from the blog post)
- Length: 150-300 words
- Always include an image — never naked link

**Type 2: Quick tip / did you know (1-2x/week, standalone)**
- Short, punchy educational post — no link
- Structure:
  - "Did you know..." or "Quick tip:" or "One thing most people get wrong about [topic]:"
  - 2-4 sentences of value
  - End with engagement prompt: "Have questions? Drop them in the comments."
- Branded quote card or simple graphic
- Length: 50-150 words

**Type 3: Engagement question (1x/week, standalone)**
- Ask a question related to retirement planning
- Examples: "What's your biggest concern about retirement?", "At what age are you planning to claim Social Security?", "What's one financial topic you wish someone explained in plain English?"
- No link, no image required (text-only posts get algorithmic boost on Facebook)
- Length: 1-3 sentences

**Type 4: Personal / behind the scenes (1x/week, standalone)**
- Thomas at the office, at a seminar, with the team, or a personal reflection
- Humanizes the brand
- Authentic — not polished marketing
- Photo of Thomas preferred (real, not stock)
- Length: 50-150 words

### Facebook quality gate
- [ ] Post is NOT just a link with no context
- [ ] Hook line grabs attention in first sentence (visible before "see more" cutoff)
- [ ] Standalone value even without clicking a link
- [ ] Branded image included (except text-only engagement posts)
- [ ] No financial jargon without explanation
- [ ] No specific investment recommendations
- [ ] Compliance disclaimer reference present (link to disclosures page, or About section)
- [ ] No fear-mongering or urgency language
- [ ] No more than 1-2 emoji (sparingly)

### Facebook compliance disclaimer placement
Set once on the profile, not on every post:
- Page's About section carries the full disclaimer with link to thomasclarkadvisor.com/disclosures
- Profile bio includes brief credential note: "Series 65 IAR. Educational content."
- Routine educational posts: no inline disclaimer text in body copy
- Triggered disclosures (only when content includes any of these):
  - Testimonials or endorsements → SEC Marketing Rule disclosure language required in the post
  - Performance claims or projections → don't make these in the first place; if a legitimately guaranteed instrument is referenced, source the guarantee
  - Third-party ratings → disclosure language required if cited
  - Specific product comparisons → balanced presentation language required
- The compliance check enforces these triggers on a per-post basis

---

## INSTAGRAM — @thomasclarkadvisor

### Audience behavior
Instagram is visual-first. The 45-55 secondary audience is growing here. **Educational carousels are the highest-engagement format on the platform** (1.92% engagement vs. 1.74% for Reels vs. 1.26% for single images). The algorithm rewards saves, shares (especially DM shares — weighted 3-5x higher than likes), and watch time.

### Posting cadence
**4 posts per week:**
- 2 educational carousels (highest priority format)
- 1 quote card or single image
- 1 short-form video / Reel

### Content types

**Type 1: Educational carousel (2x/week — highest priority format)**

This is the workhorse. Carousels live longer in feed and accumulate engagement over time.

- **Slide count:** 8-10 slides (peak reach observed around slide 13 for longer sequences; 6-10 is the sweet spot)
- **Aspect ratio:** 1080×1350 portrait (4:5) — NOT 1080×1080 square. Portrait dominates mobile screen real estate.
- **Slide structure:**
  - Slide 1: Bold hook headline + branded background (this is the thumbnail — must stop the scroll, ≤12 words)
  - Slides 2-3: Build curiosity, set up the problem or context
  - Slides 4-7: One key insight or point per slide. Large readable text. Under 30 words per slide.
  - Slides 8-9: Example, application, or "so what / why it matters"
  - Final slide: CTA — "Save this for later" or "Link in bio for the full breakdown"
- **Design:** Brand Navy `#1a2947` backgrounds with cream `#f7f1e3` text, OR cream backgrounds with navy text. Aged brass `#9c7a3c` for accents. Clean, minimal, large fonts.
- **Brand mark:** small TCA monogram or wordmark on every slide in the same position
- **Caption:** 150-300 words expanding on the carousel topic, 5-10 hashtags at the end (mix broad and niche)

**Critical brand consistency rules for carousels:**
- Same background treatment across all slides in a single carousel
- Same font family, same text positioning template
- Slide 1 may have a slightly different layout (it's the hook) but slides 2-N must match
- No stock-photo-with-text-slapped-on aesthetic. Designed-feeling, not generated-feeling.

**Type 2: Quote card / single image (1x/week)**
- Single powerful statement from Thomas or a key stat
- **Aspect ratio:** 1080×1350 portrait (4:5)
- **Design:** Branded template — navy/cream/brass, Thomas's name and credential, thomasclarkadvisor.com
- Examples:
  - "The order you draw from your retirement accounts matters more than most people realize." — Thomas Clark
  - "68% of Americans don't know their estimated Social Security benefit. Do you?"
- Caption: 50-150 words expanding on the quote, soft CTA

**Type 3: Short-form video / Reel (1x/week)**
- 30-60 second clip — Thomas speaking directly to camera, or a narrated visual
- Topics: One quick tip, one myth busted, one "here's what I'd do" scenario
- Captions/subtitles required (most people watch without sound)
- Vertical format: 1080×1920px
- Caption: 50-100 words + 5-10 hashtags

### Instagram quality gate
- [ ] Carousel slides have large, readable text (test: can you read it on a phone screen at arm's length?)
- [ ] No more than 30 words per carousel slide
- [ ] Slide 1 is a scroll-stopper (bold hook, not generic)
- [ ] All slides 1080×1350 (4:5 portrait), not square
- [ ] Brand colors consistent (navy/cream/brass)
- [ ] Brand mark on every carousel slide
- [ ] Caption provides standalone value (not just "swipe →")
- [ ] Hashtags included (5-10 for feed, 5-10 for Reels)
- [ ] Save-worthy framing for carousels ("Save this for the next time you...")
- [ ] No stock photo feel

### Hashtag bank (rotate 5-10 per post)
Core (always include 3-5):
`#RetirementPlanning #SocialSecurity #RetirementIncome #FinancialPlanning #RetirementTips`

Niche (rotate 2-5):
`#SocialSecurityTips #RetirementIncomePlanning #MedicarePlanning #BucketPlanning #RetirementStrategy #FinancialLiteracy #MoneyAfter50 #RetireWell #RetirementReady #PreRetirement`

Local (include 1-2 for FB/IG when relevant):
`#CharlotteFinancialAdvisor #NCRetirement`

### Instagram compliance disclaimer placement
Set once on the profile, not on every post:
- Bio carries credential note + link to thomasclarkadvisor.com/disclosures
- Routine educational carousels and captions: no inline disclaimer text
- Triggered disclosures only (same triggers as Facebook above)
- The compliance check enforces these triggers on a per-post basis

---

## LINKEDIN — Thomas Clark profile

### Audience behavior
LinkedIn is professional authority building. The audience here is other advisors, centers of influence (CPAs, attorneys), and professional pre-retirees. **Posts with external links experience approximately 60% reach reduction** as LinkedIn prioritizes keeping users on platform. The "link in first comment" workaround is now also penalized.

### Posting cadence
**3 posts per week**, mix calibrated to LinkedIn's reach reality:
- 2 thought leadership / standalone posts (no link)
- 1 blog share with substantial commentary (link day — reach will be lower, but gives the post longer-tail discovery)

### Content types

**Type 1: Thought leadership text post (2x/week, no link — preferred)**
- Long-form text post (no image required — text-only posts get strong organic reach on LinkedIn)
- Structure:
  - Strong opening line (hook — LinkedIn shows first 2 lines before "see more")
  - Personal insight or contrarian take on a retirement planning topic
  - 3-5 short paragraphs
  - End with a question or discussion prompt that invites real responses
- Length: 200-500 words
- Tone: Professional but Thomas's warm voice. "Industry peer" not "sales pitch."
- 1-3 hashtags max at end (more than 5 triggers spam signals)

**Type 2: Document carousel / PDF post (1x/week, optional but high-performing)**
- LinkedIn document posts (PDF carousels) achieve 6.60% engagement — the highest of any LinkedIn format
- 5-10 slides converted to PDF
- Same content adapted from a blog post or original thought leadership
- 1080×1350 (4:5 portrait) per slide

**Type 3: Blog share with commentary (1x/week, link day)**
- Share a blog post link WITH substantial commentary (not just "check out my latest")
- Structure:
  - 2-3 sentences of Thomas's personal take on the topic
  - Why this matters right now
  - Link to the blog post (in body — first-comment workaround is also penalized)
  - Optional: tag relevant connections or organizations
- Length: 100-200 words
- Accept that reach will be ~60% of standalone posts. The trade-off is search discoverability and bottom-of-funnel referral.

### LinkedIn quality gate
- [ ] Opening line strong enough to earn the "see more" click
- [ ] Tone professional but not corporate
- [ ] No hard sales language
- [ ] Value-first — reader gets insight from the post itself
- [ ] 1-3 hashtags only (never 5+)
- [ ] No emojis or very sparing use (1-2 max)
- [ ] If link day: link in body, not in first comment

### LinkedIn compliance disclaimer placement
Set once on the profile, not on every post:
- Profile About section carries the full disclaimer with link to thomasclarkadvisor.com/disclosures
- Headline includes credential: "Series 65 IAR | Retirement Planning Insights" (or similar)
- Routine thought leadership and blog share posts: no inline disclaimer text
- Triggered disclosures only (same triggers as Facebook/Instagram)
- The compliance check enforces these triggers on a per-post basis

---

## PINTEREST — Thomas Clark Advisor

### Audience behavior
Pinterest is a **visual search engine, not a social network**. Users come to Pinterest with clear intent — searching for solutions, ideas, frameworks. **97% of searches are non-branded.** Pin lifespan is months to years (vs. hours on Instagram). This is the **primary organic traffic source** for thomasclarkadvisor.com.

### Strategic principle
**Every pin links.** That's the entire point of the platform — drive traffic. Unlike LinkedIn or Facebook, "no-link days" make no sense on Pinterest.

### Posting cadence (steady state, after first week)
**5 fresh pins per day, distributed across rotating URLs.**

The mechanic:
- Each blog post gets 5 distinct pin designs (different headline framing, different visual treatment, different angle)
- Pin #1 fires the day the blog post goes live
- Pins #2-5 are queued for the next 4 days
- Steady state from day 5: 5 pins firing per day across 5 different blog post URLs (today's + previous 4 days')

This is run by the `daily-social-distribute` and `pinterest-queue-flush` routines. The skill defines the pin standards; the routines handle the queue mechanics.

**Critical rule:** never pin all 5 pins to the same URL on the same day — triggers Pinterest's spam filters. The 24+ hour stagger across 5 days for each URL is what keeps the account clean.

### Pin specifications

- **Aspect ratio:** 1000×1500 (2:3) — Pinterest's algorithm penalizes other ratios
- **Headline as text overlay:** 8-12 words, scannable, keyword-rich, written for Pinterest search
- **Design:** Brand Navy `#1a2947` backgrounds with cream `#f7f1e3` text, OR cream backgrounds with navy text. Aged brass `#9c7a3c` for headline accents. Bold typography that reads on a small phone preview.
- **Brand mark:** TCA wordmark or monogram in a consistent corner position
- **Logo placement:** Same position on every pin — builds visual recognition over time
- **Pin description:** 200-500 chars, keyword-rich, written for Pinterest search not human reading
- **Link:** Source blog post URL with UTM tracking: `?utm_source=pinterest&utm_medium=social&utm_campaign=<post-slug>&utm_content=pin-<N>`

### The 5 pin angles (generated per blog post)
For every blog post, 5 distinct pin angles are produced (the blog routine extracts these into the social hooks file):

1. **Distinct headline framing** — straightforward direct claim
2. **Different audience hook** — same content, different "this is for you" framing
3. **Different value proposition** — what the reader gets vs. what they avoid
4. **Question-led hook** — pin opens with a specific question
5. **List-led hook** — "5 things..." or "3 ways..." or numbered framing

Each pin uses a different visual treatment alongside the headline change — different background color emphasis, different layout, different typographic hierarchy. The point is **5 distinct designs**, not 5 variations of the same design.

### Pinterest board mapping
Each pin posts to the board matching the source post's category:

| Category | Pinterest Board |
|---|---|
| Market Insights | Market Commentary |
| Social Security | Social Security Strategy |
| Retirement Planning | Retirement Planning |
| Bucket Planning | Bucket Planning Method |
| Financial Education | Personal Finance |
| Trading & Markets | Trading & Markets |

### Pinterest quality gate
- [ ] Pin is exactly 1000×1500 (2:3 ratio)
- [ ] Headline overlay is 8-12 words, readable on phone preview
- [ ] Brand colors consistent (navy/cream/brass)
- [ ] Brand mark in consistent corner position
- [ ] Pin description is keyword-rich, written for Pinterest search
- [ ] Link includes UTM tracking
- [ ] Posted to category-matched board
- [ ] No more than 1 pin scheduled to same URL within 24 hours
- [ ] No hashtag stuffing in description (Pinterest doesn't reward hashtags the way IG does)

### Pinterest compliance disclaimer placement
- Pin destination (the blog post) carries the disclaimer
- Pin description itself doesn't need disclaimer language — keyword-rich utility is more important
- Profile bio carries the full disclaimer with link to disclosures page

---

## YOUTUBE — placeholder section

**Status:** YouTube standards are deferred until Thomas finalizes a video production workflow he's comfortable automating. Descript talking head avatar approach was evaluated and set aside (Thomas's assessment: avatar quality not where it needs to be). Higgsfield MCP being set up for ad-hoc creative video work, but not yet a recurring routine workflow.

**When YouTube is ready to be added back:**
- Channel: Thomas Clark Advisor (or Thomas Clark | Retirement Planning)
- Cadence target: 2-3 videos per week (per project memory; not all 5 weekly blog posts get video — prioritize the most engaging, visual, shareable topics)
- Format mix per project memory: Descript-based talking head primary (currently paused), Higgsfield cinematic 2x/month flagship, Higgsfield Vibe motion graphics weekly
- Specs to define when ready: thumbnail standards, description template, tag strategy, playlist structure, Shorts vs. long-form mix

**Until then, YouTube is not part of the daily-social-distribute routine.** Adding it back is a Thomas decision when his workflow is ready.

---

## CONTENT ADAPTATION WORKFLOW (per blog post)

When a new blog post is published, the routine creates from this skill's guidance:

| Platform | Output | Source |
|---|---|---|
| Facebook | 1 blog feature post (or skip on standalone day) | Blog post + hooks file |
| Instagram feed | 1 caption + 1 image (1080×1350) | Hooks file |
| Instagram carousel | 1 carousel (8-10 slides, 1080×1350 each) | Carousel arc from hooks file |
| LinkedIn | 1 thought leadership post (or 1 blog share on link day) | Hooks file |
| Pinterest | 5 pin designs (1 fires today, 4 queued for days 2-5) | 5 pin angles from hooks file |
| YouTube | (deferred — see placeholder section above) | — |

**Total per blog post:** ~9 distinct platform-native pieces of content (FB post, IG feed post + image, IG carousel + 8-10 slide images, LinkedIn post, 5 Pinterest pins).

The blog is the source. The skill defines the standards. The routine executes.

---

## Content adaptation principles (cross-platform)

When adapting one blog post into multiple platform pieces, hold these principles:

1. **Each piece must be valuable on its own.** Someone who only sees the Facebook post should learn something. Someone who only swipes the IG carousel should learn something. The blog link is the upgrade, not the only payoff.

2. **Different angles per platform.** Don't lead Facebook, Instagram, and LinkedIn with the same opening line. Each platform's audience expects a different entry point.

3. **Visual treatment matches platform mood.** LinkedIn = professional minimal. Instagram = polished editorial. Pinterest = scannable utility. Facebook = warm and personal. The same brand palette appears on all of them, but the emotional register shifts.

4. **CTA fits the platform.** Pinterest = always link to the blog post. LinkedIn standalone = engagement question. Instagram = "save this" or "link in bio." Facebook = mix of "read more" and "share your thoughts." Never advisory consultation.

5. **One CTA per piece.** Multiple CTAs dilute conversion. If a piece needs a CTA, make it one clear action.

---

## Compliance integration

This skill loads `tca-compliance-check` for any post touching financial topics. The compliance check runs as a final pass before scheduling.

The calling routine invokes the skill, gets the social pieces back, runs each through compliance check, and acts on the result:

- **PASS** → schedule and publish
- **FAIL_AUTOFIX_APPLIED** → use auto-fixed version, note in daily summary
- **FAIL_BLOCK** → don't publish that platform piece, log Critical Notion task with the draft

**Light social posts (engagement questions, behind-the-scenes, personal posts that don't touch financial topics)** can skip the formal compliance check — they're not making claims that require verification. Use judgment: if there's any financial topic discussion at all, run the check.

---

## Engagement management (manual, for reference)

The routine doesn't auto-respond to comments or DMs. Engagement is human-handled by Thomas. Standards for when Thomas (or future VA) handles engagement:

- Respond to comments on all platforms within 24 hours
- Answer questions with helpful, educational responses
- If a comment asks for specific personal financial advice, respond with general education + link to the disclosures page; never provide personalized advice in public comment
- Never engage with troll/bait comments — delete or hide
- Flag any compliance-sensitive interactions or negative comments for review

This skill doesn't automate engagement — it just defines the standards for when humans do it.

---

## What this skill never does

- Apply the same posting frequency or content mix to every platform (each platform has different rules)
- Default to "link to blog post" framing on every post (per-platform link strategy varies significantly)
- Use 1080×1080 square for IG carousels (always 1080×1350 portrait)
- Use anything other than 1000×1500 (2:3) for Pinterest pins
- Use more than 3 hashtags on LinkedIn
- Use more than 10 hashtags on Instagram (5-10 is the sweet spot)
- Use hashtags on Pinterest (Pinterest doesn't reward them)
- Skip the compliance check on financial-topic posts
- Use stock photos for IG carousels (designed slides only)
- Crowd carousel slides with too much text (one idea per slide, max 30 words)
- Use brand colors outside the canonical palette (navy/cream/brass/ink-brown)
- Use advisory consultation CTAs anywhere
- Use fear-mongering or urgency manipulation
- Default to listicle structure across all platforms (only when the topic genuinely calls for it)
- Try to automate engagement responses (those are human-handled)
