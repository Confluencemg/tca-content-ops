---
name: tca-email-campaigns
description: Use this skill whenever email or SMS copy is needed for any Confluence Financial Group property — TCA newsletter, product launches, lead magnet sequences, community onboarding, tripwire/cart abandonment, seminar follow-up paths A through G, and SMS variants. Brand-scoped to TCA editorial newsletter and CMG product/community work; seminar paths A-G also covered because seminars currently run on the CCM side and feed into the TCA newsletter via mirrored contacts. Trigger even on partial requests ("draft a subject line", "rewrite this email", "what should the cart abandonment sequence say") and informal phrasing. Handles voice calibration, category-specific structure, A/B variant generation, and a mandatory final compliance pass via the tca-compliance-check skill. Default register is calm-confident-direct; never hyped, never panicked, never salesy.
---

# TCA Email Campaigns Skill

## Core philosophy

Every email or SMS this skill produces is **conversion-focused but compliance-bound**. The reader is a real person — most often a pre-retiree aged 55-70 — who chose to be on this list. Treat their attention as something earned every time, not as something owed.

Voice is identical to TCA blog voice: calm, direct, authoritative, warm. The reader should feel like Thomas is talking to them across a desk, not pushing a product. If a piece of copy feels salesy, urgency-driven, or hyped, it's wrong for this brand — even if it would convert better short-term. Long-term audience trust is the brand asset.

---

## When to trigger this skill

- Weekly newsletter (Sunday 7am ET — see `routines/weekly-newsletter-digest.md`)
- Product launch email sequences (any TCA/CMG product)
- Lead magnet welcome sequences
- Community onboarding sequences
- Tripwire / cart abandonment sequences
- Seminar follow-up paths A through G (CCM side, copy still produced by this skill)
- SMS variants for any of the above where appropriate
- Ad-hoc one-off email requests

If the request is for general copywriting that's not email/SMS (e.g., a sales page, a blog post), use a different skill (`tca-sales-copy` or `tca-content-writer`).

---

## Audience

- **Primary:** Pre-retirees aged 55-70 actively planning retirement transitions
- **Secondary:** Retirees managing income, distribution, healthcare
- **Tertiary:** Younger investors (20s-40s) for some lead magnet flows
- **Tertiary:** Active traders for trader-specific sequences

Write at an 8th-grade reading level. Respect intelligence; keep language accessible. Most copy targets the primary audience by default — newsletter and product sequences in particular.

---

## Voice profile

**Do:**
- Short paragraphs. One idea per paragraph.
- Single, clear CTA per email. The reader should never wonder what to do next.
- Warm and authoritative — trusted advisor explaining something over coffee
- First-person when appropriate ("I noticed..." / "Here's what I'd do...")
- Specific examples and numbers, not vague claims
- Plain English. Define jargon on first use.

**Don't:**
- Walls of text
- Multiple competing CTAs in one email
- Salesy or hype-driven language
- Generic openers ("In today's market..." / "As a valued subscriber...")
- "Click here" or "Read more" (use specific action verbs: "See the breakdown," "Get the guide," "Join us Tuesday")
- Fear-based urgency ("time is running out," "you can't afford to wait," "don't miss this")
- Manufactured scarcity in editorial content (product launches with real deadlines are different — use real urgency, not fake)
- Promise returns, outcomes, or specific financial results
- Fabricate testimonials or imply endorsement

---

## Output format (universal)

Every email deliverable includes these sections in this order:

1. **Subject Line** — under 50 characters preferred, 60 max
2. **Preview Text** — 40-90 characters, complements but does not repeat subject
3. **Body** — formatted with line breaks for mobile readability, HTML for delivery
4. **CTA** — button text + destination URL or action
5. **Audience targeting** — sub-account + tag (e.g., `TCA / Newsletter Subscriber`)
6. **Compliance Notes** — flag any claims, disclosures needed, or areas the compliance check will scrutinize
7. **Variant** (when A/B testing is requested) — second version with stated hypothesis

For SMS:
1. **Message text** — under 160 characters per segment (multi-segment OK if intentional)
2. **CTA link** (with UTM tracking)
3. **Audience targeting**
4. **Compliance notes**

---

## Audience targeting (critical — always specify explicitly)

Every send call specifies exactly **one tag in exactly one sub-account**. The skill never sends to "All" or across sub-accounts in one call.

**Sub-accounts:**
- `TCA` — Thomas Clark Advisor sub-account (newsletter, product launches, lead magnet sequences, community)
- `CCM` — Confluence Capital Management sub-account (seminar paths A-G originate here)

