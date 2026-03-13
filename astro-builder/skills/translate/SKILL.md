---
description: Translate a content entry or page to another locale. Reads the source file, translates content while preserving frontmatter structure and markdown formatting, creates the translated file in the correct locale folder, and links it via translationKey. Arguments: file path and target locale (e.g. "src/content/articles/en/hello-world.md it").
---

# /astro-builder:translate $ARGUMENTS

You are translating content in this Astro 6 project. The argument format is: `{file-path} {target-locale}` (e.g. `src/content/articles/en/hello-world.md it`).

## Step 1 — Parse arguments and read context

Parse `$ARGUMENTS` to extract:
- `sourcePath`: the file to translate
- `targetLocale`: the target language code (e.g. `it`, `fr`, `de`)

Then read:
1. `CLAUDE.md` and `.astro-builder/style-guide.md` for tone, voice, and writing conventions.
2. `astro.config.ts` to confirm the target locale is configured.
3. The source file at `sourcePath`.
4. Any existing translated files in the target locale folder to understand the expected style.
5. `src/i18n/{targetLocale}.json` to understand translated UI strings (reference only).

## Step 2 — Validate

Check:
- The target locale exists in `astro.config.ts`. If not, stop and inform the user.
- The source file exists. If not, stop and inform the user.
- A translation doesn't already exist (look for a file with the same slug in the target locale folder). If it exists, ask the user whether to overwrite or create a new version.

## Step 3 — Translate the content

This is **localization**, not word-for-word translation. Apply these rules:

- Preserve all frontmatter fields. Translate: `title`, `description`, and any text fields. Do NOT translate: `date`, `tags` (translate their meaning but keep slugs consistent), `translationKey`, `lang` (change to target locale value), `slug`.
- Translate the body content naturally, matching the tone and voice defined in `.astro-builder/style-guide.md`.
- Preserve all markdown formatting: headings, bold, italic, code blocks, links, lists.
- Preserve all relative links (they remain the same path).
- Do NOT translate code blocks — only translate comments within code blocks if present.
- Adapt cultural references, idioms, and examples to feel natural in the target language — do not translate literally.
- Keep technical terms consistent. If a glossary or term list exists in `.astro-builder/`, follow it.

## Step 4 — Create the translated file

Determine the output path:
- Source: `src/content/articles/en/hello-world.md`
- Target: `src/content/articles/it/hello-world.md`

The slug (filename) stays the same across locales.

Update frontmatter:
- Set `lang: {targetLocale}`
- Ensure `translationKey` is set and matches the source file. If the source doesn't have a `translationKey`, generate one (use the slug) and add it to both files.

Write the translated file.

## Step 5 — Verify

1. Check that both source and target files have matching `translationKey` values.
2. Run `pnpm build`. Fix any errors before finishing.

Report:
- Source file translated
- Output file created
- `translationKey` used to link them
- Any cultural adaptations or notable translation decisions made

## Translating i18n UI strings

If `$ARGUMENTS` refers to an i18n JSON file (e.g. `src/i18n/en.json it`), translate all string values from the source locale JSON to create the target locale JSON. Preserve all keys exactly. Keep placeholders like `{count}`, `{name}` unchanged.

## Constraints

- Never machine-translate mechanically — localize for natural reading.
- Never change slugs/filenames when translating.
- Always link source and target via `translationKey`.
- Always follow `.astro-builder/style-guide.md` for tone and voice.
- Follow MDN Web API references for any web-related terminology.
