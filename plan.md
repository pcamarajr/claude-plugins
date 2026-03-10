# Plan: Add Image Generation to content-ops

## Overview

Add AI-powered image generation to the content-ops plugin using Google Gemini (Nano Banana) as the default provider. This includes a new `/init images` round for configuring image guidelines, a new agent for image generation, and integration into the write-content pipeline.

---

## Files to Create

### 1. `skills/init/rounds/images.md` — Init Round 6: Images

New init round that asks the user about their image preferences and creates an image style guide.

**Phase 1: Check existing state**
- Read `.content-ops/config.md`, check if `image_generation` section exists
- If exists: summarize and offer to keep/update/replace (same pattern as style round)

**Phase 2: Image style interview** (4-5 questions via AskUserQuestion)

- **Q1: Image provider** — Which provider to use?
  - A: Google Gemini / Nano Banana (Recommended — good quality, style consistency, $0.04/image)
  - B: OpenAI GPT Image (highest quality benchmarks)
  - C: Other (user provides details)

- **Q2: Visual style** — What style should generated images follow?
  - A: Flat illustration (clean, modern, great for tech blogs)
  - B: Photorealistic (realistic stock-photo-like images)
  - C: Watercolor / artistic (softer, editorial feel)
  - D: Minimalist diagram (technical, clean lines)
  - User can also describe their own style in detail.

- **Q3: Color palette** — Brand colors to use in images?
  - Ask for 2-4 hex codes (primary, secondary, accent)
  - Or: "No specific palette — let the AI decide based on content"

- **Q4: Placement rules** — When should images be added?
  - A: Hero image + one per H2 section (comprehensive)
  - B: Hero image only (minimal)
  - C: AI decides based on content length and structure (recommended)
  - Also ask: minimum article word count before images are added (e.g., skip images for articles under 300 words)

- **Q5: Exclusions** — When should images NOT be added?
  - Multi-select: glossary entries, short articles (under N words), translations, user can add custom rules

- **Q6: Image dimensions** — What sizes to generate?
  - Default: 1200x630 (hero/OG) + 800x450 (inline sections)
  - User can customize

**Phase 3: Create image style guide**

Create `docs/image-style-guide.md` with sections:
- **Provider** — which API to use
- **Visual Style** — description + reference keywords for prompts
- **Color Palette** — hex codes and usage guidance
- **Placement Rules** — when to add, when not to add, minimum word count
- **Dimensions** — hero size, inline size
- **Alt Text Convention** — SEO-friendly, descriptive, include article topic keywords
- **Prompt Engineering Notes** — base prompt modifiers to always include (e.g., "flat illustration style, using colors #1A1A2E and #E94560, clean background, no text")

**Phase 4: Update config**

Add to `.content-ops/config.md`:
```yaml
# Image generation
image_generation:
  enabled: true
  provider: "gemini"
  guidelines: "docs/image-style-guide.md"
  output_path: "public/images"
  hero_dimensions: [1200, 630]
  inline_dimensions: [800, 450]
  placement: "ai-driven"
  min_word_count: 300
  skip_types: ["glossary"]
```

**Phase 5: Confirm**
```
✅ Image style guide created at docs/image-style-guide.md
Config updated with image_generation settings.
→ Next: [first incomplete round]
```

---

### 2. `agents/image-generator.md` — Image Generator Agent

New focused agent (follows same pattern as draft-writer, content-researcher, etc.)

**Agent definition:**
```yaml
name: image-generator
description: |-
  Generates images for articles using AI image generation APIs. Receives article content,
  image guidelines, and placement decisions from the write-content orchestrator.
  Produces image files and returns markdown references to insert into the article.
tools: Read, Bash, Glob, Grep
model: sonnet
color: cyan
```

**Agent responsibilities:**

1. **Read the image style guide** (from `image_generation.guidelines` in config)
2. **Analyze the article** — read the written draft, identify:
   - Article title and topic (for hero image)
   - H2 sections and their key concepts (for inline images)
   - Overall tone and subject matter
3. **Decide placement** — based on guidelines:
   - If `placement: "ai-driven"`: decide which sections benefit from an image and which don't. Not every section needs one. Consider: does the concept benefit from visual representation? Is it abstract enough to need illustration?
   - Skip if article word count < `min_word_count`
   - Skip if content type is in `skip_types`
4. **Generate prompts** — for each image:
   - Incorporate the visual style from guidelines
   - Include color palette constraints
   - Reference the specific section's content
   - Add the base prompt modifiers from the style guide
   - Include dimensions
5. **Call the image API** — using Bash to make API calls:
   - For Gemini: use `curl` to call the Gemini API with the generated prompt
   - Save images to `{output_path}/articles/{slug}/hero.webp`, `{output_path}/articles/{slug}/section-{n}.webp`
6. **Generate SEO alt text** — descriptive, keyword-rich, based on article topic
7. **Return results** — list of image paths and corresponding markdown to insert

