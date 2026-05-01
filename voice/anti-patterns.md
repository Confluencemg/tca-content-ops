# TCA Anti-Patterns

What to avoid when writing as Thomas Clark. If a draft contains any of these, fix it before the compliance pass.

This file is the negative companion to `tca-voice-examples.md`. Voice examples show what to do; this file shows what kills the voice.

---

## Generic financial-advisor mush

The biggest single failure mode. AI-generated financial content drifts toward bland, hedged, brochure-style prose. Every one of these phrases is a flag:

- "In today's complex financial landscape"
- "Navigating the world of [retirement/investing/Social Security]"
- "It's never too late / never too early to start"
- "A holistic approach to your finances"
- "Empower yourself to make informed decisions"
- "Take control of your financial future"
- "Whether you're a beginner or a seasoned investor"
- "Many people struggle with…"
- "Financial planning can be overwhelming"
- "We all know that saving is important"
- "At the end of the day, the choice is yours"

If the draft contains any of these, rewrite the surrounding paragraph. They don't fix in isolation — they're symptoms of a paragraph that's not saying anything specific.

---

## Hedging where Thomas has a position

Thomas has clear views on variable annuities, income floors, the bucket strategy, and Social Security claiming. Drafts that soften those positions into "some advisors prefer" or "it depends on your situation" — when Thomas would actually take a stand — read as inauthentic.

**Bad:** "Variable annuities can be a useful tool in some retirement plans, depending on your circumstances."

**Good:** "Variable annuities are not part of how I build an income floor. They combine market risk with high fees, defeating the purpose of guaranteed income security heading into retirement."

The pattern: when Thomas has a view, the draft states it. The "your situation may vary" qualifier goes at the *end* of the explanation, not the start.

Exception: legitimate "it depends" cases — pension lump sum vs. monthly, when to claim Social Security, Roth conversion timing — should be presented as genuine tradeoffs with the actual variables that determine the answer. Don't fake-hedge ("there's no one-size-fits-all answer") to avoid taking positions Thomas would actually take.

---

## Hype, urgency, and curiosity-bait

The TCA register is calm. Anything that screams violates the brand. Examples that fail:

- "You won't BELIEVE what's happening with Social Security"
- "The retirement mistake that could cost you EVERYTHING"
- "Most advisors don't want you to know this…"
- "Time is running out!"
- "Act now before it's too late"
- "Discover the secret to…"
- All caps for emphasis (use bold sparingly instead)
- Multiple exclamation points
- "🚨" or alarm-bell emoji

Subject lines, headlines, and social hooks should be specific and observational. "The 76% Social Security increase most people leave on the table" is calm and specific. "SHOCKING Social Security trick they don't tell you!" is the wrong brand.

---

## Lead funnels for advisory services

This is a compliance issue and a brand issue. TCA editorial publishes under Confluence Media Group LLC as a standalone publishing entity. The products are not lead magnets for the advisory practice.

Forbidden CTAs in editorial content:

- "Schedule a free consultation"
- "Book a call to review your plan"
- "Let's talk about your specific situation"
- "Reach out for personalized advice"
- "I help clients like you with…"
- Any CTA driving to Calendly or to Confluence Capital Management

Allowed CTAs in editorial content:

- "Download the free Social Security guide"
- "Get the Just in Case Binder"
- "Subscribe to the weekly newsletter"
- Internal blog cross-links
- "Read the related post on [topic]"

If the draft tries to convert a reader into an advisory prospect, rewrite the CTA to point at a publishing product instead. The compliance skill catches this, but it should never get there in the first place.

---

## Performance claims and predictions

These are SEC Marketing Rule trip-wires. Avoid:

- Specific dollar projections framed as expectations ("you'll earn $X")
- Historical returns implying future returns
- "This strategy returned X% over the past Y years" without disclosure
- "Most clients see…" or any aggregate claim about Thomas's actual clients
- Predictions about market direction, interest rates, or specific securities
- Anything that sounds like "buy this / sell that"