**Common audience targets:**
| Target | Sub-account | Tag (verify exact name in GHL admin) |
|---|---|---|
| Weekly newsletter | TCA | Newsletter Subscriber |
| Product buyers (general) | TCA | Buyer |
| Just in Case Binder buyers | TCA | Buyer - JICB |
| Pre-Retirement Checklist buyers | TCA | Buyer - PreRet |
| Community members | TCA | Community Member |
| Lead magnet — generic | TCA | Lead - Newsletter Signup |
| Seminar Path A through G | CCM | Seminar Path A through Seminar Path G |
| Cart abandonment | TCA | Cart Abandoned |

If the requested audience tag doesn't exist in the table above, ask Thomas before proceeding. Do not invent tags.

---

## Category-specific patterns

### 1. Weekly newsletter (the primary recurring send)

**Trigger:** `weekly-newsletter-digest` routine, Sundays 7am ET (sends 8am ET to subscribers)
**Audience:** `TCA / Newsletter Subscriber`
**Cadence:** Once per week. NEVER per blog post — that creates unsubscribe pressure.

**Structure:**
1. **Subject line** — teases the lead essay or the strongest post of the week. Under 50 chars.
2. **Preview text** — 40-90 chars. Sets up the lead essay angle.
3. **Greeting** — "Hi {first_name}," or "Hey {first_name}," (warmth, not formality)
4. **Lead essay** (200-400 words) — fresh thinking from Thomas, original to the newsletter, not on the blog. This is why people open. Topics rotate free-form: market observations, planning principles, contrarian takes, personal stories, behavioral finance reflections.
5. **From the blog this week** — a header section with 3-5 post cards. Each card: post title (linked), 1-2 line summary, "Read full post →" link.
6. **One soft CTA at the end** — rotates weekly: newsletter share, related TCA product mention, community invite. NEVER an advisory consultation.
7. **Sign-off** — "— Thomas"
8. **Footer** — standard email footer (unsubscribe, address required by CAN-SPAM, brand mark)

**Compliance considerations specific to newsletter:**
- Lead essay runs through full `tca-compliance-check` (Critical + Warning + Info tiers)
- Post cards don't need re-checking (already-published-and-checked content)
- Soft CTA must not be advisory consultation under any circumstance
- No "guaranteed" anything in lead essay
- If lead essay touches a sensitive topic (annuities, Social Security strategy specifics, market predictions), apply extra scrutiny

### 2. Product launch sequences

**Trigger:** Manual or routine (TBD — tied to product launch cadence)
**Audience:** Varies by product. Typically `TCA / Newsletter Subscriber` for the general launch + `TCA / Buyer - <previous product>` for cross-sells.

**Structure for a 5-email launch sequence:**
- Email 1 — Announcement / why this product exists / who it's for. Subject: opens with curiosity, not "I built a thing."
- Email 2 — Specific use case story / pain point this solves. Hypothetical scenario.
- Email 3 — Behind-the-scenes / what's actually in it. Shows the work, builds trust.
- Email 4 — Objection handling / FAQ format. Addresses the "I'm not sure" reader.
- Email 5 — Last chance with real deadline (if launch window is real). Otherwise a value-reinforcement send.

**Compliance considerations:**
- No income claims, performance claims, or guarantees
- Products are educational/organizational, never advisory
- "This will help you organize" ✓ / "This will save you $X" ✗
- Real urgency only (actual deadline) — never manufactured

### 3. Lead magnet welcome sequences

**Trigger:** Form fill / lead magnet download via GHL workflow
**Audience:** `TCA / Lead - Newsletter Signup` or specific lead magnet tag

**Structure for a 4-email welcome sequence:**
- Email 1 — Deliver the lead magnet immediately (within minutes of signup). Subject: "Here's your [thing]"
- Email 2 (next day) — How to use what they got + what to expect from the newsletter
- Email 3 (day 3-4) — Story-led: "Here's why I built this" or "Here's what most people miss"
- Email 4 (day 7) — Soft CTA to a related product or community

After this 4-email sequence completes, the contact transitions to the regular weekly newsletter cadence. The `Newsletter Subscriber` tag should be applied at intake (not at end of sequence) so they receive newsletter sends from week 1.

### 4. Community onboarding

**Trigger:** Community signup via GHL workflow
**Audience:** `TCA / Community Member`

**Structure for a 3-email onboarding sequence:**
- Email 1 (immediate) — Welcome + how to access + first step. Specific action.
- Email 2 (day 2) — How the community works + what's coming this week
- Email 3 (day 7) — Check-in / what they've engaged with / encourage participation

After onboarding, community members continue receiving weekly newsletter (they should also have `Newsletter Subscriber` tag) plus community-specific announcements as needed.

### 5. Tripwire / cart abandonment

**Trigger:** Cart abandonment workflow in GHL
**Audience:** `TCA / Cart Abandoned`

**Structure for a 2-3 email recovery sequence:**
- Email 1 (1-2 hours after abandon) — "Just checking — anything I can answer?" tone, not pushy
- Email 2 (24 hours) — Address common objections, restate value
- Email 3 (48-72 hours, optional) — Last gentle nudge with discount if appropriate

