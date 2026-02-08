# 04 — Color Pipeline Proposal: rampensau Integration

**Date:** 2026-02-08
**Context:** Charter feedback unanimously flagged SCSS color math (`mix()`, `lighten()`, `darken()`) as producing perceptually incorrect results. This proposal replaces SCSS color functions with [rampensau](https://github.com/meodai/rampensau) for all color generation, operating in OKLCH (perceptually uniform) color space.

---

## The Problem

SCSS color functions operate in RGB/HSL — perceptually non-uniform color spaces:
- `mix($red, $yellow, 50%)` produces muddy oranges, not vibrant ones
- `lighten()` / `darken()` can wash out or shift hue
- `lighten(#1a1b2e, 4%)` loses chromatic tint, shifts toward gray
- No way to do perceptual blending or controlled hue rotation

## The Solution

**Hybrid build pipeline:**
```
[Scheme JSON configs] → [Node script + rampensau] → [Generated SCSS partials] → [Sass build] → [theme.css]
```

1. Color schemes are defined as **JSON config files** (designer-friendly, readable, diffable)
2. A **Node script** uses rampensau + culori to generate full palettes in OKLCH space
3. Output is **SCSS partial files** containing maps of computed hex values
4. **SCSS consumes** the generated partials and handles everything else (nesting, components, Style Settings assembly)

The color math moves to a proper color library. SCSS never touches a color function.

---

## Build Commands

```json
{
  "scripts": {
    "colors": "node scripts/generate-palettes.js",
    "build": "pnpm run colors && sass src/main.scss theme.css --style=expanded --no-source-map",
    "dev": "concurrently \"chokidar 'src/color-schemes/configs/*.json' -c 'pnpm run colors'\" \"sass --watch src/main.scss:theme.css --style=expanded --no-source-map\"",
    "preflight": "pnpm run lint && pnpm run build"
  }
}
```

In dev mode, `chokidar` watches the JSON configs and re-runs palette generation when they change. Sass watches the SCSS files (including the generated partials) and recompiles.

---

## Scheme Config Format

Each color scheme is a JSON file in `src/color-schemes/configs/`.

### Curated Scheme (hand-picked colors)

For established palettes like Catppuccin, Dracula, Nord — use the official hex values. No generation needed.

```json
// src/color-schemes/configs/catppuccin-mocha.json
{
  "name": "catppuccin-mocha",
  "label": "Catppuccin Mocha",
  "mode": "dark",
  "type": "curated",
  "colors": {
    "bg":        "#1e1e2e",
    "surface":   "#313244",
    "text":      "#cdd6f4",
    "accent":    "#89b4fa",
    "red":       "#f38ba8",
    "orange":    "#fab387",
    "yellow":    "#f9e2af",
    "green":     "#a6e3a1",
    "cyan":      "#94e2d5",
    "blue":      "#89b4fa",
    "purple":    "#cba6f7",
    "pink":      "#f5c2e7"
  }
}
```

The script still derives secondary values (muted text, borders, hover states) from these seeds — but in OKLCH space, not HSL.

### Generated Scheme (rampensau-powered)

For original/custom schemes, define generation parameters. rampensau creates the palette.

```json
// src/color-schemes/configs/ultra-potato-dark.json
{
  "name": "ultra-potato-dark",
  "label": "Ultra Potato Dark",
  "mode": "dark",
  "type": "generated",
  "generation": {
    "hStart": 260,
    "hCycles": 1,
    "sRange": [0.5, 0.7],
    "lRange": [0.55, 0.75],
    "curveMethod": "lamé",
    "curveAccent": 0.4
  },
  "overrides": {
    "bg":      "#1a1a2e",
    "surface": "#252540",
    "text":    "#e0e0ec"
  }
}
```

The `generation` block tells rampensau how to create the 8 semantic colors (red through pink). The `overrides` block lets the designer hand-pick any value that the algorithm gets wrong. Overrides always win.

### Minimal Scheme (just seeds, derive everything)

For quick experimentation — provide the minimum and let the pipeline figure out the rest.

```json
// src/color-schemes/configs/midnight.json
{
  "name": "midnight",
  "label": "Midnight",
  "mode": "dark",
  "type": "minimal",
  "colors": {
    "bg":     "#0d1117",
    "accent": "#58a6ff",
    "text":   "#c9d1d9"
  }
}
```

The script derives surface, border, muted/faint text, hover states, and generates a full 8-color semantic palette harmonized with the accent hue — all in OKLCH.

---

## What the Script Generates

For each scheme config, the script outputs an SCSS partial:

```scss
// src/color-schemes/_generated/_catppuccin-mocha.scss (auto-generated, do not edit)
// Generated from: configs/catppuccin-mocha.json
// Generated at: 2026-02-08T18:00:00Z

$catppuccin-mocha: (
  // Seeds (from config)
  "bg":           #1e1e2e,
  "surface":      #313244,
  "text":         #cdd6f4,
  "accent":       #89b4fa,
  "red":          #f38ba8,
  "orange":       #fab387,
  "yellow":       #f9e2af,
  "green":        #a6e3a1,
  "cyan":         #94e2d5,
  "blue":         #89b4fa,
  "purple":       #cba6f7,
  "pink":         #f5c2e7,

  // Derived in OKLCH (auto-generated)
  "bg-secondary": #262637,
  "bg-tertiary":  #2e2e42,
  "text-muted":   #9399b2,
  "text-faint":   #6c7086,
  "border":       #45475a,
  "border-hover": #585b70,
  "hover":        #32324a,
  "accent-muted": rgba(137, 180, 250, 0.15),
  "accent-hover": #9dbffc,
  "shadow":       rgba(0, 0, 0, 0.25),
);
```

Each scheme becomes a simple SCSS map. No color math in SCSS at all — just lookups.

---

## How SCSS Consumes the Generated Palettes

```scss
// src/color-schemes/_index.scss
@use 'generated/catppuccin-mocha' as *;
@use 'generated/catppuccin-latte' as *;
@use 'generated/nord' as *;
// ... etc

// The applicator mixin reads from the map and emits CSS custom properties
@mixin apply-scheme($scheme-map, $class-name) {
  body.#{$class-name} {
    --up-bg-primary:    #{map-get($scheme-map, "bg")};
    --up-bg-secondary:  #{map-get($scheme-map, "bg-secondary")};
    --up-bg-surface:    #{map-get($scheme-map, "surface")};
    --up-text-normal:   #{map-get($scheme-map, "text")};
    --up-text-muted:    #{map-get($scheme-map, "text-muted")};
    --up-text-faint:    #{map-get($scheme-map, "text-faint")};
    --up-accent:        #{map-get($scheme-map, "accent")};
    --up-accent-muted:  #{map-get($scheme-map, "accent-muted")};
    --up-accent-hover:  #{map-get($scheme-map, "accent-hover")};
    --up-border:        #{map-get($scheme-map, "border")};
    --up-border-hover:  #{map-get($scheme-map, "border-hover")};
    --up-hover:         #{map-get($scheme-map, "hover")};
    --up-color-red:     #{map-get($scheme-map, "red")};
    --up-color-orange:  #{map-get($scheme-map, "orange")};
    --up-color-yellow:  #{map-get($scheme-map, "yellow")};
    --up-color-green:   #{map-get($scheme-map, "green")};
    --up-color-cyan:    #{map-get($scheme-map, "cyan")};
    --up-color-blue:    #{map-get($scheme-map, "blue")};
    --up-color-purple:  #{map-get($scheme-map, "purple")};
    --up-color-pink:    #{map-get($scheme-map, "pink")};
    --up-shadow:        #{map-get($scheme-map, "shadow")};
  }
}

// Apply each scheme
@include apply-scheme($catppuccin-mocha, "up-catppuccin-mocha");
@include apply-scheme($catppuccin-latte, "up-catppuccin-latte");
@include apply-scheme($nord, "up-nord");
```

The SCSS is dead simple — just map lookups. All the intelligence is in the Node script.

---

## Derivation Logic (in the Node script)

Using rampensau + culori for perceptually correct derivations:

```js
import { generateColorRamp, colorUtils } from 'rampensau';
import { oklch, formatHex, interpolate, wcagContrast } from 'culori';

function deriveFromBase(colors, mode) {
  const bg = oklch(colors.bg);
  const text = oklch(colors.text);
  const accent = oklch(colors.accent);
  const isDark = mode === 'dark';

  return {
    // Surface: slight lightness shift from bg, preserving hue + chroma
    'bg-secondary': shiftLightness(bg, isDark ? 0.03 : -0.03),
    'bg-tertiary':  shiftLightness(bg, isDark ? 0.06 : -0.06),

    // Text: perceptual blend between text and bg
    'text-muted': perceptualMix(text, bg, 0.35),
    'text-faint': perceptualMix(text, bg, 0.6),

    // Border: derived from bg with controlled contrast
    'border':       shiftLightness(bg, isDark ? 0.1 : -0.08),
    'border-hover': shiftLightness(bg, isDark ? 0.15 : -0.12),

    // Hover: subtle bg shift
    'hover': shiftLightness(bg, isDark ? 0.05 : -0.04),

    // Accent variants
    'accent-muted': withAlpha(accent, 0.15),
    'accent-hover': shiftLightness(accent, 0.06),

    // Shadow: mode-appropriate
    'shadow': isDark ? 'rgba(0, 0, 0, 0.25)' : 'rgba(0, 0, 0, 0.08)',
  };
}

function shiftLightness(color, amount) {
  // Shift in OKLCH space — preserves hue and chroma
  const c = oklch(color);
  c.l = Math.max(0, Math.min(1, c.l + amount));
  return formatHex(c);
}

function perceptualMix(color1, color2, ratio) {
  // Interpolate in OKLCH space — perceptually uniform blending
  const mix = interpolate([color1, color2], 'oklch');
  return formatHex(mix(ratio));
}
```

### Semantic Color Generation (for "generated" and "minimal" schemes)

```js
function generateSemanticColors(accentHue, mode) {
  const isDark = mode === 'dark';

  // Use rampensau to generate 8 harmonious hues from the accent
  const hues = colorUtils.colorHarmonies.analogous(accentHue);
  // Or generate a full spread:
  const colors = generateColorRamp({
    total: 8,
    hStart: 0,
    hCycles: 1,
    hueList: [0, 30, 60, 120, 180, 210, 270, 330], // red, orange, yellow, green, cyan, blue, purple, pink
    sRange: isDark ? [0.6, 0.7] : [0.55, 0.65],
    lRange: isDark ? [0.7, 0.8] : [0.45, 0.55],
    curveMethod: 'lamé',
    curveAccent: 0.3,
    transformFn: (color) => colorUtils.colorToCSS(color, 'oklch'),
  });

  return {
    red:    colors[0],
    orange: colors[1],
    yellow: colors[2],
    green:  colors[3],
    cyan:   colors[4],
    blue:   colors[5],
    purple: colors[6],
    pink:   colors[7],
  };
}
```

---

## Directory Structure (Updated)

```
ultra-potato/
├── scripts/
│   └── generate-palettes.js    # rampensau + culori palette generator
│
└── src/
    └── color-schemes/
        ├── configs/             # Designer-editable JSON scheme definitions
        │   ├── catppuccin-mocha.json
        │   ├── catppuccin-latte.json
        │   ├── dracula.json
        │   ├── nord.json
        │   ├── gruvbox-dark.json
        │   ├── ultra-potato-dark.json
        │   └── ...
        │
        ├── _generated/          # Auto-generated SCSS partials (gitignored? or committed?)
        │   ├── _catppuccin-mocha.scss
        │   ├── _catppuccin-latte.scss
        │   ├── _dracula.scss
        │   └── ...
        │
        └── _index.scss          # Imports all generated partials + apply-scheme mixin
```

---

## Dependencies

```json
{
  "devDependencies": {
    "rampensau": "^2.0.0",
    "culori": "^4.0.0",
    "chokidar-cli": "^3.0.0",
    "concurrently": "^8.0.0",
    "sass": "^1.77.0"
  }
}
```

- **rampensau** — palette generation, hue harmonies, curve-based ramps
- **culori** — OKLCH color manipulation, perceptual interpolation, hex conversion, contrast checking
- **chokidar-cli** — watches JSON configs for changes during dev
- **concurrently** — runs JSON watcher + Sass watcher in parallel

---

## Benefits Over Pure SCSS

| Concern | SCSS approach | rampensau + culori approach |
|---|---|---|
| Color blending | RGB space (perceptually wrong) | OKLCH space (perceptually correct) |
| Hue preservation | Loses tint on lighten/darken | Preserves hue + chroma |
| Designer workflow | Edit SCSS seed values, rebuild, check | Edit JSON config, auto-rebuild, check |
| Scheme format | SCSS maps (code) | JSON (data, designer-friendly, toolable) |
| Escape hatches | Override SCSS variables | Override any value in JSON `overrides` block |
| Generated palette inspection | Run build, inspect DevTools | Read the generated SCSS partial directly |
| Contrast validation | Manual | Can add `wcagContrast()` checks to the script |
| Future portability | Locked to SCSS | JSON configs portable to any build system |

---

## Open Questions

1. **Should generated SCSS partials be committed or gitignored?**
   - Committed: simpler CI, no Node dependency needed just to read the theme. Designers can inspect the output.
   - Gitignored: cleaner git history, single source of truth is the JSON config.
   - **Recommendation:** Commit them. Designers need to see the output without running the build.

2. **Should we add WCAG contrast validation to the script?**
   - culori provides `wcagContrast()`. The script could warn when text/bg combinations fail AA.
   - **Recommendation:** Yes, as warnings (not errors). Flag but don't block.

3. **Should the generated partials include a preview comment?**
   - Each generated file could include a comment block showing all hex values and their OKLCH coordinates for designer reference.
   - **Recommendation:** Yes. This directly addresses the designer reviewer's request for "show me the output."
