# Astro 6 Canonical Patterns and Anti-Patterns

This document is the authoritative reference for Astro 6 architectural patterns enforced by the astro-builder plugin.

**Always verify against the live docs**: https://docs.astro.build/llms-small.txt

---

## Canonical Patterns

### Project configuration

```typescript
// astro.config.ts — always TypeScript, never .mjs
import { defineConfig } from "astro/config";
import vercel from "@astrojs/vercel";
import sitemap from "@astrojs/sitemap";

export default defineConfig({
  site: "https://example.com",
  output: "static",
  adapter: vercel(),
  integrations: [sitemap()],
  i18n: {
    defaultLocale: "en",
    locales: ["en", "it"],
    routing: {
      prefixDefaultLocale: true,
      redirectToDefaultLocale: false,
    },
  },
  redirects: {
    "/": "/en",
  },
});
```

### Content collections (Astro 6)

```typescript
// src/content.config.ts — at project root, NOT src/content/config.ts
import { defineCollection, z } from "astro:content";
import { glob } from "astro/loaders";

const articles = defineCollection({
  loader: glob({ pattern: "**/*.{md,mdx}", base: "./src/content/articles" }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    date: z.coerce.date(),
    tags: z.array(z.string()).default([]),
    translationKey: z.string().optional(),
    lang: z.enum(["en", "it"]).default("en"),
    draft: z.boolean().default(false),
  }),
});

export const collections = { articles };
```

Content folder structure — single collection, locale subfolders:
```
src/content/
└── articles/
    ├── en/
    │   └── my-article.md
    └── it/
        └── my-article.md   ← same slug, different lang
```

### Page-views pattern

```
src/pages/en/about.astro          ← thin wrapper (≤5 lines)
src/page-views/AboutPageView.astro ← all markup and data
src/layouts/BaseLayout.astro      ← page shell
```

**Page file** (`src/pages/en/about.astro`):
```astro
---
import AboutPageView from "@page-views/AboutPageView.astro";
---
<AboutPageView />
```

**Page-view** (`src/page-views/AboutPageView.astro`):
```astro
---
import { createTranslator } from "@lib/i18n";
import BaseLayout from "@layouts/BaseLayout.astro";

const tl = createTranslator(Astro.currentLocale);
const locale = Astro.currentLocale ?? "en";

const { data } = await getEntry("pages", `${locale}/about`);
---
<BaseLayout title={tl("about.title")} description={tl("about.description")}>
  <!-- markup here -->
</BaseLayout>
```

### i18n

```typescript
// src/lib/i18n.ts
import en from "@i18n/en.json";
import it from "@i18n/it.json";

const translations = { en, it } as const;
type Locale = keyof typeof translations;
type TranslationKey = keyof typeof en;

export function createTranslator(locale: string | undefined) {
  const lang = (locale ?? "en") as Locale;
  const t = translations[lang] ?? translations.en;
  return (key: TranslationKey): string => t[key] ?? en[key] ?? key;
}
```

Usage in components — never passed as props:
```astro
---
// In any component or page-view
import { createTranslator } from "@lib/i18n";
const tl = createTranslator(Astro.currentLocale);
---
<h1>{tl("about.title")}</h1>
```

### Dynamic routes with content collections

```astro
---
// src/page-views/ArticlePageView.astro
import { getCollection } from "astro:content";
import { render } from "astro:content";

export async function getStaticPaths() {
  const articles = await getCollection("articles");
  return articles.map((article) => ({
    params: { slug: article.id.split("/").pop() },
    props: { article },
  }));
}

const { article } = Astro.props;
const { Content } = await render(article);
---
```

### URL builders

```typescript
// src/lib/urls.ts
export function buildArticleUrl(slug: string, lang: string): string {
  return `/${lang}/articles/${slug}`;
}

export function buildTagUrl(tag: string, lang: string): string {
  return `/${lang}/tags/${encodeURIComponent(tag)}`;
}
```

### Path aliases (tsconfig.json)

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@layouts/*": ["./src/layouts/*"],
      "@styles/*": ["./src/styles/*"],
      "@lib/*": ["./src/lib/*"],
      "@page-views/*": ["./src/page-views/*"],
      "@i18n/*": ["./src/i18n/*"]
    }
  }
}
```

---

## Anti-Patterns

These are explicitly forbidden. Any code matching these patterns must be refactored.

| Anti-pattern | Why | Correct pattern |
|---|---|---|
| `src/content/config.ts` | Astro 4 pattern, ignored in Astro 6 | `src/content.config.ts` |
| `astro.config.mjs` | JavaScript config loses type safety | `astro.config.ts` |
| Separate collections per language | Duplicates schema, breaks `reference()` | Single collection with lang subfolders |
| Fixed `category` enum | Inflexible, hard to extend | `tags: string[]` |
| Hardcoded UI strings in `.astro` | Breaks i18n, untranslatable | `tl('key')` from JSON |
| Logic/data in page files | Breaks page-views pattern | Move to page-view |
| `lang` or `tl` as props | Creates tight coupling across component tree | Each component calls `createTranslator(Astro.currentLocale)` |
| Parse `Astro.url` for locale | Fragile, breaks on URL changes | `Astro.currentLocale` |
| `prefixDefaultLocale: false` | Default locale has no prefix, links break | `prefixDefaultLocale: true` |
| `redirectToDefaultLocale: true` | Causes redirect flash in production | `redirectToDefaultLocale: false` + explicit `redirects` |
| ESLint + Prettier | Two tools for one job, config conflicts | Biome (replaces both) |
| `npm install` or `yarn add` | Inconsistent lockfile | `pnpm install` / `pnpm add` |
| Custom sitemap | Maintenance burden | `@astrojs/sitemap` |
| Custom RSS | Maintenance burden | `@astrojs/rss` |
| JS-heavy UI frameworks in page-views | Breaks SSG, inflates bundle | Astro components + vanilla JS only |
| Relative imports across `src/` | Brittle path strings | Path aliases (`@lib/`, `@components/`, etc.) |
| `// @ts-ignore` in new code | Hides real errors | Fix the type error |
| `git add .` in commits | May include unintended files | Stage specific files |

---

## Quality gate commands

```bash
pnpm build          # Zero build errors required
tsc --noEmit        # Zero TypeScript errors required
biome check .       # Zero lint/format errors required
biome check --write . # Auto-fix formatting (run before committing)
```

---

## Official integrations (use only these)

| Purpose | Package |
|---|---|
| Sitemap | `@astrojs/sitemap` |
| RSS feed | `@astrojs/rss` |
| MDX support | `@astrojs/mdx` |
| Vercel deployment | `@astrojs/vercel` |
| Netlify deployment | `@astrojs/netlify` |
| Cloudflare deployment | `@astrojs/cloudflare` |
| Tailwind CSS | `@astrojs/tailwind` |
| Image optimization | Built-in `<Image />` from `astro:assets` |

Never install community adapters when an official one exists.
