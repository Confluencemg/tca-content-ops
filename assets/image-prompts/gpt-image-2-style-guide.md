# GPT Image 2 Style Guide for TCA

Production-ready prompt templates for generating brand-aligned images with GPT Image 2. Used by `daily-blog-publish` and `daily-social-distribute` routines whenever they call the OpenAI image API.

This file is the single source of truth for TCA visual style. If a routine generates an image that doesn't match what's specified here, the routine has a bug — fix the prompt construction, not the output.

---

## Brand visual system (locked)

These values apply to every TCA image. Do not deviate.

### Color palette

- **Navy** — `#1a2947` — primary text, primary fields, deep accents
- **Warm cream** — `#f7f1e3` — primary background, page fields, breathing room
- **Aged brass** — `#9c7a3c` — accent, callouts, highlight elements, key data points
- **Ink brown** — `#241e14` — secondary text, fine detail, borders

When prompting GPT Image 2, name these as actual hex codes in the prompt — the model handles them. Don't say "warm cream" alone; say "warm cream `#f7f1e3`."

### Typography style

GPT Image 2 chooses fonts based on prompt direction. Use these descriptors:

- **For headlines and display text:** "elegant serif typography, similar to Fraunces or Recoleta — high contrast, slight warmth"
- **For body text:** "clean modern serif, generous letter spacing, comfortable reading rhythm"
- **For small labels and captions:** "small-caps sans-serif, like Inter All Caps with tracking, used sparingly for structural labels only"
- **NEVER:** generic fonts ("Arial," "Times New Roman"), decorative fonts, handwritten/script fonts, all-caps everywhere, novelty typography

### Mood and register

- Calm, warm, grounded.
- Editorial publication, not financial advertisement.
- Reads more like a hardcover book or longform magazine than a brochure or app.
- Warmth and texture over flat-color minimalism. Cream backgrounds, not white.
- Real spaces (kitchens, side tables, desks, sunlit rooms) over abstract corporate compositions.
- Diverse, age-appropriate subjects (ages 55+ heavily represented, but content for younger audiences uses appropriately younger subjects).
- Subjects should look like actual people, not stock-photography models.

### Always avoid

- White backgrounds (use warm cream `#f7f1e3` instead)
- Bright primary colors (no pure red, pure green, pure blue, pure yellow)
- Stock-photo gestures (suit-and-tie handshakes, men in glasses pointing at charts, falsely diverse boardrooms)
- Generic AI looks (uncanny faces, plastic skin, melted hands, weird teeth)
- Cluttered compositions
- Text overlays on images that have illegible contrast
- "Financial advisor" tropes (skyscrapers, golden retrievers on yachts, sunset beach walks)

---

## Template 1 — Blog featured image (1200×600)

The hero image at the top of every blog post. Lifestyle photography aesthetic; subject contextual to the post topic; no text overlay (the post headline appears separately).

### Prompt template

```
Photographic editorial image, 1200×600 horizontal aspect ratio.

SUBJECT: [SUBJECT_DESCRIPTION — e.g., "a couple in their late 60s reviewing financial documents at a wooden kitchen table, morning light, warm and unposed"]

SETTING: [SETTING_DESCRIPTION — e.g., "modern but lived-in kitchen with white oak cabinetry, warm cream walls, a window with soft natural light, a coffee cup and reading glasses on the table"]

LIGHTING: Soft natural daylight, warm tones, no harsh shadows. Photographed as if shot during golden hour or with a north-facing window.

COMPOSITION: Subjects positioned slightly off-center using the rule of thirds. Foreground depth — table edge, papers, or plant in soft focus. Background slightly defocused but readable.

COLOR PALETTE: Warm cream `#f7f1e3` and navy `#1a2947` dominant, accents of aged brass `#9c7a3c` (a brass lamp, a brass picture frame, warm metallic warmth somewhere in the scene), ink brown `#241e14` for grounding contrast (wooden surfaces, dark accents).

STYLE: Editorial lifestyle photography, like a Kinfolk or Cup of Jo article. Real, warm, calm. NOT corporate stock photography. NOT financial-advisor brochure aesthetic.

