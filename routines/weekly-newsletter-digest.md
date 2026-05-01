# Routine: weekly-newsletter-digest

**Trigger:** Cron, Sundays at 7:00am Eastern (`0 7 * * 0`)
**Runtime budget:** ~25 minutes
**Output:** One newsletter email sent to `TCA / Newsletter Subscriber` audience at 8:00am ET, containing a fresh 200-400 word lead essay from Thomas plus a recap of the past 7 days of blog posts

**Why Sunday 7am:** Sends to subscribers at 8am ET on Sunday morning — coffee window, leisurely reading time, before the work week starts. Format is reflective ("here's what I was thinking about this week") rather than action-oriented. Better fit for a multi-generational audience that includes retirees with morning routines.

**Why weekly, not per-post:** A daily email broadcast for every blog post would create significant unsubscribe pressure on a 55-70 audience. One thoughtful weekly digest with fresh thinking from Thomas plus recap respects subscriber attention and builds the newsletter as a destination people actually open.

---

## Mission

Send one newsletter email per week. The email leads with original Thomas content (not on the blog), follows with a recap of the week's posts, and closes with one rotating soft CTA.

The lead essay is the reason people open. The recap is the reinforcement. The CTA is light — never advisory consultation.

---

## Step 0 — Load TCA skills (mandatory)

Before generating any content, load:
- **`tca-content-writer`** — for the lead essay (newsletter lead essay is just another content type the writer skill produces, calibrated to 200-400 words with newsletter framing)
- **`tca-email-campaigns`** — for the email assembly, audience targeting, GHL send via API, and the universal output format
- `tca-compliance-check` — both writer and email-campaigns skills load this internally; loading explicitly here for visibility in the routine log

If `tca-email-campaigns` or `tca-content-writer` fails to load: log Critical, stop the routine. Do not send unchecked content.

---

## Step-by-step

### 1. Query the past week of blog posts

Query WordPress for posts published in the past 7 days:

```
GET /wp-json/wp/v2/posts?after=<7 days ago ISO>&per_page=10&orderby=date&order=desc
```

Capture for each post: title, URL, excerpt, category, featured image URL.

If 0 posts found this week: log Medium-priority Notion task ("no posts to digest"), still send the newsletter — the lead essay alone is valuable. Just skip the recap section.

If 5+ posts found: cap recap at 5 posts. Pick the strongest 5 (judgment call: variety across categories preferred over homogeneity).

### 2. Generate the lead essay

Invoke `tca-content-writer` skill with parameters:
- `content_type: newsletter_lead_essay`
- `length_target: 200-400 words`
- `category: free-form` (skill picks based on what feels timely this week)
- `must_be_original: true` (not pulled from any existing blog post)

The writer skill returns the lead essay text plus runs its own compliance check. If the writer skill returns FAIL_BLOCK, stop and log Critical Notion task — do not send the newsletter without a clean lead essay.

### 3. Compose the subject line and preview text

Use `tca-email-campaigns` skill to compose:
- **Subject line** (under 50 chars) — teases the lead essay's strongest hook
- **Preview text** (40-90 chars) — sets up the lead essay angle, complements but does not repeat subject

Generate one A/B variant alongside the control. State the hypothesis in the Notion log (e.g., "Variant tests question-led subject vs. statement-led"). Use the control by default unless overridden.

### 4. Assemble the email HTML

Use `tca-email-campaigns` skill to build the full HTML email per the universal output format:
- Greeting: "Hi {first_name},"
- Lead essay (200-400 words)
- Section header: "From the blog this week"
- 3-5 post cards: title (linked), 1-2 line summary, "Read full post →" link
- One soft CTA at end (rotates weekly — see CTA rotation below)
- Sign-off: "— Thomas"
- Standard footer (CAN-SPAM compliant: physical address, unsubscribe, brand mark)

UTM tracking on all links: `?utm_source=newsletter&utm_medium=email&utm_campaign=weekly-<YYYY-MM-DD>`

### 5. CTA rotation (one per week)

Track which CTA was used in the previous 4 newsletters via a simple file `logs/newsletter-cta-history.md`. Rotate through:

- Week 1 — Newsletter share: "Forward this to someone who'd find it useful"
- Week 2 — Related TCA product mention (current focus product, e.g., Just in Case Binder)
- Week 3 — Community invite (link to community signup)
- Week 4 — Back to newsletter share

