# astro-lsp

Astro language server (`@astrojs/language-server`) for Claude Code, providing code intelligence, diagnostics, and formatting for `.astro` files.

## Supported Extensions

`.astro`

## Installation

Install the Astro language server globally via npm:

```bash
npm install -g @astrojs/language-server
```

Or with yarn:

```bash
yarn global add @astrojs/language-server
```

## Capabilities

- Diagnostics and error checking for `.astro` files
- Go-to-definition across Astro components
- Find references and hover documentation
- TypeScript/JavaScript intellisense inside frontmatter and script blocks
- CSS/SCSS/Less support inside `<style>` blocks
- Prettier formatting via `prettier-plugin-astro` (optional)

For Prettier formatting support, also install:

```bash
npm install -g prettier prettier-plugin-astro
```

## More Information

- [Astro Language Tools on GitHub](https://github.com/withastro/astro/tree/main/packages/language-tools/language-server)
- [`@astrojs/language-server` on npm](https://www.npmjs.com/package/@astrojs/language-server)
- [Astro Documentation](https://docs.astro.build/)