NO TEXT in the image.
```

### Example fills

For a Social Security claiming post:
> SUBJECT: a woman in her early 60s sitting at a small writing desk reviewing a Social Security statement, pen in hand, calm focused expression
> SETTING: a sunlit home office with a wooden desk, a cream linen chair, a brass desk lamp, a small potted plant, an open window with sheer curtains

For a retirement income floor post:
> SUBJECT: a couple in their late 60s reviewing a one-page financial summary together at a kitchen table, morning coffee, unhurried
> SETTING: a warm cream-walled kitchen with white oak cabinetry, a brass pendant light, a window with soft natural light

For a bucket planning post:
> SUBJECT: an over-the-shoulder view of hands writing in a leather-bound notebook, with a cup of coffee and a pair of reading glasses nearby
> SETTING: a quiet morning desk scene, warm cream desk surface, navy fountain pen, brass paperweight, soft natural light from the side

---

## Template 2 — Blog body image (varies — typically 800×500 or 1000×600)

In-post images that illustrate concepts, comparisons, or data. These can include simple text labels (chart axes, comparison labels), but never long copy. Use sparingly — one or two per post maximum.

### Prompt template

```
Editorial infographic image, [WIDTH]×[HEIGHT].

CONCEPT: [CONCEPT_DESCRIPTION — e.g., "a side-by-side comparison of two retirement income strategies"]

LAYOUT: [LAYOUT_DESCRIPTION — e.g., "two columns separated by a thin vertical line in aged brass `#9c7a3c`, each column with a clear header, illustrative icon or simple chart, and 2-3 short labels"]

VISUAL ELEMENTS: [VISUAL_DESCRIPTION — e.g., "simple line illustrations of stacked income sources, geometric icons for Social Security and pension, no human figures"]

COLOR PALETTE: Warm cream `#f7f1e3` background. Navy `#1a2947` for primary elements and text. Aged brass `#9c7a3c` for highlights and key data points. Ink brown `#241e14` for secondary text.

TYPOGRAPHY: Headers in elegant serif (Fraunces or Recoleta style), labels in small-caps sans-serif (Inter All Caps style), used sparingly. All text legible at thumbnail size.

STYLE: Editorial infographic, like a New York Times or Wall Street Journal explainer graphic. Clean, calm, generous whitespace, no excess decoration. NOT a corporate infographic. NOT cluttered.

ALL TEXT must be readable and contextually correct. Verify spelling carefully.
```

### Variants

- **Two-column comparison:** Left side / right side, separated by a vertical line.
- **Stacked/layered concept:** Three layers (e.g., bucket strategy — Now / Soon / Later), labeled, vertically stacked.
- **Process flow:** Three to five steps with arrows or progression markers.
- **Data callout:** A single large number or stat with explanatory caption.

### Hallucination warning

GPT Image 2 will invent specific numbers and labels if not given them. ALWAYS pass the actual data and labels in the prompt. Verify the rendered image against the prompt before publishing — if any text is wrong, regenerate.

---

## Template 3 — Instagram carousel slides (1080×1350 portrait)

Multi-slide educational carousels. Typically 8–10 slides. Visual consistency across the carousel matters more than any single slide.

### Carousel structure

- **Slide 1:** Hook slide — large headline, one image or icon, sets the topic.
- **Slides 2 through n-1:** Content slides — one idea per slide, headline + 1–3 lines + small visual element.
- **Final slide:** CTA slide — "Save this", "Read the full post — link in bio", or product CTA.

### Prompt template (per slide)

```
Instagram carousel slide, 1080×1350 portrait aspect ratio.

SLIDE TYPE: [HOOK / CONTENT / CTA]

CONTENT:
- Headline: "[EXACT_HEADLINE_TEXT]"
- Body: "[EXACT_BODY_TEXT — keep under 25 words for content slides]"
- Visual element: [VISUAL_DESCRIPTION — e.g., "small line illustration of a retirement piggy bank in aged brass on the lower right"]

LAYOUT: Generous top margin (180px). Headline in upper third, body text in middle third, visual element in lower third. Slide number indicator (e.g., "1 / 8") in small-caps in the bottom-right corner.