Never advisory consultation. Never stacked CTAs.

If a product launch is active per `content-calendar.md`, override the rotation to feature the launching product for that week.

### 6. Run final compliance check

The `tca-email-campaigns` skill internally runs `tca-compliance-check` on the assembled email. Confirm result:

- `PASS` → proceed to send
- `FAIL_AUTOFIX_APPLIED` → use auto-fixed version, capture fixes for the Notion log
- `FAIL_BLOCK` → stop. Log Critical Notion task with full email draft and findings. Do not send.

### 7. Send via GHL broadcast API

POST to GHL `/conversations/messages` (or appropriate broadcast endpoint) with:
- Sub-account: TCA
- Audience tag: `Newsletter Subscriber`
- Subject: from Step 3
- Preview text: from Step 3
- HTML body: from Step 4
- Plain-text version: stripped HTML for accessibility/deliverability
- Send time: 8:00am ET today (one hour after this routine runs)

Capture the GHL broadcast ID for the Notion log.

### 8. Update CTA rotation history

Append to `logs/newsletter-cta-history.md`:

```yaml
- date: <YYYY-MM-DD>
  cta_type: <newsletter_share | product_mention | community_invite>
  product_or_topic: <if applicable>
  variant_used: <control | A>
```

### 9. Log to Notion

Create a page in the Operations Hub:
- **Task:** `Newsletter sent — week of <YYYY-MM-DD>`
- **Project:** `TCA`
- **Status:** `Done`
- **Type:** `Marketing`
- **Owner:** `Claude`
- **Priority:** `Medium`
- **Notes:**
  - Send time
  - Audience: TCA / Newsletter Subscriber
  - GHL broadcast ID
  - Subject line used (control or variant)
  - A/B variant alternate (for reference)
  - Lead essay topic and word count
  - Post recap count + post URLs
  - CTA used this week
  - Compliance check status, any auto-fixes applied
- **Source Chat:** `weekly-newsletter-digest routine — <date>`

### 10. Send the run summary

To Slack `#tca-content-ops` (or email if not connected):

```
✅ Weekly newsletter sent — <date>

Subject: <subject line>
Audience: TCA / Newsletter Subscriber (<contact count>)
Send time: 8:00am ET

Lead essay: <topic>, <word count> words
Posts recapped: <N>
CTA: <type>
A/B variant: <if used>

Compliance: <PASS | autofix applied with details>
GHL broadcast ID: <id>
Notion log: <URL>
```

---

## Failure modes and how to handle each

| What fails | What to do |
|---|---|
| 0 posts in past 7 days | Send newsletter with lead essay only, skip recap, log Medium Notion task |
| Lead essay generation fails | Stop. Critical Notion task. Do not send. |
| Lead essay fails compliance (FAIL_BLOCK) | Stop. Critical Notion task. Do not send. |
| Email assembly fails | Stop. Critical Notion task. |
| GHL API rate limit hit | Pause 60 seconds, retry once. If still failing, log Critical Notion task — Thomas can manually broadcast via GHL UI. |
| GHL send fails | Critical Notion task with the full assembled HTML for manual send. |
| Notion logging fails | Send Slack/email summary. Email Thomas about Notion separately. |
| CTA rotation file unreadable | Default to newsletter_share for this week. Log Low-priority task. |

---

## What this routine never does

- Send more than once per week
- Send to any sub-account other than TCA
- Send to any tag other than `Newsletter Subscriber`
- Reuse a lead essay that's also been published as a blog post
- Use a CTA that drives toward advisory consultation
- Skip the compliance check
- Send if compliance returns FAIL_BLOCK
- Stack multiple CTAs in one newsletter
- Run if the previous week's send is still in progress

---

## Mirror handoff context (reference)

Most TCA Newsletter Subscriber contacts originated from CCM-side seminar workflows, were mirrored to TCA after seminar completion, and tagged as Newsletter Subscriber on the TCA side at intake.

This routine targets only the TCA side — it does not interact with CCM. The mirror handoff is upstream of this routine and handled by existing GHL workflows (or whatever Thomas's mirror automation does). If the mirror is broken, contacts won't appear in the TCA Newsletter Subscriber tag and won't receive the newsletter — that's a GHL config issue, not a routine issue.
