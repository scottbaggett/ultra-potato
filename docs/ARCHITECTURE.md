# Ultra Potato Theme — Audit & Architecture Proposal

**Date:** 2026-02-08
**Prepared by:** Claude Agent team (css-auditor-1, css-auditor-2, theme-architect)
**For:** Scott Baggett (new maintainer)
**Original Theme:** Ultra Lobster v2.0.0.0 by kneecaps

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Audit: obsidian.css](#audit-obsidiancss)
3. [Audit: theme.css](#audit-themecss)
4. [Architecture Proposal](#architecture-proposal)
5. [Migration Plan](#migration-plan)
6. [Quick Wins](#quick-wins)

---

## Executive Summary

The original Ultra Lobster theme is a monolithic 7,827-line, 1.96MB CSS file suffering from massive duplication (base64 fonts, copy-pasted color palettes, triplicated canvas gradients), 314 `!important` declarations from specificity wars, and zero modularity. A secondary file (`obsidian.css`, 1,203 lines) is entirely dead code — Obsidian ignores it when `manifest.json` is present.

This document proposes redesigning it as a build-from-source SCSS project that compiles to the single `theme.css` Obsidian requires, cutting the file by ~60%, eliminating duplication, and making it maintainable.

---

## Audit: obsidian.css

**File:** `obsidian.css`
**Lines:** 1,203

### Critical Finding: This File Is Dead Code

In Obsidian's theme loading system:
- **Pre-1.0 (legacy):** Themes used `obsidian.css` as the sole CSS file
- **Post-1.0 (modern):** Themes use `manifest.json` + `theme.css` as the entry point

Since this theme has a `manifest.json` (version 2.0.0.0), **Obsidian loads `theme.css` and ignores `obsidian.css` entirely.** The file is a historical artifact from the original "Ultra Lobster" before it became "Ultra Lobster: Unity."

Evidence:
- obsidian.css identifies itself as `"Ultra Lobster"` with Style Settings id `ul-theme-settings`
- theme.css identifies itself as `"Ultra Lobster: Unity"` with Style Settings id `ulu-theme-settings`
- They use completely different CSS variable namespaces (`--ul-*` vs `--ulu-*`)
- They have different Style Settings configurations

### Structure Breakdown

| Lines | Section | Description |
|-------|---------|-------------|
| 1-5 | Header | Theme identification comment |
| 6-52 | Style Settings | `@settings` block (id: `ul-theme-settings`) |
| 54-55 | Font Imports | Google Fonts: Rubik, Space Mono via `@import url()` |
| 56-65 | Font Face | Zoho Puvi font embedded as base64 woff2 (~175KB inline) |
| 67-81 | `:root` Variables | Global custom properties |
| 82-193 | Color Schemes | 5 schemes using `--ul-*` variables |
| 194-277 | Variable Mapping | Maps `--ul-*` to Obsidian's native variables |
| 278-311 | Theme Mode | `.theme-dark` and `.theme-light` specifics |
| 312-465 | Body & UI | Inline title, fonts, links, workspace elements |
| 466-530 | Smooth Live Headers | Animated header transitions with keyframes |
| 531-600 | Tags & Lists | Custom bullets, tag styling |
| 601-700 | Content Formatting | Bold, italic, highlights, horizontal rules, callouts |
| 702-788 | Blockquotes | Custom blockquote styling |
| 789-852 | Workspace Layout | Tab headers, status bar, workspace splits |
| 852-887 | Concourse Index Font | Ordered list custom counter font (14KB base64) |
| 889-940 | Plugin: Influx | Influx plugin styling |
| 941-1027 | Plugins: Command Palette, Omnisearch | Modal styling |
| 1027-1080 | Plugins: Kanban, Reminder | Board and reminder styling |
| 1081-1128 | File Navigation | Enhanced nav folder/file styling |
| 1129-1203 | Plugin: Memo | Memos plugin styling |

### Overlap with theme.css

Both files are essentially complete, standalone themes. Duplicated code includes:
- **Smooth Live Headers animations** — identical keyframes in both files
- **Concourse Index font** — identical 14KB base64 font in both files
- **Workspace layout patterns** — styled in both with diverging approaches

### Specificity Issues

- **7 `!important` instances** (conservative)
- `body > div > div > div > div` selector chain — extremely fragile
- CSS-modules hashed class names (`.root-0-2-1`) — will break across plugin updates
- Triple class repetition hack: `.list-bullet.list-bullet.list-bullet:after`

### Recommendation

**Delete this file entirely.** It has zero impact on the theme. If kept for reference, rename to `obsidian.css.legacy`.

---

## Audit: theme.css

**File:** `theme.css`
**Lines:** 7,827
**File Size:** 1.96 MB

### Quantitative Summary

| Metric | Count |
|---|---|
| Total lines | 7,827 |
| File size | 1.96 MB (1,958,556 bytes) |
| Rule blocks (approx.) | 661 |
| Unique CSS custom properties | 606 |
| `!important` declarations | 314 |
| Unique `.ulu-*` class selectors | 124 |
| `@font-face` declarations | 17 (12 base64-encoded) |
| `@settings` blocks | 1 (massive: lines 9-774) |
| `@media` queries | 0 |
| `@keyframes` definitions | 12 |
| `transition` usages | 29 |
| `animation` usages | 18 |
| `display: none` usages | 10 |
| `aria-label` selectors (fragile) | 16 |
| External URL references | 1 (Pinterest test image) |

### Structure Map

| Line Range | Section |
|---|---|
| 1-6 | Header comment |
| 9-774 | **@settings block** — Single massive YAML block for Style Settings plugin |
| 775-885 | **Font imports** — 12+ base64-embedded `@font-face` declarations (~1MB!) |
| 886-920 | More font definitions (Aspekta, Getai) |
| 920-1560 | **Base variable definitions** in `body` selector |
| 1560-1738 | **`.theme-light`** and **`.theme-dark`** base colors |
| 1738-3235 | **Color theme variants** — ~40 sub-themes |
| 3236-3365 | **Accent color settings** (8 colors x light/dark) |
| 3365-3650 | **Accent-tinted backgrounds** (8 colors x 2 modes x 2 contrast) |
| 3650-3770 | Body-level variable overrides |
| 3770-4100 | **Custom CSS base** — layout, workspace structure |
| 4100-4500 | **UI components** — settings, modals, tooltips, menus |
| 4500-4800 | **Smooth Live Headers**, icon fades, tags |
| 4800-5000 | **Tabs** styling |
| 5000-5180 | **Navigation**, blockquotes, mobile, stacked tabs |
| 5180-5400 | **List bullet pins**, we3 callout styles |
| 5400-5600 | **Plugin support** (cMenuToolbar, calendar, kanban, etc.) |
| 5600-5730 | **Animations** (@keyframes) |
| 5730-6200 | **Ultra Lobster 2.0** — gradients, cards, canvas nodes |
| 6200-6600 | **Canvas gradient/gummy/fancy** color variants (massive duplication) |
| 6600-6700 | **Origami sub-theme**, **we3 sub-theme** overrides |
| 6700-6900 | Properties card, gummy workspace, codeblock styles |
| 6900-7100 | **Callout variants** (gummy, line, we1, brutal, soft, notyoutube) |
| 7100-7300 | **Border toggles**, misc toggles |
| 7300-7500 | **ntosx codeblock**, gradient halo animation |
| 7500-7700 | **Gradient canvas** variant (triple duplication) |
| 7700-7827 | **Final overrides** |

### Critical Issues

#### Embedded Base64 Fonts (~1MB)
Lines 775-885: 12+ fonts (Monaspace, Nightingale, Fraunces, General Sans, Martian Mono, Terminus, Aspekta, Getai) are base64-encoded directly in CSS. This is **50% of the total file size**.

#### `!important` Abuse (314 instances)
Key problem areas:
- Gradient background variables set with `!important` in every sub-theme variant
- `--background-modifier-border-alt` with `!important` in nearly every light theme
- Sub-theme overrides (origami, we3) use `!important` on virtually every property
- Final overrides (lines 7770+) escalate further

#### Broken/Buggy Selectors
- **Line 3631**: `.ul.ulu-con-accent-bg-d.theme-dark.ulu-pink` — broken selector, `.ul.` prefix is invalid
- **Line 3601**: `.ulu-cono-accent-bg-d.theme-dark.ulu-blue` — typo, should be `ulu-accent-bg-lowcon-d`
- **Line 3242**: `--pill-border:` — empty declaration, CSS syntax error
- **Line 4846**: `background: var` — truncated `var()` reference

#### External URL Reference
**Line 1724**: `--test-image: url("https://i.pinimg.com/...")` — test/debug variable left in production.

### Color Scheme Architecture

- **2 base themes**: `.theme-light`, `.theme-dark`
- **~22 dark sub-themes**: spotlight, soft, arch, shark, min, control, blueprint, end, dracula, lyt, lord, gradient, amoled, solarized, gruvbox, lobstertime, obplus, etc.
- **~15 light sub-themes**: soft, arch, shark, min, light, creation, solarized, gruvbox, etc.
- **8 accent colors**: red, green, orange, yellow, cyan, blue, purple, pink
- **4 contrast modes**: normal, low contrast, super contrast, accent-tinted

**Core problem**: Every variant block copies the full color palette (~20-30 variables). No token abstraction layer — colors are hardcoded hex values. The combinatorial explosion creates ~100+ variant blocks.

### Style Settings Integration

36 user-configurable settings in a single 774-line YAML block. Typos present: "danvas styling" (line 353), "cnvas styling" (line 413).

### Redundant/Duplicate Code

| Issue | Lines Wasted |
|---|---|
| Canvas gradients triplicated (3 identical copies) | ~600 |
| Callout variants redeclare all 12 types each | ~400+ |
| Accent-tinted background blocks (32 blocks, same pattern) | ~300 |
| Dead `.we1` code (not exposed in @settings) | ~285 |
| Duplicate `.nav-folder-title` declarations | ~10 |
| Duplicate `#cMenuToolbarModalBar.top` rule | ~10 |

### Fragile Selectors

**High-risk (will break):**
- 16 `aria-label` selectors: `div[aria-label="Settings"]`, `div[aria-label="New note"]`, etc.
- Svelte-generated classes: `.svelte-1vwr9dd`, `.svelte-pcimu8`, `.svelte-egt0yd`
- Plugin-specific: `.loom-*`, `.kanban-plugin__*`, `#cMenuToolbarModalBar`

**Medium-risk:**
- `.cm-s-obsidian`, `.HyperMD-*` (CodeMirror internals)
- `.mod-root`, `.mod-vertical`, `.mod-left-split`

### Architecture Assessment

The "three themes in a trench-coat" description is accurate:

1. **Layer 1** (lines 1-1738): Solid base theme with well-commented variables. Strongest part.
2. **Layer 2** (lines 1738-3650): Color variant system. Functional but massive duplication.
3. **Layer 3** (lines 3650-7827): Merged origami/gummy/working-edits. Specificity chaos.

---

## Architecture Proposal

### 1. Build System: SCSS (Dart Sass)

| Criterion | SCSS | PostCSS | Native CSS |
|---|---|---|---|
| Mixins for color palettes | Native `@mixin` | Requires plugin | Not available |
| Loops for 40+ color variants | `@each`, `@for` | Not available | Not available |
| Color math (lighten/darken/mix) | Built-in | Requires plugin | Not available |
| Maps for token systems | Native `map` type | Not available | Not available |
| Obsidian theme ecosystem | Most themes use SCSS | Some usage | Emerging |

**Verdict:** SCSS wins because the core problems — generating 40+ color variants from seeds, eliminating palette duplication, managing tokens — are exactly what SCSS was designed for.

#### package.json

```json
{
  "name": "ultra-potato-theme",
  "version": "0.1.0",
  "description": "Ultra Potato theme for Obsidian",
  "scripts": {
    "build": "sass src/main.scss theme.css --style=expanded --no-source-map",
    "build:min": "sass src/main.scss theme.css --style=compressed --no-source-map",
    "watch": "sass --watch src/main.scss:theme.css --style=expanded --no-source-map",
    "dev": "npm run watch",
    "lint": "stylelint 'src/**/*.scss'",
    "lint:fix": "stylelint 'src/**/*.scss' --fix",
    "clean": "rm -f theme.css",
    "preflight": "npm run lint && npm run build"
  },
  "devDependencies": {
    "sass": "^1.77.0",
    "stylelint": "^16.6.0",
    "stylelint-config-standard-scss": "^13.1.0",
    "stylelint-order": "^6.0.0"
  }
}
```

#### .stylelintrc.json

```json
{
  "extends": ["stylelint-config-standard-scss"],
  "plugins": ["stylelint-order"],
  "rules": {
    "declaration-no-important": true,
    "selector-max-specificity": "0,4,2",
    "max-nesting-depth": 4,
    "order/properties-alphabetical-order": true,
    "scss/no-global-function-names": null,
    "no-descending-specificity": null,
    "selector-class-pattern": null
  }
}
```

### 2. Directory Structure

```
ultra-potato/
├── manifest.json
├── theme.css                    # Build output (Obsidian loads this)
├── package.json
├── .stylelintrc.json
├── .gitignore                   # node_modules/, .sass-cache/
├── CHANGELOG.md
│
├── fonts/                       # Extracted font files
│   ├── eb-garamond-regular.woff2
│   ├── ibm-plex-mono-regular.woff2
│   ├── inter-regular.woff2
│   ├── inter-bold.woff2
│   └── ...
│
└── src/
    ├── main.scss                # Master entry point
    │
    ├── _settings.scss           # @settings YAML block
    │
    ├── tokens/
    │   ├── _index.scss
    │   ├── _colors.scss         # Base color palette maps
    │   ├── _typography.scss     # Font stacks, sizes, weights
    │   ├── _spacing.scss        # Spacing scale, radius, borders
    │   ├── _elevation.scss      # Shadows, z-index
    │   └── _motion.scss         # Transitions, animation timing
    │
    ├── foundations/
    │   ├── _index.scss
    │   ├── _reset.scss          # Obsidian overrides
    │   ├── _custom-properties.scss  # Root-level CSS variables
    │   ├── _font-faces.scss     # @font-face declarations (url() to fonts/)
    │   └── _keyframes.scss      # All 12 @keyframes consolidated
    │
    ├── color-schemes/
    │   ├── _index.scss
    │   ├── _generator.scss      # Mixin: seed -> full palette
    │   ├── _base-dark.scss
    │   ├── _base-light.scss
    │   ├── _catppuccin.scss
    │   ├── _nord.scss
    │   ├── _gruvbox.scss
    │   ├── _dracula.scss
    │   ├── _solarized.scss
    │   └── _custom.scss
    │
    ├── components/
    │   ├── _index.scss
    │   ├── _workspace.scss      # Workspace chrome, tabs, sidebar
    │   ├── _editor.scss         # CodeMirror / editor pane
    │   ├── _preview.scss        # Reading view
    │   ├── _headings.scss
    │   ├── _links.scss
    │   ├── _lists.scss
    │   ├── _callouts.scss
    │   ├── _tables.scss
    │   ├── _codeblocks.scss
    │   ├── _canvas.scss         # Deduplicated canvas gradients
    │   ├── _modals.scss
    │   ├── _status-bar.scss
    │   ├── _search.scss
    │   ├── _file-explorer.scss
    │   ├── _graph.scss
    │   └── _tags.scss
    │
    ├── plugins/
    │   ├── _index.scss
    │   ├── _dataview.scss
    │   ├── _kanban.scss
    │   ├── _calendar.scss
    │   ├── _tasks.scss
    │   ├── _excalidraw.scss
    │   └── _style-settings.scss
    │
    └── utilities/
        ├── _index.scss
        ├── _mixins.scss
        ├── _functions.scss
        └── _accessibility.scss
```

#### main.scss Entry Point

```scss
// 1. Style Settings (must be comment block at top)
@use 'settings';

// 2. Design tokens (SCSS-only, no CSS output)
@use 'tokens';

// 3. Foundations (reset, custom properties, fonts, keyframes)
@use 'foundations';

// 4. Color schemes (generated from seed palettes)
@use 'color-schemes';

// 5. Components
@use 'components';

// 6. Plugin compatibility (isolated overrides)
@use 'plugins';

// 7. Utilities
@use 'utilities';
```

### 3. CSS Custom Property Strategy

Three-tier token system: **Primitive -> Semantic -> Component**.

```
Tier 1 - Primitives (raw values, never used directly):
  --ul-red-500, --ul-blue-300, --ul-gray-800

Tier 2 - Semantic (role-based, theme-switchable):
  --ul-color-primary, --ul-color-surface, --ul-color-on-surface
  --ul-color-accent, --ul-color-muted, --ul-color-error

Tier 3 - Component (specific UI tokens):
  --ul-heading-color, --ul-link-color, --ul-sidebar-bg
  --ul-callout-bg, --ul-canvas-edge-color
```

**Prefix:** `--ul-` (Ultra Lobster legacy, kept for continuity) — consistent, short, avoids collisions with Obsidian's variables.

### 4. Color Scheme Generator

The core innovation that eliminates ~4,000 lines of copy-pasted palettes.

#### Generator Mixin

```scss
// color-schemes/_generator.scss

@mixin generate-scheme($name, $seeds) {
  $bg:      map-get($seeds, 'bg')      or #1e1e2e;
  $surface: map-get($seeds, 'surface')  or lighten($bg, 5%);
  $text:    map-get($seeds, 'text')     or #cdd6f4;
  $accent:  map-get($seeds, 'accent')   or #89b4fa;
  $red:     map-get($seeds, 'red')      or #f38ba8;
  $green:   map-get($seeds, 'green')    or #a6e3a1;
  $yellow:  map-get($seeds, 'yellow')   or #f9e2af;
  $blue:    map-get($seeds, 'blue')     or $accent;

  // Derive full palette algorithmically
  $text-muted: mix($text, $bg, 65%);
  $text-faint: mix($text, $bg, 40%);
  $bg-secondary: if(lightness($bg) > 50%, darken($bg, 4%), lighten($bg, 4%));
  $border: if(lightness($bg) > 50%, darken($bg, 12%), lighten($bg, 12%));
  $hover: if(lightness($bg) > 50%, darken($bg, 6%), lighten($bg, 6%));

  body.theme-#{$name} &,
  body[data-ul-scheme="#{$name}"] & {
    --ul-bg-primary: #{$bg};
    --ul-bg-secondary: #{$bg-secondary};
    --ul-bg-surface: #{$surface};
    --ul-text-normal: #{$text};
    --ul-text-muted: #{$text-muted};
    --ul-text-faint: #{$text-faint};
    --ul-accent-primary: #{$accent};
    --ul-border-normal: #{$border};
    --ul-color-red: #{$red};
    --ul-color-green: #{$green};
    --ul-color-yellow: #{$yellow};
    --ul-color-blue: #{$blue};
    --ul-color-orange: #{mix($red, $yellow, 50%)};
    --ul-color-cyan: #{mix($green, $blue, 50%)};
    --ul-color-purple: #{mix($red, $blue, 40%)};
    --ul-color-pink: #{lighten($red, 10%)};

    // Bridge to Obsidian's expected variables
    --background-primary: var(--ul-bg-primary);
    --background-secondary: var(--ul-bg-secondary);
    --text-normal: var(--ul-text-normal);
    --text-muted: var(--ul-text-muted);
    --interactive-accent: var(--ul-accent-primary);
  }
}
```

#### Sub-Theme Definitions (Dramatically Simplified)

```scss
// color-schemes/_catppuccin.scss
@use 'generator' as *;

:root {
  @include generate-scheme('catppuccin-mocha', (
    'bg':      #1e1e2e,
    'surface': #313244,
    'text':    #cdd6f4,
    'accent':  #89b4fa,
    'red':     #f38ba8,
    'green':   #a6e3a1,
    'yellow':  #f9e2af,
  ));
}
```

**Impact:** Each sub-theme: ~80-120 lines -> **7-8 lines**. 40 schemes = ~320 lines instead of ~4,000. **92% reduction.**

### 5. Style Settings Organization

SCSS string interpolation assembles modular sections into the single comment block Style Settings requires:

```scss
// src/_settings.scss
$settings-header: "
name: Ultra Potato
id: ultra-potato
settings:
";

$settings-color-scheme: "
    -
        id: up-color-scheme
        title: Color Scheme
        type: class-select
        allowEmpty: false
        default: catppuccin-mocha
        options:
            -
                label: Catppuccin Mocha
                value: theme-catppuccin-mocha
";

$settings-typography: "
    -
        id: up-typography
        title: Typography
        type: heading
        level: 1
        collapsed: true
";

/* @settings
#{$settings-header}#{$settings-color-scheme}#{$settings-typography}
*/
```

### 6. Font Strategy

1. **Extract all base64 fonts to WOFF2 files** in `fonts/` directory
2. **Audit actual usage** — cross-reference font-family declarations
3. **Keep only actively used fonts** (likely: Inter, EB Garamond, IBM Plex Mono)
4. **Use `url()` paths** instead of base64 embedding
5. **Add `font-display: swap`** to prevent FOIT

**Impact:** ~1MB removed from theme.css.

### 7. Plugin Compatibility Layer

- Move all plugin selectors to isolated files in `plugins/`
- Document plugin version last tested, fragile selectors, and visual targets
- Replace `aria-label` selectors with class-based alternatives where possible
- Remove Svelte-generated class selectors (`.svelte-*`)

### 8. Development Workflow

```bash
npm run dev          # Sass watcher (auto-compiles on save)
                     # Obsidian hot-reloads theme.css automatically

npm run lint         # Check for issues
npm run lint:fix     # Auto-fix
npm run preflight    # Lint + build (pre-commit)
```

---

## Migration Plan

### Phase 0: Preparation (Quick Wins)
1. Delete `obsidian.css` (confirmed dead code)
2. Remove external Pinterest test URL (line 1724)
3. Fix 4 broken selectors (lines 3242, 3601, 3631, 4846)
4. Remove dead `.we1` code (~285 lines)
5. Commit: "chore: clean up dead code and broken selectors"

### Phase 1: Extract Foundations
1. Extract 17 `@font-face` declarations -> `foundations/_font-faces.scss`
2. Decode base64 fonts to `fonts/` directory
3. Extract 12 `@keyframes` -> `foundations/_keyframes.scss`
4. Extract `@settings` YAML -> `src/_settings.scss`
5. **Validate:** Build output should match original (minus dead code)

### Phase 2: Extract Token System
1. Audit all 606 custom properties, categorize into tiers
2. Create token files
3. Create CSS custom property declarations
4. **Validate:** Visual regression testing with representative notes

### Phase 3: Refactor Color Schemes
1. Build the generator mixin
2. Convert default dark theme first, diff output
3. Convert 2-3 more schemes to validate
4. Convert remaining ~37 schemes (mechanical)
5. **Validate:** Toggle each scheme in Style Settings, verify colors

### Phase 4: Extract Components
1. Start with self-contained: canvas (deduplicate!), callouts, tables
2. Move to editor and preview components
3. Extract workspace chrome last
4. **Validate:** Rebuild and compare after each extraction

### Phase 5: Isolate Plugin Overrides
1. Move all plugin selectors to `plugins/`
2. Replace `aria-label` selectors where possible
3. Document remaining fragile selectors

### Phase 6: Eliminate `!important`
1. Fix specificity structurally via selector order and nesting
2. Remove `!important` in batches, testing each
3. **Target:** Zero `!important` (or < 5 with documented justification)

### Phase 7: Polish
1. Add `prefers-reduced-motion` and `prefers-color-scheme` media queries
2. Review accessibility: focus indicators, contrast ratios
3. Minify for release: `npm run build:min`
4. Update manifest.json version

### Validation Strategy (All Phases)
- **Diff testing:** Compare compiled output after each phase
- **Visual spot checks:** Test vault with headings, code, callouts, tables, canvas, graph
- **Style Settings sweep:** Toggle every option after Phase 3
- **Community beta:** Share with 3-5 users before tagging v1.0.0

---

## Quick Wins

Can be done immediately in the current monolithic `theme.css`:

| # | Fix | Impact | Risk |
|---|---|---|---|
| 1 | Delete `obsidian.css` entirely | Remove confusion | Zero |
| 2 | Remove external test URL (line 1724) | Security/privacy | Zero |
| 3 | Fix 4 broken selectors | Eliminates dead/broken rules | Low |
| 4 | Remove dead `.we1` code block | ~285 lines removed | Zero |
| 5 | Deduplicate canvas gradients (triplicated) | ~600 lines removed | Low |
| 6 | Add `font-display: swap` to all `@font-face` | Prevents FOIT | Zero |
| 7 | Add `prefers-reduced-motion` wrapping keyframes | Accessibility | Zero |

**Estimated impact:** ~700-800 lines removed, improved accessibility, cleaner file.

---

## Expected Outcomes

| Metric | Current | After Refactor |
|---|---|---|
| File size | 1.96 MB | ~200-300 KB (+font files) |
| Lines of CSS | 7,827 | ~3,000-3,500 compiled |
| Source files | 1 | ~35 modular SCSS files |
| `!important` count | 314 | 0 (target) |
| Color scheme lines/variant | 80-120 | 7-8 seeds |
| Canvas gradient copies | 3 | 1 |
| Build time | N/A | ~1-2 seconds |
| Hot reload | Manual | Automatic |
