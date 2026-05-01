---
name: tca-compliance-check
description: Use this skill BEFORE publishing any TCA content (blog posts, email/SMS campaigns, social posts touching financial topics, community content, video scripts, lead magnet copy). Applies SEC Marketing Rule-aligned standards as defensive best practice for content authored by anyone with a Series 65 IAR credential, even though TCA content publishes as standalone editorial under Confluence Media Group LLC. Returns pass/fail with specific findings, severity levels, and either auto-fix instructions or stop-and-log directives. Trigger this skill any time content is about to be sent, scheduled, or published — including partial requests like "review this draft" or "is this compliant" or whenever any other TCA content skill (tca-content-writer, tca-email-campaigns, etc.) reaches its pre-publish gate.
---

# TCA Compliance Check Skill

## Core philosophy

Confluence Media Group LLC publishes content authored by Thomas Clark, who happens to hold a Series 65 IAR credential. The publishing posture is that this is **standalone editorial content** — not RIA marketing — analogous to a physician-author writing books about health.

**Despite that posture, this skill applies SEC Marketing Rule-aligned standards as defensive best practice.** Reasoning: if regulators ever decided to treat the publishing as advertising-by-affiliation, the content needs to hold up under that scrutiny. Belt and suspenders. The legal theory of standalone publishing is preserved; the content quality bar is set as if Marketing Rule applied.

**This skill never frames itself as Marketing Rule enforcement.** When findings reference standards, the framing is "best-practice standards aligned with SEC Marketing Rule principles" — not "Marketing Rule violation." The distinction matters for legal posture.

---

## When to trigger this skill

Mandatory triggers — any content skill or routine MUST load and run this skill before publishing:
- `tca-content-writer` (blog posts) — final pass before WordPress submission
- `tca-email-campaigns` (email/SMS) — final pass before GHL queue
- `tca-social-media` (financial-topic social posts) — final pass before scheduling
- `tca-community-engagement` (paid community posts touching financial topics) — final pass

Optional triggers — proactively recommend when:
- User pastes content and asks "is this OK?" or "review this"
- User asks about disclaimers, disclosures, or what they can/can't say
- User mentions an SEC inquiry, complaint, or regulatory matter

---

## What this skill checks

The skill applies a tiered check. Critical findings block publication. Warnings get auto-fix attempts. Info findings are logged but don't block.

### Tier 1 — Critical (block publication)

If any Critical finding is present, the content does not publish. The calling routine logs a Critical Notion task and stops.

**1. Specific investment recommendations**
Content must not say "you should invest in X," "buy/sell Y," or recommend specific securities, funds, ETFs, or strategies for the reader. Educational discussion of categories ("indexed annuities work well for income," "broadly diversified ETFs reduce single-stock risk") is acceptable. Specific recommendations are not.

**2. Performance claims, guarantees, or projected returns**
No "you'll earn X%," "this strategy returns Y%," "guaranteed Z% growth," or "you will have $N at retirement." Acceptable: ranges with hypothetical labels, illustrations clearly marked as hypothetical, language like "could," "may," "in some cases," "historically has averaged."

Exception: legitimately guaranteed instruments (Social Security benefit amounts based on actual claiming math, pension calculations based on stated formulas, FIA income rider guarantees as defined by contract). These can be discussed as guaranteed because they are.

**3. Personalized advice**
Content addresses readers in general terms. No "in your situation, you should..." or "for someone with your risk tolerance..." Acceptable: "if you're in this situation, here are the considerations" — that's educational, not personalized.

**4. Real client stories or testimonials without proper disclosure**
Every example must be hypothetical with fictional names. "Consider a hypothetical 62-year-old named Sarah" is fine. "My client Sarah" is not. Testimonials in any context require SEC Marketing Rule-compliant disclosure language; default is to not include them in publishing content at all.

**5. Advisory CTAs**
No "schedule a consultation," "book a call with our advisor team," "let's review your portfolio," or any CTA that drives readers toward an advisory relationship with Thomas, CCM, or Altitude. Acceptable CTAs: read related blog post, sign up for newsletter, get the free guide, buy a product, join the community.

**6. "Fiduciary Advisor" credential framing**
Author bio and any credential reference uses "Series 65 Investment Advisor Representative" — never "Fiduciary Advisor," "Wealth Manager," "Financial Planner," or any title that implies an advisory relationship with the reader.