**Output format:**
```text
## Images Generated

### Placement Plan
- Hero: [description of what was generated]
- Section "H2 title": [description] (or "skipped — concept is too abstract")
- ...

### Files Created
- public/images/articles/{slug}/hero.webp
- public/images/articles/{slug}/section-1.webp
- ...

### Markdown References
[Ordered list of markdown image tags with alt text, ready to insert at specific locations]

### Notes
[Any issues, e.g., "API rate limited, retried successfully"]
```

---

### 3. `skills/content-image-style/SKILL.md` — Image Style Reference (auto-loaded)

Similar to `content-style` skill — an auto-loaded, non-user-invocable skill that provides image style rules to agents.

```yaml
name: content-image-style
description: Image style reference — visual guidelines, prompt templates, and placement rules for image generation. Reads project-specific values from plugin config.
user-invocable: false
```

Content:
- Condensed reference of image guidelines (like content-style does for writing)
- Prompt template patterns
- Default dimensions
- Placement decision framework
- Alt text conventions
- File naming conventions (`{output_path}/articles/{slug}/hero.webp`, etc.)

---

## Files to Modify

### 4. `skills/init/SKILL.md` — Add `images` round

Changes:
- Add `images` to the description: `"Rounds: project, content-types, style, strategy, infra, images."`
- Add routing for `images` → Read `skills/init/rounds/images.md`
- Add `images` to the unknown argument list
- Add completion check to Status Dashboard:
  ```
  | images | `image_generation` is set in config AND `image_generation.guidelines` file exists on disk |
  ```
- Add dashboard line:
  ```
  ⬜ /init images         — not started
  ```
- Add to "What each step does":
  ```
  images         — Configure image generation: style, colors, placement rules
  ```

### 5. `skills/write-content/SKILL.md` — Add Phase 5.5: Image Generation

Insert a new phase between Phase 5 (Write) and Phase 6 (Style Review):

**Phase 5.5: Image Generation**

```
## Phase 5.5: Image Generation

**Skip this phase if:**
- `image_generation.enabled` is not `true` in config
- The content type is in `image_generation.skip_types`
- The article word count is below `image_generation.min_word_count`

Spawn the `image-generator` agent via the Task tool:

    Use the image-generator agent.

    Article path: [file path from Phase 5]
    Article slug: [slug]
    Content type: [article|glossary]
    Image output path: [image_generation.output_path from config]/articles/[slug]/
    Image guidelines: [image_generation.guidelines from config]
    Hero dimensions: [image_generation.hero_dimensions from config]
    Inline dimensions: [image_generation.inline_dimensions from config]
    Placement mode: [image_generation.placement from config]

    Analyze the article, decide image placement, generate images, and save them.
    Return the list of image paths and markdown references.

After the agent returns:
1. Insert the hero image markdown after the frontmatter (before the first paragraph)
2. Insert inline images before their corresponding H2 sections
3. Store the list of created image files — they are included in the Phase 9 commit
```

Also update Phase 9 (Reindex + Build + Commit):
- Add image files to the `git add` list
- Update commit message template:
  ```
  content: add article "<title>"

  - New article: [article path]
  - Images: [count] generated at [image output path]
  - New glossary: [list or "none"]
  - Updated: [list modified files]
  ```

### 6. `config.example.md` — Add image_generation section

Add the `image_generation` config block with comments explaining each field. Add to the Schema Reference table and .content-ops/ Layout sections.

### 7. `agents/draft-writer.md` — Awareness of image phase

Add a note to the draft-writer rules:
- "Do not add image placeholders or references — the image-generator agent handles images in a later phase."

This prevents the draft-writer from adding `![](...)` tags that would conflict with the image generator.

### 8. `README.md` — Document the new feature

Add:
- `/init images` to the setup rounds list
- Image generation to the write-content pipeline description
- New config keys to the configuration reference
- Mention the image-generator agent

---

## Implementation Order

1. Create `skills/init/rounds/images.md` (init round definition)
2. Create `skills/content-image-style/SKILL.md` (auto-loaded image style reference)
3. Create `agents/image-generator.md` (image generator agent)
4. Modify `skills/init/SKILL.md` (add images round routing + dashboard)
5. Modify `skills/write-content/SKILL.md` (add Phase 5.5)
6. Modify `config.example.md` (add image_generation section)
7. Modify `agents/draft-writer.md` (add note about images)
8. Modify `README.md` (document the feature)

---

## Design Decisions

- **Provider via Bash/curl**: Since the plugin has no npm dependencies, image API calls use `curl` via Bash. This keeps the plugin dependency-free. The API key is expected as an environment variable (e.g., `GEMINI_API_KEY`).
- **WebP output format**: Modern, good compression, widely supported. Falls back to PNG if the API doesn't support WebP.
- **AI-driven placement**: The image-generator agent reads the full article and decides placement based on content structure + guidelines. This produces more natural results than rigid rules.
- **Separate agent**: Follows the existing pattern of focused, single-responsibility agents. The image-generator doesn't write articles and the draft-writer doesn't generate images.
- **Auto-loaded skill**: `content-image-style` follows the same pattern as `content-style` — loaded automatically when image-related work is happening, providing condensed reference rules.
