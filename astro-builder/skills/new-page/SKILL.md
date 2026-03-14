---
description: Scaffold a new page in the Astro 6 project. Creates the page file(s), a matching page-view, wires i18n, and registers the route for all configured locales. Arguments: page name and optional path (e.g. "about" or "blog/archive").
---

# /astro-builder:new-page $ARGUMENTS

You are scaffolding a new page in this Astro 6 project. The argument is the page name or path (e.g. `about`, `blog/archive`, `tags/[tag]`).

## Step 1 — Read project context

Before writing any code:

1. Read `CLAUDE.md` and `.astro-builder/content-schema.md` to understand the project.
2. Read `astro.config.ts` to get the i18n config (locales, default locale, routing).
3. Read `src/i18n/en.json` (or the default locale file) to understand the translation key format.
4. Look at an existing page in `src/pages/` to understand the thin-wrapper pattern.
5. Look at an existing page-view in `src/page-views/` to understand the page-view pattern.

## Step 2 — Plan

Based on the argument `$ARGUMENTS`, determine:
- The page slug and any dynamic segments (e.g. `[tag]`, `[...slug]`).
- Whether this page needs data from a content collection.
- What i18n keys will be needed (page title, meta description, any UI strings).

Announce the plan: list the files you will create and the i18n keys you will add.

## Step 3 — Create files

### 3.1 Page files (one per locale)

For each configured locale, create `src/pages/{locale}/{path}.astro`:

```astro
---
import {PageViewName} from "@page-views/{PageViewName}.astro";
---

<PageViewName />
```

That is all. No imports of data, no props, no logic. The page is a thin wrapper.

### 3.2 Page-view file

Create `src/page-views/{PageViewName}.astro`:

```astro
---
import { createTranslator } from "@lib/i18n";
import BaseLayout from "@layouts/BaseLayout.astro";

const tl = createTranslator(Astro.currentLocale);
const locale = Astro.currentLocale ?? "en";

// Data fetching goes here if needed
---

<BaseLayout title={tl("pageName.title")} description={tl("pageName.description")}>
  <!-- Page markup here -->
</BaseLayout>
```

### 3.3 Add i18n keys

For each locale JSON file in `src/i18n/`, add the required translation keys:
- `{pageName}.title` — page title
- `{pageName}.description` — meta description
- Any other UI strings specific to this page

### 3.4 Dynamic pages

If the route has a dynamic segment (e.g. `[tag]`), add `getStaticPaths()` to the page-view, not the page file.

## Step 4 — Verify

Run `pnpm build`. If there are TypeScript or build errors, fix them before finishing.

Report what was created:
- List all new files
- List all i18n keys added
- Show the URL(s) the page will be available at (e.g. `/en/about`, `/it/chi-siamo`)

## Constraints

- Never put data fetching, props, or logic in page files — pages are thin wrappers only.
- Never hardcode UI strings in `.astro` files — always use `tl('key')`.
- Never parse `Astro.url` to detect locale — always use `Astro.currentLocale`.
- Always create one page file per locale (do not use `getStaticPaths` to handle locales in a single page).
- Follow the path alias conventions from `CLAUDE.md` (`@/*`, `@components/*`, etc.).
- Always follow the Astro 6 documentation: https://docs.astro.build/llms-small.txt
