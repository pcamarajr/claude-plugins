# content-stack

A curated collection of Claude Code plugins by [@pcamarajr](https://github.com/pcamarajr), covering content operations, SEO, and static site tooling.

## Installation

Add this marketplace to your Claude Code project:

```
/plugin marketplace add pcamarajr/content-stack
```

## Available Plugins

### [astro-lsp](./astro-lsp/README.md)

Astro language server for Claude Code. Provides code intelligence, diagnostics, and formatting for `.astro` files. Automatically installs the language server binary at session start — works out of the box in remote and cloud environments.

To install this plugin:

```
/plugin install astro-lsp@content-stack
```

### [content-ops](./content-ops/README.md)

Content creation and management plugin for static site blogs. Handles writing, translation, research, internal linking, style review, and knowledge indexing.

**Agents:** content-linker, content-researcher, draft-writer, glossary-creator, image-generator, style-enforcer

**Skills:** write-content, translate, review-content, internal-linking, fact-check, suggest-content, content-style, content-image-style, content-inventory, reindex, update-trackers, init

To install this plugin:

```
/plugin install content-ops@content-stack
```

## License

MIT
