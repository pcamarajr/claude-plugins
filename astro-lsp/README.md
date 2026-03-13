# astro-lsp

Astro language server (`@astrojs/language-server`) for Claude Code, providing code intelligence, diagnostics, and formatting for `.astro` files.

## Supported Extensions

`.astro`

## Installation

First, add the content-stack marketplace to your project:

```bash
/plugin marketplace add pcamarajr/content-stack
```

Then install the plugin:

```bash
/plugin install astro-lsp@content-stack
```

The plugin will automatically install `@astrojs/language-server` at session start if it is not already available on your system. This means it works out of the box in remote environments such as GitHub Codespaces and other cloud development setups — no manual setup required.

## Capabilities

- Diagnostics and error checking for `.astro` files
- Go-to-definition across Astro components
- Find references and hover documentation
- TypeScript/JavaScript intellisense inside frontmatter and script blocks
- CSS/SCSS/Less support inside `<style>` blocks

## More Information

- [Astro Language Tools on GitHub](https://github.com/withastro/astro/tree/main/packages/language-tools/language-server)
- [`@astrojs/language-server` on npm](https://www.npmjs.com/package/@astrojs/language-server)
- [Astro Documentation](https://docs.astro.build/)