Hypothetical scenarios are fine and encouraged — they must be labeled as hypothetical and use illustrative numbers, not back-tested or claimed actual performance.

---

## Testimonials and case studies of real clients

Do not write content that references real client outcomes, even anonymized. The line between "client story" and "testimonial" under the SEC Marketing Rule is thin and not worth crossing for editorial publishing.

If the post needs an example, build a clearly-labeled hypothetical with named characters (Richard and Diane, Patricia, Margaret) and synthetic numbers. The compliance skill flags any draft that sounds like a real client.

---

## Fake authority and credential creep

Thomas's credential is "Series 65 Investment Advisor Representative." That's the line that runs in author bios and back matter. Avoid:

- "Fiduciary Advisor" (the homepage uses this — it's being phased out)
- "Wealth Manager" without qualification
- "Financial Planner" (Thomas is not a CFP)
- "Retirement Specialist" — vague title with no regulatory meaning
- Anything implying Thomas is "one of the top advisors in [region]"
- Comparisons to other advisors framed as "unlike most advisors"

The author bio block in `assets/author-bio.md` is the canonical credential statement. Don't drift from it.

---

## Lists where prose belongs

This is the AI tell that breaks the voice fastest. When given a topic, models reach for bullet lists. Thomas writes in paragraphs.

**Bad** (AI default):
> Here are 5 reasons to delay Social Security:
> - Higher monthly benefit
> - Larger COLA base
> - Better survivor benefit
> - More predictable income
> - Less reliance on portfolio

**Good** (TCA voice):
> Delaying Social Security is the single most powerful lever most retirees have, and it works through several reinforcing mechanisms. The monthly benefit grows by 6–8% per year of delay between 62 and 70 — a 76% lifetime increase if you go all the way. Every future cost-of-living adjustment then compounds on that larger base. The survivor benefit for a spouse is also tied to the higher number, which means delaying protects the household income for both lives, not just the claimant's.

Lists are appropriate for: Key Takeaways blocks, comparison tables, FAQ sections, step-by-step instructions where order matters. They are not appropriate for: explaining why something is true.

---

## Em-dash overuse

The voice does use em-dashes — they're part of the rhythm. But they're not a substitute for periods, and a paragraph with three em-dashes is doing something wrong. If the draft has more than two em-dashes per paragraph, rewrite for varied punctuation.

---

## "It's important to" / "You should know that"

These phrases announce that important content is coming, instead of just delivering the content. Cut them.

**Bad:** "It's important to understand that your income floor is the most important number in your retirement plan."

**Good:** "Your income floor is the most important number in your retirement plan."

---

## Restating the question in the answer

FAQ sections especially fall into this. The question already shows the topic; the answer doesn't need to repeat it.

**Bad:**
> **Q: How does inflation affect my income floor?**
> A: Inflation can affect your income floor in several ways. The way that inflation affects your income floor is…

**Good:**
> **Q: How does inflation affect my income floor?**
> A: Social Security has automatic COLA adjustments. Most pensions and basic annuities do not. Over a 25-year retirement, even 3% inflation cuts your purchasing power roughly in half.

---

## Closing with "I hope this helped" or "Feel free to reach out"

The TCA closing pattern is a synthesizing paragraph that pulls the post together, not a service-letter sign-off. The author bio at the bottom of the post handles attribution and contact context. The body of the post should end on the idea, not on Thomas.

---

## Padding that signals AI

These are the "this is AI" tells that show up most often:

- Repeating the same idea in three different ways within one section
- Closing every paragraph with a forward-pointing "Let's explore this further" sentence
- "What this means for you" sub-sections that restate the prior section
- Numbered lists where the numbers don't matter
- "Pros and cons" framing for things that aren't actually balanced (e.g., variable annuities don't get a "pros" column in TCA content)
- Generic transitions: "With that in mind", "That being said", "Moving forward"

If the draft uses three or more of these patterns, the whole structure needs work, not line edits.

---

## Living document

When the routines or skills produce drafts that hit any of these failure modes, log a Notion task and update this file. Anti-patterns get added; the file should grow over time as new failure modes emerge.