COLOR PALETTE: Warm cream `#f7f1e3` background. Navy `#1a2947` for headline. Ink brown `#241e14` for body text. Aged brass `#9c7a3c` for accent elements and slide number indicator.

TYPOGRAPHY: Headline in elegant serif (Fraunces or Recoleta style, weight 600). Body in clean modern serif. Slide number in small-caps sans-serif with letter-spacing.

STYLE: Editorial publication aesthetic, like a magazine spread. Calm and warm. NOT a sales graphic. NOT a meme template. Generous whitespace.

ALL TEXT must be readable at thumbnail size. Verify spelling carefully.

Brand identifier: small "TCA" wordmark in aged brass, lower-left corner, very subtle.
```

### Carousel consistency rules

- All slides in a carousel use the same color treatment.
- Slide numbers ("1 / 8", "2 / 8", etc.) must be present and correct.
- Same typeface family across the carousel.
- Visual elements escalate in interest from slide 1 (simple) to slide n-1 (richest illustration), then settle on the CTA slide.
- The hook slide and CTA slide are visually distinct from content slides — slightly larger headline, more visual weight, minimal body copy.

### Hook slide pattern

Pattern that works: a sharp, specific claim as headline. "The 76% Social Security increase most people leave on the table." Not "Social Security claiming strategies."

### CTA slide pattern

Three approved CTAs (rotate, don't repeat consecutively):
- "Save this post for later." (engagement)
- "Read the full post — link in bio." (traffic)
- "Get the free Social Security guide — link in bio." (lead magnet)
- "Follow for more retirement planning that actually makes sense." (audience)

---

## Template 4 — Instagram feed image (1080×1080 square)

Single-image feed posts. Used for quote cards, single-stat posts, or topical announcements.

### Prompt template

```
Instagram feed image, 1080×1080 square aspect ratio.

CONTENT TYPE: [QUOTE_CARD / STAT_CARD / ANNOUNCEMENT]

CONTENT:
- Primary text: "[EXACT_TEXT — keep under 30 words]"
- Attribution: "— Thomas Clark"  [if quote card]
- OR: Stat with caption [if stat card]

LAYOUT: Quote/stat in the upper two-thirds, centered or slightly left-aligned. Attribution in lower third, small-caps. Generous margins (120px each side).

COLOR PALETTE: Warm cream `#f7f1e3` background. Navy `#1a2947` for primary text. Aged brass `#9c7a3c` for accent elements (a small line above the attribution, or a bracket / quote mark).

TYPOGRAPHY: Primary text in elegant serif (Fraunces or Recoleta style), large enough to fill the upper two-thirds. Attribution in small-caps sans-serif.

STYLE: Editorial quote-card aesthetic, like a magazine pull-quote spread. Calm, warm, generous whitespace.

Brand identifier: small "TCA" wordmark in aged brass, lower-right corner, very subtle.

ALL TEXT must be readable. Verify spelling carefully.
```

### Quote card examples

Use Thomas's actual phrasing from blog posts. Strong candidates:

- "The single greatest predictor of retirement happiness isn't net worth — it's the gap between essential expenses and guaranteed income."
- "When your grocery bill doesn't depend on the S&P 500, you make better investment decisions."
- "Most retirement tax savings happen in November and December — not in April when you file."
- "Variable annuities are not how I build an income floor."
- "The right answer depends entirely on your personal situation."

### Stat card pattern

A single large number, a one-line explanation, and a source line.

> **76%**
> The lifetime Social Security increase from claiming at 70 instead of 62.
> *— SSA, 2026*

---

## Template 5 — Pinterest pin (1000×1500 vertical, 2:3 ratio)

Pinterest is the primary organic traffic driver. Pins must work as standalone clickable visuals — strong headline, clear value, branded look.

### Pin structure

- **Top third:** Strong, specific headline (the value proposition).
- **Middle third:** Sub-headline or supporting line, OR an illustrative image.
- **Bottom third:** Brand identifier, URL, or CTA cue.

### Prompt template

```
Pinterest pin, 1000×1500 vertical aspect ratio.

