# TCA Content Calendar

Topic rotation logic for `daily-blog-publish` routine. Defines what gets published when, across the six TCA content pillars.

The blog publishes 5 days a week (Monday–Friday). The six pillars rotate so each gets covered, with weight given to the highest-performing categories.

---

## Six pillars

| Pillar | Weight | Typical day |
|---|---|---|
| Social Security | High (1.5x) | Tuesday + 1 floating |
| Retirement Planning | High (1.5x) | Wednesday |
| Bucket Planning | Medium | Thursday or floating |
| Market Insights | Medium | Monday or Friday |
| Financial Education | Medium | Floating |
| Trading & Markets | Low (deferred) | Once monthly until traffic builds |

The weights reflect current audience composition and what the blog is actually known for. Adjust this table as analytics come in. Any change to weights is a Thomas decision, not a routine decision — the routine reads this file, doesn't write to it.

---

## Weekly default rotation

This is the baseline rotation when there's no overriding seasonal or news driver:

- **Monday** — Market Insights (weekend recap, week-ahead framing, or general market commentary)
- **Tuesday** — Social Security (claiming, spousal, survivor, COLA, earnings test, taxation)
- **Wednesday** — Retirement Planning (income coordination, withdrawal strategy, account ordering, RMDs)
- **Thursday** — Bucket Planning OR Financial Education (alternate weeks)
- **Friday** — Floating slot — whichever pillar is underweight that month, or topical news driver

Trading & Markets gets a once-monthly slot until the trader audience builds. Default first Friday of the month if no other Friday topic is queued.

---

## Topic queue logic

When the routine picks a topic, the precedence is:

1. **Time-sensitive:** seasonal, regulatory, or news-driven topics with a window. (Year-end tax moves in November/December. RMD reminders in late Q4. COLA announcements in October. Medicare open enrollment October 15–December 7. Tax-filing deadline content in March/April.)
2. **Inbox-flagged:** if Thomas has dropped a `topic-priority.md` note in `inbox/`, that runs next.
3. **Pillar gap:** if a pillar hasn't been covered in the last 10 publishing days, it moves to top of queue.
4. **Default rotation:** the day-of-week pillar.
5. **Topic queue file** (below): pull from the next unqueued evergreen topic in that pillar.

---

## Evergreen topic queue by pillar

This is the rolling list. The routine consumes from the top, marks consumed, and Thomas adds to the bottom as new ideas come in. Thomas can also reorder by editing this section directly.

### Social Security (queue)

1. How earnings affect Social Security if you keep working part-time after claiming
2. The widow's Social Security trap — survivor benefit timing for surviving spouses
3. Social Security and divorce — claiming on an ex-spouse's record
4. WEP and GPO — what changed and what still applies
5. The Social Security restricted application — who can still use it
6. How a Roth conversion affects your Social Security taxation
7. Social Security disability to retirement transition — what changes at FRA
8. The 35-year earnings calculation — can extra working years still help your benefit
9. Social Security suspension and re-claiming — when this still makes sense
10. Family maximum benefit — when household claiming hits the cap

### Retirement Planning (queue)

1. The retirement paycheck — how to actually structure withdrawals across accounts
2. Account ordering in retirement — which accounts to draw down first and why
3. The four-percent rule — what it actually says, what it doesn't, and what to use instead
4. Healthcare costs from 60 to 65 — bridging the pre-Medicare gap
5. Long-term care planning without long-term care insurance
6. The retirement bucket reset — when to refill cash buckets and when not to
7. Required Minimum Distributions — strategy, timing, and the most common mistakes
8. Roth conversion windows — the years between retirement and Social Security
9. Sequence of returns risk — what it means and the three ways to defend against it
10. Inflation in retirement — what actually erodes purchasing power and what doesn't

### Bucket Planning (queue)

1. Sizing the Soon bucket — how to think about your income floor target
2. The Later bucket and risk tolerance — getting the growth allocation right
3. Refilling the Now bucket — quarterly, annually, or when triggered?
4. Why the bucket strategy beats the systematic withdrawal approach for most retirees
5. Bucket planning with a pension — how the math changes
6. Bucket planning without a pension — building the income floor from scratch
7. Bucket planning and Social Security delay — the bridge years problem
8. Tax-aware bucketing — putting the right account types in the right buckets
9. Bucket planning vs. the three-fund portfolio — why both can be right

#### Consumed
- Now / Soon / Later — the full framework explained for the first-time reader — published 2026-05-03 (post 749, foundational pillar piece, 14+ day Bucket Planning gap filler)

