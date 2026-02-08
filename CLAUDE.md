# Ultra Potato — Obsidian Theme

## Project Overview

Ultra Potato is an Obsidian CSS theme forked from [Ultra Lobster](https://github.com/7368697661/Ultra-Lobster) by kneecaps (MIT licensed). It is being refactored from a monolithic 7,827-line CSS file into a modular SCSS architecture.

Read `docs/claude-refactor.md` for the full audit findings, architecture proposal, and migration plan.

## Current State

The theme is in **active refactoring** following a 7-phase migration plan. Check the migration phase tracker below before making changes.

### Migration Phases
- [ ] Phase 0: Quick wins (dead code removal, broken selector fixes)
- [ ] Phase 1: Extract foundations (fonts, keyframes, @settings)
- [ ] Phase 2: Extract token system (CSS custom properties)
- [ ] Phase 3: Refactor color schemes (generator mixin)
- [ ] Phase 4: Extract components (workspace, editor, callouts, canvas, etc.)
- [ ] Phase 5: Isolate plugin overrides
- [ ] Phase 6: Eliminate `!important`
- [ ] Phase 7: Polish (a11y, media queries, minification)

## Build System

- **Preprocessor:** SCSS (Dart Sass)
- **Output:** Single `theme.css` file (Obsidian requirement)
- **Commands:**
  - `pnpm run dev` — watch mode, auto-compiles on save
  - `pnpm run build` — one-time build (expanded)
  - `pnpm run build:min` — production build (compressed)
  - `pnpm run lint` — stylelint check
  - `pnpm run lint:fix` — auto-fix lint issues
  - `pnpm run preflight` — lint + build (run before committing)

## Key Constraints

- **Obsidian loads only `theme.css`** from the theme folder. All SCSS must compile to this single file.
- **Fonts must use `url()` paths** to WOFF2 files in `fonts/`, not base64 embedding.
- **Style Settings plugin** requires a single contiguous `/* @settings ... */` comment block at the top of the compiled CSS.
- **No `!important`** — the stylelint config blocks it. If absolutely needed, document why.
- **Selector specificity cap:** `0,4,2` enforced by stylelint.

## Architecture

```
ultra-potato/
├── theme.css              # Build output — DO NOT EDIT DIRECTLY
├── manifest.json          # Obsidian theme manifest
├── fonts/                 # Extracted WOFF2 font files
└── src/
    ├── main.scss          # Entry point
    ├── _settings.scss     # Style Settings YAML block
    ├── tokens/            # Design tokens (colors, typography, spacing, elevation, motion)
    ├── foundations/        # Reset, CSS custom properties, @font-face, @keyframes
    ├── color-schemes/     # Generator mixin + seed files per scheme
    ├── components/        # UI components (workspace, editor, callouts, canvas, etc.)
    ├── plugins/           # Isolated third-party plugin overrides
    └── utilities/         # Shared mixins, functions, a11y
```

## CSS Variable Convention

Three-tier token system with `--ul-` prefix:

| Tier | Purpose | Example |
|------|---------|---------|
| Primitive | Raw color values | `--ul-red-500`, `--ul-gray-800` |
| Semantic | Role-based, theme-switchable | `--ul-bg-primary`, `--ul-text-normal`, `--ul-accent-primary` |
| Component | Specific UI element | `--ul-heading-color`, `--ul-callout-bg` |

## Color Schemes

Color schemes use a **generator mixin** — each scheme provides 7-8 seed values that expand into the full palette:

```scss
@include generate-scheme('catppuccin-mocha', (
  'bg': #1e1e2e, 'surface': #313244, 'text': #cdd6f4,
  'accent': #89b4fa, 'red': #f38ba8, 'green': #a6e3a1, 'yellow': #f9e2af,
));
```

Do NOT copy-paste full palettes for new color schemes. Use the generator.

## Rules

- **Never edit `theme.css` directly** — it is a build artifact. Edit files in `src/`.
- **Never use `!important`** — fix specificity structurally instead.
- **Never use `aria-label` selectors** — they break on Obsidian updates and i18n.
- **Never use Svelte-generated class selectors** (`.svelte-*`) — they change on every plugin rebuild.
- **Never embed fonts as base64** — use `url()` references to `fonts/` directory.
- **Isolate plugin overrides** in `src/plugins/` with documented version and fragile selector notes.
- **Validate after every change** — rebuild and check Obsidian visually. Toggle Style Settings options to confirm nothing broke.

## Testing

There is no automated test suite. Validation is manual:

1. `pnpm run preflight` — must pass lint + build
2. Open Obsidian with Ultra Potato active
3. Check representative content: headings, code blocks, callouts, tables, canvas, graph view
4. Toggle all Style Settings options
5. Switch between light/dark mode and 2-3 color schemes

## Symlink Setup

This repo lives outside Obsidian vaults. To use in a vault:

```bash
ln -s ~/code/obsidian-themes/ultra-potato "/path/to/vault/.obsidian/themes/Ultra Potato"
```
