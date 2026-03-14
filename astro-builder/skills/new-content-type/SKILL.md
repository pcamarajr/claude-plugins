---
description: Add a new content type (Astro 6 content collection) to the project. Defines the schema in src/content.config.ts, creates example content files, and wires up utility functions. Arguments: content type name (e.g. "tutorials", "changelogs", "case-studies").
---

# /astro-builder:new-content-type $ARGUMENTS

You are adding a new Astro 6 content collection to this project. The argument is the collection name (e.g. `tutorials`, `changelogs`, `case-studies`).

## Step 1 — Read project context

1. Read `CLAUDE.md` and `.astro-builder/content-schema.md` to understand existing content types.
2. Read `src/content.config.ts` to understand the existing collection definitions.
3. Read `src/lib/content.ts` to understand the utility function patterns.
4. Read `astro.config.ts` to get the i18n configuration (locales, default locale).
5. Look at an existing content collection folder (e.g. `src/content/articles/`) to understand the folder structure.

## Step 2 — Interview the user

Ask questions one at a time to define the new content type. Use the `AskUserQuestion` tool.

Ask:
1. What fields does this content type have? (title, date, author, tags, description, image, etc.)
2. Are any fields optional? Which ones are required?
3. Does this content type need to be translatable (i.e. support multiple locales)?
4. Does this content type reference other collections (e.g. articles reference glossary entries)?
5. What is the URL pattern for individual entries? (e.g. `/en/tutorials/[slug]`)

## Step 3 — Define the schema

### 3.1 Update `src/content.config.ts`

Add the new collection using Astro 6 `glob()` loader and `defineCollection()`. If multilingual:

```typescript
const tutorials = defineCollection({
  loader: glob({ pattern: "**/*.{md,mdx}", base: "./src/content/tutorials" }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    date: z.coerce.date(),
    tags: z.array(z.string()).default([]),
    translationKey: z.string().optional(),
    lang: z.enum(["en", "it"]).default("en"), // adjust to configured locales
    // add fields from the interview
  }),
});
```

Export the collection in the `collections` object.

### 3.2 Create content folders

For each configured locale, create the folder:
- `src/content/{name}/en/`
- `src/content/{name}/it/` (if multilingual)

Create one example `.md` file per locale with all required frontmatter fields populated with realistic placeholder content.

### 3.3 Update `src/lib/content.ts`

Add utility functions for the new collection following the existing patterns:

```typescript
export async function getTutorialsByLang(lang: string) {
  const all = await getCollection("tutorials");
  return all
    .filter((entry) => entry.data.lang === lang)
    .sort((a, b) => b.data.date.valueOf() - a.data.date.valueOf());
}
```

Add any other helpers that are appropriate (e.g. `getTutorialBySlug()`, `getAllTutorialTags()`).

### 3.4 Update URL builders in `src/lib/urls.ts`

Add a URL builder for this content type:

```typescript
export function buildTutorialUrl(slug: string, lang: string): string {
  return `/${lang}/tutorials/${slug}`;
}
```

## Step 4 — Scaffold pages (optional)

Ask the user: "Do you want me to scaffold the listing page and detail page for this content type?"

If yes, run `/astro-builder:new-page` for each needed page.

## Step 5 — Verify

Run `pnpm build` and `tsc --noEmit`. Fix any errors before finishing.

Report:
- Updated files and new files created
- Schema definition summary
- Example content file locations
- Utility functions added

## Constraints

- Never create separate collections per language — one collection with lang subfolders.
- Never use `src/content/config.ts` — always `src/content.config.ts`.
- Always use `glob()` loader (Astro 6 pattern).
- Always use flexible `tags: string[]` — never fixed category enums.
- Always use `translationKey` to link content across locales.
- Always follow the Astro 6 documentation: https://docs.astro.build/llms-small.txt
