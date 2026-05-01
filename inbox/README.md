# Inbox — drop your pre-made assets here

This folder is where you drop images, carousels, and pins you've made yourself (in ChatGPT, Canva, Photoshop, wherever). Both routines check `inbox/` before generating any visual asset via the API. If a matching asset exists, the routine uses it. If not, it generates fresh.

The whole point: when you have time and want to make nicer-than-API assets, drop them here and the pipeline picks them up automatically over the next days/weeks.

---

## Folder structure

```
inbox/
├── blog-images/        ← featured (1200×630) and in-body (1200×800) images
├── pinterest-pins/     ← 1000×1500 (2:3) pins
├── ig-carousels/       ← carousel sets, one subfolder per set
├── ig-feed-images/     ← 1080×1350 (4:5) IG feed images
└── used/               ← consumed assets archived here automatically
```

---

## Naming convention

Each image gets a sidecar `.json` file with the **same base name** as the image:

```
inbox/pinterest-pins/
├── social-security-claiming-cost.png
├── social-security-claiming-cost.json    ← sidecar
├── bucket-planning-vs-3-percent-rule.png
└── bucket-planning-vs-3-percent-rule.json
```

For carousels, the **subfolder** is the unit. Slides named `slide-1.png` through `slide-N.png`, plus a `set.json` sidecar at the subfolder level:

```
inbox/ig-carousels/bucket-planning-101/
├── slide-1.png
├── slide-2.png
├── slide-3.png
├── ...
├── slide-9.png
└── set.json
```

---

## Sidecar JSON spec

### Pinterest pin sidecar

```json
{
  "type": "pinterest_pin",
  "topic": "social security claiming age",
  "headline": "The Real Cost of Claiming Social Security at 62",
  "category": "Social Security",
  "use_with_post_slug": null,
  "use_by_date": "2026-12-31",
  "description": "Most people claim too early and lock in a 30% lifetime reduction. Here's how the math actually works."
}
```

### Blog image sidecar

```json
{
  "type": "blog_image",
  "slot": "featured",
  "topic": "bucket planning basics",
  "category": "Bucket Planning",
  "use_with_post_slug": null,
  "use_by_date": "2026-12-31"
}
```

`slot` is one of: `featured`, `in_body_1`, `in_body_2`.

### IG feed image sidecar

```json
{
  "type": "ig_feed_image",
  "topic": "fixed index annuities for income",
  "category": "Retirement Planning",
  "use_with_post_slug": null,
  "use_by_date": "2026-12-31"
}
```

### IG carousel set sidecar (`set.json` at subfolder level)

```json
{
  "type": "ig_carousel",
  "topic": "bucket planning basics",
  "category": "Bucket Planning",
  "use_with_post_slug": null,
  "slide_count": 9,
  "caption_draft": "Most people get retirement planning backwards. They optimize for growth when they should be optimizing for income. Here's the bucket framework that actually works.\n\nSwipe through →",
  "use_by_date": "2026-12-31"
}
```

`caption_draft` is optional. If present, the routine uses it. If null, the routine generates a caption.

---

## Field reference

| Field | Required | Notes |
|---|---|---|
| `type` | Yes | One of: `pinterest_pin`, `blog_image`, `ig_feed_image`, `ig_carousel` |
| `topic` | Yes | Free-form topic description for matching. Be specific. |
| `category` | Yes | One of TCA's six categories: `Market Insights`, `Social Security`, `Retirement Planning`, `Bucket Planning`, `Financial Education`, `Trading & Markets` |
| `use_with_post_slug` | No | If set, asset only used for this specific post. If `null`, used whenever topic/category matches. |
| `use_by_date` | No | If set and date has passed, asset moves to `inbox/expired/`. Optional but recommended for time-sensitive content. |
| `headline` | Pinterest only | The text overlay headline on the pin. |
| `description` | Pinterest only | Pin description (200-500 chars, Pinterest SEO). |
| `slot` | Blog images only | `featured`, `in_body_1`, or `in_body_2` |
| `slide_count` | Carousels only | Number of slides in the set. |
| `caption_draft` | Carousels only | Optional pre-written caption. |

---

## Two ways to use the inbox

### Option A — Topic-matched (recommended for most cases)

You make 5 nice Pinterest pins about Social Security claiming when you have time. You drop them in `inbox/pinterest-pins/` with `use_with_post_slug: null`. Over the next few weeks, when the routine writes Social Security posts, it pulls those pins as needed.

### Option B — Bound to a specific post

You know Tuesday's post is about bucket planning vs. the 3% rule, and you want a specific carousel for it. You drop the carousel into `inbox/ig-carousels/bucket-vs-3-percent/` with `use_with_post_slug: "why-bucket-planning-beats-the-3-percent-rule"`. The routine grabs it only for that specific post.

---

## What happens after the routine uses an asset

1. The file (and sidecar) move from `inbox/<type>/` to `inbox/used/<YYYY-MM>/`
2. The sidecar gets a new field: `used_in: <published URL or scheduler reference>`
3. You have a permanent record in `inbox/used/` of what asset landed where

If you ever need to re-use an asset, copy it back into the active inbox folder.

---

## Quick examples to copy

**Drop a Pinterest pin about Social Security:**

1. Save your pin as `inbox/pinterest-pins/ss-spousal-strategy.png` (1000×1500)
2. Create `inbox/pinterest-pins/ss-spousal-strategy.json`:

```json
{
  "type": "pinterest_pin",
  "topic": "social security spousal benefits",
  "headline": "How Spousal Benefits Work (And When to Use Them)",
  "category": "Social Security",
  "use_with_post_slug": null,
  "description": "Spousal benefits can be worth $500-1500/month. Here's exactly how they work and the one timing rule that matters most."
}
```

That's it. Next time the routine writes a Social Security post, your pin gets used.

**Drop a 9-slide carousel for a specific upcoming post:**

1. Create folder `inbox/ig-carousels/three-buckets-explained/`
2. Save 9 slides as `slide-1.png` through `slide-9.png` (1080×1350 each)
3. Create `inbox/ig-carousels/three-buckets-explained/set.json`:

```json
{
  "type": "ig_carousel",
  "topic": "bucket planning Now Soon Later framework",
  "category": "Bucket Planning",
  "use_with_post_slug": "the-three-bucket-framework-explained",
  "slide_count": 9
}
```

When the routine generates that specific post, it uses your carousel instead of generating one.