CONTENT:
- Headline: "[EXACT_HEADLINE — 6-12 words, specific and value-driven]"
- Sub-line: "[OPTIONAL_SUBLINE — 5-10 words]"
- URL/brand: "thomasclarkadvisor.com"

LAYOUT:
- Top section (50% of height): Headline, large serif type, navy `#1a2947` on warm cream `#f7f1e3` background. Headline takes up about 60% of vertical space in this section.
- Middle section (35% of height): Sub-line in smaller serif, OR small editorial illustration (line drawing of a relevant concept — a clock, a stack of papers, a calendar — in aged brass `#9c7a3c`).
- Bottom section (15% of height): Solid navy `#1a2947` band with "thomasclarkadvisor.com" centered in warm cream `#f7f1e3`, small-caps sans-serif.

COLOR PALETTE: Warm cream `#f7f1e3` for upper background. Navy `#1a2947` for headline text and bottom band. Aged brass `#9c7a3c` for illustration and accent elements. Ink brown `#241e14` for sub-line.

TYPOGRAPHY: Headline in elegant serif (Fraunces or Recoleta, weight 700). Sub-line in clean serif (lighter weight). URL in small-caps sans-serif with letter-spacing.

STYLE: Editorial Pinterest pin in the style of high-end publications (Cup of Jo, NYT Wirecutter, Apartment Therapy). NOT a Canva template. NOT a "5 SECRETS!!!" hype-pin. Calm, specific, valuable.

ALL TEXT must be sharp and readable. Verify spelling carefully.
```

### Pin headline patterns that work

- **Number + specificity:** "5 Social Security Mistakes That Cost Retirees Thousands"
- **Question:** "Should You Claim Social Security at 62 or 70?"
- **Contrarian:** "Why Variable Annuities Don't Belong in Most Retirement Plans"
- **Year-specific:** "What's Changing for Retirees in 2026"
- **Comparison:** "Medicare Supplement vs. Medicare Advantage: How to Choose"
- **Process:** "How to Build a Retirement Income Floor in 4 Steps"

### Pin headline patterns to avoid

- "You won't believe…"
- "The shocking truth about…"
- "What financial advisors don't want you to know…"
- "10 things every retiree MUST do…"
- All-caps headlines
- Excessive emoji
- "Click to read more!" (Pinterest pins always link — don't waste space on this)

### Multiple pins per blog post

Each blog post needs 5 pins for the Pinterest pipeline (one fires day-of, four queue for subsequent days). Vary the pin design while keeping the brand system consistent:

- **Pin 1:** Headline-driven (the post title or strongest claim).
- **Pin 2:** Stat-driven (a specific number from the post).
- **Pin 3:** Question-driven (the question the post answers).
- **Pin 4:** Quote-driven (a "Thomas's Take" pulled from the post).
- **Pin 5:** List-driven (e.g., "5 things to check before…").

This variation lets Pinterest test different headlines for the same destination URL — A/B testing for free.

---

## Common failure modes (and how to fix the prompt)

### "Too AI-looking"
- Add: "editorial photography style, real materials and textures, not synthetic"
- Add: "imperfections preserved — slight asymmetry, natural depth of field, real lighting"
- Avoid: "perfect," "pristine," "ideal"

### Text is misspelled or wrong
- Pass the exact text inside quotes in the prompt.
- Add: "ALL TEXT must be readable and exactly as specified. Verify spelling carefully."
- If still wrong: regenerate. Don't try to edit a bad output.

### Subjects look uncanny
- Add: "natural skin texture, real-looking faces, age-appropriate features"
- Avoid: "beautiful," "perfect," "young"
- Specify age explicitly: "couple in their late 60s" not "older couple"

### Colors drift from brand
- Always use hex codes, not color names alone.
- Reinforce: "Warm cream `#f7f1e3` not white. Navy `#1a2947` not black."

### Composition is cluttered
- Add: "generous whitespace, calm composition, single focal point"
- Reduce the number of elements you're asking for.

---

## Living document

When the routine produces an image that's off-brand, log a Notion task with the prompt and the image path. Update this file when patterns emerge — new common failure modes get added to the common-failures section. Color palette, typography, and brand mood are locked and don't change without an explicit Thomas decision.