**7. Missing required disclaimer**
Every blog post must include the standard disclaimer block from `assets/disclaimers/standard-blog-footer.md`. Verbatim, no abbreviations. Email and social may use shortened versions per their respective skill specs.

**8. Fear-mongering or urgency manipulation**
No "act NOW or you'll lose everything," catastrophizing language, or manufactured scarcity. Inform with data, not anxiety. Critical because it's both a Marketing Rule integrity issue AND a brand-quality issue — Thomas's voice is calm-confident, never panicked.

### Tier 2 — Warning (attempt auto-fix, log if can't)

If a Warning finding is present, attempt to auto-fix in place. If the fix would change meaning, log a High-priority Notion task and let the routine decide whether to publish without the fix or skip the day.

**1. Hedging or vague claims that could be read as guarantees**
"This will probably work" → "This approach works in many cases." "You'll likely see results" → "Many people see results." Strengthen the qualifier.

**2. Performance comparisons without balance**
If the content compares two options (annuities vs. mutual funds, claiming early vs. delaying), both sides must include risks/limitations. If one side is presented favorably without its tradeoffs, flag and suggest balancing language.

**3. Unsubstantiated material claims**
Claims like "most retirees regret claiming early" or "studies show bucket planning outperforms" need a citation or a softening rewrite. Suggested fix: either add a citation to an authoritative source (SSA.gov, IRS.gov, peer-reviewed study, reputable industry publication) or rewrite as a non-statistical observation ("many retirees I've worked with have shared regret about claiming early").

**4. Variable Annuity language drift**
Per Thomas's stated philosophy, Variable Annuities are counterproductive for retirement income planning. Content should not present VAs neutrally or favorably. Acceptable: explaining what they are factually while noting why Thomas's bucket planning approach doesn't use them. Not acceptable: presenting VAs as a viable option for the Soon bucket.

**5. Indexed Annuity framing drift**
Per Thomas's philosophy, FIAs are for INCOME, not growth. Content should not position FIAs as growth vehicles or yield-chasing instruments. Acceptable: positioning FIAs as Soon-bucket tools for guaranteed income with optional indexed growth as a secondary feature. Not acceptable: presenting FIAs as growth alternatives to equities.

**6. Missing internal links**
Blog posts should include 3+ internal links to existing TCA content. Email/social may have fewer per skill spec. If missing, the skill suggests specific internal links from the WordPress posts API based on category match.

**7. Missing external authoritative sources**
Any factual claim about Social Security, IRS rules, Medicare, or market data should cite an authoritative external source. Auto-fix: suggest the appropriate source URL (SSA.gov, IRS.gov, Medicare.gov, CMS.gov, SEC.gov, FINRA.org).

### Tier 3 — Info (log only, do not block or fix)

**1. Disclaimer placement**
Disclaimer should appear at the end of the post, not mid-content. If found mid-content, log Info finding with placement suggestion.

**2. Readability concerns**
Walls of text, sentences over 30 words, paragraphs over 4 sentences. Log Info finding with specific locations.

**3. Consistency drift**
If the same term is used inconsistently (e.g., "Fixed Index Annuity" in one paragraph, "FIA" in another, "indexed annuity" in a third), log Info finding suggesting standardization. Not blocking — just hygiene.

---

## Output format

The skill returns a structured result the calling routine can act on:

```yaml
status: PASS | FAIL_AUTOFIX_APPLIED | FAIL_BLOCK
checked_at: <timestamp>
content_type: blog_post | email | social_facebook | social_linkedin | social_instagram | community_post | other
critical_findings:
  - rule: "<rule number and short name>"
    location: "<paragraph number, sentence excerpt, or 'global'>"
    excerpt: "<offending text>"
    explanation: "<why this fails>"
    fix_suggestion: "<what to do — but skill does NOT auto-fix Critical findings>"
warning_findings:
  - rule: "<rule number and short name>"
    location: "<location>"
    excerpt: "<offending text>"
    explanation: "<why this is a concern>"
    fix_applied: true | false
    fix_text: "<what was changed if fix_applied=true>"
    requires_human_review: true | false
info_findings:
  - rule: "<rule number and short name>"
    location: "<location>"
    note: "<observation>"
notion_log_payload:
  task: "Compliance check: <content type> — <date>"
  project: TCA
  type: Compliance
  priority: Critical | High | Medium | Low
  notes: "<full structured findings for the log>"
```

### Status interpretation