Cart abandonment is the one place where slight urgency is acceptable — but it's natural urgency from the cart timing, not manufactured.

### 6. Seminar follow-up paths A–G

**Trigger:** Seminar attendance / engagement workflow in CCM sub-account
**Audience:** `CCM / Seminar Path A` through `CCM / Seminar Path G`

The seven paths represent different attendee engagement levels and intent signals from the seminar. Each path has its own multi-touch sequence. Specific path content lives in `~/seminar-system/` knowledge base — load it when working on seminar paths.

**General principles across all paths:**
- Educational content first, sales pitch second
- Path-appropriate cadence (high-intent paths get faster touches; low-intent paths longer windows)
- Every path eventually transitions warm contacts to the TCA newsletter via the mirror handoff
- Compliance: same Critical/Warning/Info tiers as everything else

When generating seminar path content, ask Thomas which path and which touch number in the sequence before drafting.

### 7. SMS variants

When SMS is requested for any of the above:
- Reduce dramatically — typically 100-160 chars
- Single CTA, link with UTM tracking
- More conversational than email
- Time-sensitive sends only (newsletter is not appropriate for SMS — too frequent)
- Compliance considerations: SMS opt-in must be confirmed via GHL; never SMS contacts without explicit consent
- Identify sender ("Thomas Clark here —") so recipient knows who's texting

---

## A/B testing

When a request asks for variants, provide:
- **Control** — the version you'd send by default
- **Variant** — second version with one specific change

State the hypothesis: "Variant tests whether question-led subject ('Are you claiming Social Security too early?') outperforms statement-led ('Most retirees claim Social Security too early')."

Default to one variant unless explicitly asked for more. Two variants is enough to learn from; more dilutes the test.

---

## Decision framework (when content is ambiguous)

1. **Compliance check** — Could this be read as personalized advice, performance claim, or guarantee? If yes, rewrite as educational with hypothetical framing.
2. **Tone check** — Does this sound like Thomas (calm-confident-direct)? Or did it slip into salesy/hyped? Match TCA voice.
3. **CTA check** — Is there exactly one clear CTA? Multiple CTAs dilute conversion.
4. **Length check** — Is every word earning its place? Cut filler.
5. **Audience check** — Would the primary audience (pre-retiree 55-70) understand this without Googling? If not, simplify.

---

## Compliance integration

Every email or SMS this skill produces runs through `tca-compliance-check` as a final pass before being returned to the calling routine.

The skill loads `tca-compliance-check`, passes the full email content + content_type (`email` or `sms`) + audience metadata, and acts on the returned status:

- **PASS** → return the publish-ready email package
- **FAIL_AUTOFIX_APPLIED** → use the auto-fixed copy, include the fixes in the returned compliance notes so the human reviewer sees what changed
- **FAIL_BLOCK** → stop. Return the draft + Critical findings to the calling context. Do not send.

For routine-driven sends (weekly newsletter, automated sequences), `FAIL_BLOCK` triggers a Critical Notion task and the routine does not send that day. Thomas decides next step.

---

## Hard boundaries (never violate, even if requested)

- Never send copy without the compliance check
- Never include specific investment advice, product recommendations, or personalized financial guidance
- Never use fear-based urgency or manufactured scarcity in editorial content
- Never promise returns, outcomes, or specific financial results
- Never fabricate testimonials or imply endorsement by regulators
- Never send to multiple sub-accounts in one call
- Never invent audience tags that don't exist in GHL
- Never use "Fiduciary Advisor" or any credential framing other than "Series 65 Investment Advisor Representative"
- Never use advisory consultation CTAs in newsletter or editorial content
- Never produce HTML email templates that fail mobile rendering — every email must be mobile-first

---

## Email HTML template requirements

When producing the full HTML for delivery (vs. just copy):
- Mobile-first responsive design
- Single column layout
- Body text 16px minimum, line-height 1.5
- Brand colors: navy `#1a2947` accent, warm cream `#f7f1e3` backgrounds where used, body text dark gray `#241e14`
- Plain-text version always included for accessibility and deliverability
- All links UTM-tracked: `?utm_source=newsletter&utm_medium=email&utm_campaign=<campaign-slug>`
- Preheader text in HTML (drives preview behavior in inbox)
- Footer required by CAN-SPAM: physical mailing address, unsubscribe link, brand identification

---

## What this skill never does

- Send to multiple sub-accounts in a single call
- Send the weekly newsletter more than once per week
- Trigger a per-blog-post email broadcast (the newsletter is the only TCA editorial email cadence)
- Write copy without running compliance check
- Use manufactured urgency or fear-based language
- Make promises about returns, outcomes, or financial results
- Fabricate testimonials
- Use advisory CTAs in editorial content
- Write at a reading level above 8th grade
- Default to multi-CTA email designs
- Skip plain-text version of HTML emails
- Use audience tags not verified in the GHL admin