### Market Insights (queue)

These are partly evergreen, partly week-of-publication. Routine should check the date and pick whichever fits.

1. What a 10% market correction actually means for retirees
2. Why the headline economic numbers don't match what retirees see at the grocery store
3. Reading the bond market — what yield curve shape signals for retirement portfolios
4. Election-year volatility — what 80 years of data actually shows
5. Why "this time is different" usually isn't — and when it actually is
6. The Fed's dual mandate — what each rate decision means for retirement income
7. Sector rotation in retirement portfolios — does it make sense?
8. Quarterly earnings season — why retirees can mostly ignore it
9. Geopolitical events and your portfolio — when to pay attention and when not to
10. The retirement red zone — five years before and after retirement

### Financial Education (queue)

1. The difference between an IRA and a 401(k) — explained without the jargon
2. Compound interest — the math, the metaphor, and the misconception
3. What an expense ratio actually costs you over 30 years
4. Asset location vs. asset allocation — why both matter
5. The emergency fund debate — three months, six months, or twelve?
6. Tax-loss harvesting — what it does, when it helps, when it doesn't
7. The HSA as a retirement account — the triple tax advantage explained
8. Paying off the mortgage in retirement — the actual math behind the decision
9. Estate basics — wills, beneficiary designations, and the hierarchy of who gets what

#### Consumed
- Why your beneficiary designations are more important than your will — published 2026-05-02 (post 739, Mother's Day seasonal driver)

### Trading & Markets (queue — deferred, monthly cadence)

1. What technical analysis can and cannot do
2. Position sizing — the most underrated skill in trading
3. Why most traders lose money — the three structural reasons
4. The trading journal — what to track and why most traders don't
5. Risk per trade — the math behind the 1% rule
6. When trading and retirement planning should never mix
7. The difference between trading and investing, in plain language
8. Reading a chart — the three things that actually matter
9. Earnings, news, and event-driven trading — when the edge exists
10. Building a trading process you can actually stick to

---

## Seasonal calendar

These topics override the default rotation when their window is open. Routine should check this section first.

### January
- Tax-year changeover — new contribution limits, IRMAA brackets, COLA adjustments
- "What's new for retirees in [year]" overview post

### February
- Tax planning for retirees — what to gather before April
- Roth conversion window planning for the year

### March
- Tax-filing reminders specific to retirees (RMDs, QCDs, basis)
- The March tax deadline pre-filing checklist

### April
- Q1 review for retirement portfolios
- April 15 tax post-mortem — what to do differently next year

### May
- Mother's Day / family financial planning angle (legacy binder, beneficiary review)
- Spring market commentary

### June
- Mid-year retirement plan check-in
- Father's Day / generational wealth conversation

### July
- Mid-year tax planning preview
- Q2 market recap

### August
- Back-to-school / 529 educational angle (for grandparent gifting)
- Late summer market recap

### September
- Pre-Medicare-open-enrollment prep
- Q3 review

### October
- Social Security COLA announcement (mid-October — write quickly when announced)
- Medicare open enrollment kickoff (October 15)
- Year-end tax planning preview

### November
- Year-end tax moves series (peak topic)
- Open enrollment closing reminders
- Charitable giving / QCD strategy posts

### December
- Year-end RMD reminders
- Year-end tax move final calls
- New-year preview content
- "What's changing for retirees in [next year]"

---

## How the routine uses this file

The `daily-blog-publish` routine reads this file and:

1. Checks today's date against the **Seasonal calendar**. If a window is open, prioritize.
2. Checks `inbox/topic-priority.md` for Thomas overrides.
3. Checks the last 10 published posts (in `logs/`) for pillar gaps.
4. Falls back to **Weekly default rotation** by day-of-week.
5. Pulls the top unconsumed topic from the **Evergreen topic queue** for that pillar.
6. Logs the chosen topic, marks it consumed in this file, and proceeds.

When a topic is consumed, the routine moves it from the queue to a `# Consumed` section at the bottom of that pillar's queue, with the publish date appended. Don't delete consumed topics — the historical record is useful for spotting which topics performed and which didn't.

---

## Living document

Thomas adds new topic ideas to the bottom of any queue at any time. Reordering a queue is a Thomas decision, made by editing this file. The routine never reorders the queue on its own.

When a pillar's queue drops below five unconsumed topics, the routine logs a Medium-priority Notion task asking Thomas to refill the queue. That's the early warning that the calendar is running thin.