- **PASS** — no Critical findings, all Warnings either auto-fixed or determined not to require human review. Content can publish.
- **FAIL_AUTOFIX_APPLIED** — no Critical findings, but Warnings required fixes that materially changed the content. Routine should publish but log the fixes for Thomas to review.
- **FAIL_BLOCK** — Critical findings present. Content does not publish. Routine logs a Critical Notion task with the full draft and findings. Thomas decides next step manually.

---

## How calling routines should invoke this skill

The compliance skill is invoked at a specific point in the calling routine's pipeline — after content is fully drafted but before any publishing action. The calling routine passes:

1. The full content as it would publish (HTML or markdown, whichever the publishing platform expects)
2. The content type (blog_post, email, social_facebook, etc.)
3. Any context the skill needs to apply specific rules (e.g., for blog posts, the post slug and category)

The skill returns the structured output above. The calling routine acts on `status`:

- `PASS` → proceed to publish
- `FAIL_AUTOFIX_APPLIED` → use the auto-fixed content for publishing, include fixes in the daily summary so Thomas sees what changed
- `FAIL_BLOCK` → do not publish, write the Critical Notion task with full draft + findings, send alert in daily summary

---

## Compliance log pattern tracking

This skill writes to a Notion view called "Compliance Findings" within the Operations Hub database (filter: `Type: Compliance`). Over time, this log surfaces patterns the routine spec doesn't capture in a single run, like:

- Specific content types that repeatedly trigger the same Warning (e.g., "every Social Security post triggers the spousal benefits guarantee warning")
- Recurring auto-fixes that suggest a skill instruction is unclear (e.g., "Variable Annuity neutrality drift triggers in 4 of 5 retirement planning posts" → suggests the writer skill needs sharper VA language)
- Frequency of Critical findings (which should be near-zero in steady state — non-zero suggests writer skill needs tightening)

**At the start of each compliance run, the skill should:**
1. Query Notion for the last 14 days of compliance findings on the same content type
2. If a recurring pattern exists (3+ same Warning in 14 days), apply extra scrutiny to that area in the current check
3. If 3+ Critical findings on the same rule in 14 days, escalate to a Notion task for Thomas: "Recurring compliance pattern — review writer skill for [rule]"

---

## What this skill never does

- Modify Critical findings (those require human decision)
- Approve content with any Critical finding present
- Frame findings as "Marketing Rule violations" (the framing is "best-practice standards alignment")
- Frame Charles Johnson at Altitude as a required gate (Thomas makes his own compliance calls)
- Skip checks because content seems "obviously fine" (every piece gets the same check)
- Approve content with missing required disclaimers under any circumstance
- Approve fear-mongering or urgency-manipulation language even if it improves engagement
- Suggest Variable Annuities as a viable retirement income vehicle in any context
- Apply different standards to different audience segments (younger investors get the same scrutiny as pre-retirees)

---

## Standard disclaimer block (canonical text)

The skill verifies blog posts include this exact text in the footer. Located at `assets/disclaimers/standard-blog-footer.md` in the content ops repo:

```
Thomas Clark is the founder of Confluence Media Group LLC and a Series 65 Investment Advisor Representative. The information presented in this article is for educational and informational purposes only and does not constitute personalized investment, tax, legal, or financial advice. Examples and scenarios discussed are hypothetical unless specifically labeled otherwise. Past performance is not indicative of future results. Individual circumstances vary; readers should consult with qualified professionals before making decisions related to their personal financial situation.
```

If the post is missing this exact text (or a verbatim equivalent), it's a Critical finding.

---

## Reference: Marketing Rule principles applied as best practice

For internal reference (not for inclusion in any output), the eight Critical checks above map to SEC Marketing Rule principles:

1. Specific recommendations → Rule 206(4)-1(a)(1) anti-fraud baseline
2. Performance claims → Rule 206(4)-1(d) performance presentation requirements
3. Personalized advice → Rule 206(4)-1(a)(1) general prohibitions
4. Testimonials → Rule 206(4)-1(b) testimonials and endorsements requirements
5. Advisory CTAs → general anti-fraud principles + advertising integrity
6. Credential misrepresentation → Rule 206(4)-1(a)(2) misleading implications
7. Missing disclaimers → Rule 206(4)-1(a)(5) material fact disclosure
8. Fear-mongering → Rule 206(4)-1(a)(1) general prohibitions on misleading content

Again: the skill applies these as best-practice standards. Content publishes as standalone editorial. The framing of why the standards apply is publishing integrity, not RIA advertising compliance.
