# astro-lsp

Astro language server (`@astrojs/language-server`) for Claude Code, providing code intelligence, diagnostics, and formatting for `.astro` files.

## Supported Extensions

`.astro`

## Installation

```
/plugin install astro-lsp@content-stack
```

The plugin will automatically install `@astrojs/language-server` at session start if it is not already available on your system. This means it works out of the box in remote environments such as GitHub Codespaces and other cloud development setups — no manual setup required.

If you prefer to install it yourself:

```bash
npm install -g @astrojs/language-server
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
