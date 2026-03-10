# Round: Image Generation

Goal: Configure image generation for articles — visual style, color palette, placement rules, and output settings.

---

## Phase 1: Check existing state

Read `.content-ops/config.md`. Check if an `image_generation` section exists.

Also check if `docs/image-style-guide.md` exists on disk.

**If both already exist:**

Read the image style guide. Summarize it in 3–5 bullets:

- Provider
- Visual style
- Color palette (if set)
- Placement rule
- Any skip conditions

Ask:

```text
I found an existing image style guide at docs/image-style-guide.md:

  • [bullet 1]
  • [bullet 2]
  • [bullet 3]

Want to:
  A — Keep it as-is (skip this round)
  B — Update specific sections
  C — Replace it entirely
```

- If **A**: stop and guide to the next incomplete round.
- If **B**: ask which sections to update; run only the relevant questions from Phase 2.
- If **C**: continue to Phase 2.

**If not yet configured:** continue to Phase 2.

---

## Phase 2: Image style interview

Ask all questions in a **single AskUserQuestion call** with multiple questions.

### Q1: Provider

```text
Which image generation provider should the plugin use?
  A — Google Gemini / Nano Banana (Recommended — single SDK for text+image, $0.04/image, supports brand style reference)
  B — OpenAI GPT Image (highest quality benchmarks, $0.04/image)
  C — I'll configure it manually — skip this question
```

Store answer. If C, note `provider: manual` — the user will set the API details themselves in config.

### Q2: Visual style

```text
What visual style should generated images follow?

  A — Flat illustration (clean shapes, modern, great for tech and product blogs)
  B — Photorealistic (realistic, stock-photo-like scenes)
  C — Minimalist diagram (clean lines, technical clarity, neutral tones)
  D — Watercolor / editorial (soft, artistic, magazine feel)
  E — Describe your own style
```

If E: capture their description verbatim — it becomes the style prompt modifier.

### Q3: Color palette

```text
Should generated images follow a brand color palette?

  A — Yes — I'll provide hex codes
  B — No specific palette — let the AI choose colors based on content
```

If A: follow up immediately: "Please enter 2–4 hex codes (e.g. #1A1A2E, #E94560). Label them if you like (primary, accent, etc.)."

### Q4: Placement

```text
When should images be added to an article?

  A — AI decides based on content structure (Recommended — natural, context-aware placement)
  B — Hero image + one image per H2 section (comprehensive)
  C — Hero image only (minimal)
```

Also ask: "What is the minimum article word count before images are generated? (e.g. 300 — skip images for very short articles)"

### Q5: Skip conditions

```text
When should image generation be skipped? (select all that apply)

  A — Glossary entries (usually too short)
  B — Articles under the minimum word count you set
  C — Translation runs (images already exist from the original language)
  D — Any other conditions? (describe below)
```

---

## Phase 3: Create image style guide

Using the answers from Phase 2, create `docs/image-style-guide.md`.

The guide must be **actionable** — it is read verbatim by the `image-generator` agent to build image prompts and make placement decisions.

Include these sections:

### Provider

Which API to use and any relevant notes.

```text
Provider: [google-gemini | openai-gpt-image | manual]
API key environment variable: [GEMINI_API_KEY | OPENAI_API_KEY | as specified]
```

### Visual Style

A clear, specific description the agent can embed directly into image prompts.

Include:
- Style name (e.g., "flat illustration")
- 3–5 adjective/descriptor keywords for prompt use (e.g., "clean lines, bold shapes, minimal texture")
- What to avoid (e.g., "no photorealism, no drop shadows, no gradients")
- One example prompt fragment showing how the style should be expressed

### Color Palette

If a palette was provided:
- List each color with its hex code and label (primary, accent, background, etc.)
- Instruction for the agent: "Always include these colors as dominant tones in generated images"

If no palette: "Let the API choose colors based on content context. Prefer clean, professional tones."

### Placement Rules

```text
Mode: [ai-driven | hero-plus-sections | hero-only]
Minimum word count: [N] — skip image generation for articles below this threshold
```

If mode is `ai-driven`, add guidance for the agent:
- Add a hero image for every article that passes the word count threshold
- Add a section image when a concept is abstract, visual, or benefits from illustration
- Skip section images when the content is already concrete (e.g., code examples, numbered steps, short factual sections)
- Prefer fewer, higher-quality images over one per section by default

### Skip Conditions

List the conditions under which the `image-generator` agent should do nothing:

- `content_type: glossary` — always skip
- `word_count < [N]` — skip if article is too short
- `is_translation: true` — skip if this is a translation run
- Any user-specified additional conditions

### Output Settings

```text
Hero image dimensions: 1200 × 630 px
Inline image dimensions: 800 × 450 px
Output format: webp (fallback: png)
Output path pattern: public/images/articles/{slug}/
Hero filename: hero.webp
Inline filename: section-{n}.webp
```

### Alt Text Convention

Instructions for generating SEO-friendly alt text:
- Describe what the image shows, not what it "is"
- Include the article topic keyword naturally
- Keep it under 125 characters
- No "image of" or "photo of" prefix

### Base Prompt Template

A reusable prompt fragment the agent should append to every generation call:

```text
[Style keywords from Visual Style section], [color palette instruction], white or neutral background,
no text overlays, no watermarks, professional quality, suitable for a blog article about [topic]
```

---

## Phase 4: Update config

Update `.content-ops/config.md` — add or update the `image_generation` block:

```yaml
# Image generation
image_generation:
  enabled: true
  provider: "[google-gemini | openai-gpt-image | manual]"
  guidelines: "docs/image-style-guide.md"
  output_path: "public/images"
  hero_dimensions: [1200, 630]
  inline_dimensions: [800, 450]
  placement: "[ai-driven | hero-plus-sections | hero-only]"
  min_word_count: [N]
  skip_types: ["glossary"]
```

Preserve all other existing config fields.

---

## Phase 5: Confirm and guide

```text
✅ Image style guide created at docs/image-style-guide.md

Settings:
  • Provider: [provider]
  • Style: [style name]
  • Placement: [placement mode]
  • Min word count: [N]
  • Output: public/images/articles/{slug}/

Config updated with image_generation settings.

Before running /write-content, set your API key:
  export [GEMINI_API_KEY | OPENAI_API_KEY]=[your-key]

→ Next: [next incomplete round, or "All rounds complete — run /reindex to build the content index"]
```
