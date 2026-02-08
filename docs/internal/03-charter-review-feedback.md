# 02 — Charter Review Feedback

**Date:** 2026-02-08
**Reviewers:** 4-agent panel (Obsidian Theme Veteran, Design Systems Architect, Designer Who Codes, AI-Assisted Development Specialist)
**Reviewed:** `docs/internal/02-charter.md`, `CLAUDE.md`, `docs/internal/01-claude-refactor.md`

---

## Critical Issues (Must Fix Before Writing Code)

### 1. Font `url()` Won't Work for Marketplace Distribution

**Source:** Obsidian Veteran

Obsidian's community theme store downloads only `theme.css` + `manifest.json`. A `fonts/` subdirectory will not be included. Users who install via the theme browser will get broken fonts.

**How other themes handle it:**
- **Minimal** — system font stacks only. No custom fonts. Safest approach.
- **AnuPpuccin** — Google Fonts `@import url()`. Works but requires internet.
- **ITS Theme / Blue Topaz** — base64-encoded fonts inline. Bloats the file but guarantees they work.

**Options:**
1. System font stacks + Google Fonts for web-hosted options. Remove all custom fonts.
2. Keep 2-3 critical fonts as base64 (under 50KB each). Accept the size cost for identity-defining fonts.
3. Offer a "font pack" companion snippet users install manually.
4. Acknowledge `fonts/` directory is development-only and won't survive community distribution.

**Resolution needed:** Pick a font strategy and document the tradeoff explicitly in the charter.

---

### 2. Style Settings Selector Bug in the Generator

**Source:** Obsidian Veteran

The generator mixin proposal uses `body.theme-#{$name}` and `body[data-ul-scheme="#{$name}"]`, but Style Settings' `class-select` option adds a **body class** (e.g., `body.ulu-catppuccin-mocha`), not a data attribute. The selector pattern needs to match what Style Settings actually emits.

**Fix:** The generator selector should be `body.#{$class-value}` where the class value matches the `class-select` option value in the `@settings` YAML.

---

### 3. `--up-` vs `--ul-` Prefix Inconsistency

**Source:** All three reviewers flagged this

- `docs/internal/02-charter.md` says `--up-` (Ultra Potato)
- `01-claude-refactor.md` says `--ul-` (Ultra Lobster legacy)
- `CLAUDE.md` says `--ul-` in the token table

`--ul-` also reads as "unordered list" to anyone who's written HTML.

**Resolution:** Use `--up-` everywhere. Update all docs before any code is written.

---

## Architecture Adjustments

### 4. Collapse to 2 Token Tiers, Not 3

**Source:** Design Systems Architect

The proposed Primitive tier (`--up-red-500`, `--up-gray-800`) implies a full numbered scale, but the generator produces exactly one red, one green, one yellow, etc. The Primitive tier is phantom architecture — it exists in the naming convention but not in reality.

**Recommended:**
- **Tier 1 — Semantic tokens** (the palette from the generator): `--up-bg-primary`, `--up-text-normal`, `--up-color-red`, `--up-accent-primary`
- **Tier 2 — Component tokens** (only where a component deviates from the semantic default): `--up-callout-bg`, `--up-heading-color`

Skip numbered primitives entirely. If needed later, going from 2 tiers to 3 is additive (easy). Going from 3 to 2 means removing a layer and updating all references (hard).

---

### 5. Generator Mixin Needs Optional Overrides for ALL Derived Colors

**Source:** All three reviewers

`mix()` operates in RGB space (perceptually non-uniform). `lighten()`/`darken()` use HSL (can produce washed-out or muddy results). For ~20% of palettes, algorithmic derivation will produce "close but off" colors.

**Examples of where it breaks:**
- `mix($red, $yellow, 50%)` for orange → washed-out, shifts toward brown with some inputs
- `lighten($red, 10%)` for pink → neon or washed-out depending on input saturation
- `lighten(#1a1b2e, 4%)` for bg-secondary → loses chromatic tint, shifts toward gray
- `darken(#f38ba8, 20%)` → dull brownish-red, not a rich dark red

**Fix:** Accept all derived colors as optional seeds with algorithmic fallbacks:

```scss
$orange: map-get($seeds, 'orange') or mix($red, $yellow, 50%);
$cyan: map-get($seeds, 'cyan') or mix($green, $blue, 50%);
$purple: map-get($seeds, 'purple') or mix($red, $blue, 40%);
$pink: map-get($seeds, 'pink') or lighten($red, 10%);
$bg-secondary: map-get($seeds, 'bg-secondary') or $auto-bg-secondary;
$text-muted: map-get($seeds, 'text-muted') or mix($text, $bg, 65%);
```

Most schemes provide 7-8 seeds. Curated schemes (Catppuccin, Gruvbox, Dracula, Solarized) can provide 12-15 for precision.

**Also consider:** `color.scale()` over `lighten()`/`darken()` — scales toward white/black proportionally rather than adding absolute lightness. And `color.adjust()` gives direct HSL channel control.

---

### 6. Separate the Obsidian Bridge from the Generator

**Source:** Design Systems Architect

The generator mixin should output only `--up-*` tokens. A separate `foundations/_obsidian-bridge.scss` maps them to Obsidian's native variables (`--background-primary: var(--up-bg-primary)`).

**Why:** The generator is reusable. The bridge is Obsidian-specific. Coupling them means the generator can never be used outside Obsidian. Even if that's hypothetical now, it's architecturally clean and costs nothing.

---

### 7. Add YAML Validation to the Build Pipeline

**Source:** Obsidian Veteran

The SCSS string interpolation approach for `@settings` is creative but fragile. YAML is whitespace-sensitive. SCSS string concatenation doesn't validate YAML structure. A single indentation error produces broken Style Settings with zero error feedback — the settings panel just doesn't appear.

**Fix:** Add a `validate-settings` script to the preflight pipeline:
1. Extract the `/* @settings ... */` block from compiled `theme.css`
2. Run it through a YAML parser (js-yaml or similar)
3. Fail the build if YAML is invalid

**Alternative:** Keep `_settings.scss` as a hand-written YAML block (not assembled from SCSS parts). It changes infrequently and doesn't benefit from SCSS features.

---

### 8. Missing: Specificity Architecture Strategy

**Source:** Obsidian Veteran

The charter bans `!important` and caps specificity at `0,4,2`, but doesn't explain HOW to achieve this with 40+ color scheme classes, sub-theme overrides, light/dark mode splits, callout variants, and border toggles all fighting for precedence. The original theme uses `!important` 314 times precisely because the cascade gets out of control.

**Needed:** A specificity layering strategy document:
- Base styles: `0,1,0` to `0,2,0`
- Color scheme overrides: `0,2,0` (body class + target element)
- Sub-theme overrides: `0,3,0` (body class + body class + target)
- Component variants: `0,2,0` to `0,3,0`
- Plugin overrides: `:where()` wrapper to keep specificity at 0

---

## Design-First Gaps

### 9. "Design-First" Should Literally Be Principle #1

**Source:** Designer

The charter leads with "Clean, Simple, Effective" (an engineering principle). "Design-First" is principle #2. If design is truly the priority, reorder the principles so design leads.

---

### 10. No Visual Identity Defined Anywhere

**Source:** Designer

The charter says each color scheme "tells a story" — but none of the stories are told. There's no mood, no character description, no screenshots, no reference images. The migration plan has 7 engineering phases and zero design phases ("define the visual identity," "establish the design language").

**What would make it genuinely design-first:**
- A visual design brief: "Ultra Potato aims for [warm/cool/neutral], [minimal/expressive/playful], [tight/airy spacing]. It should feel like [metaphor]."
- Reference screenshots showing the theme in its default state, 2-3 color schemes, light and dark.
- Design principles stated before engineering principles.

---

### 11. "Where Do I Change X?" Quick-Reference

**Source:** Designer

Add a lookup table to CLAUDE.md:

| I want to change... | Edit this file |
|---|---|
| A color in a scheme | `src/color-schemes/_schemename.scss` |
| Font stack | `src/tokens/_typography.scss` |
| Callout appearance | `src/components/_callouts.scss` |
| Border radius / spacing | `src/tokens/_layout.scss` |
| Shadows | `src/tokens/_layout.scss` |
| A specific heading style | `src/components/_headings.scss` |

---

### 12. Algorithmic Color Generation Needs Visual Proof

**Source:** Designer

Designers don't trust `mix()` and `lighten()`. They need to see the output. Recommendations:
- Include a comment block in sample scheme files showing every generated variable and its computed hex value.
- Create a "Create Your Own Color Scheme" tutorial with step-by-step guidance.
- Provide a `_template.scss` scheme file with inline comments explaining each seed value's role visually, not technically. Not "bg: background primary" but "bg: the color of the empty page — this sets the overall mood."

---

## Minor Adjustments

### File Structure

- **Merge** `_spacing.scss` + `_elevation.scss` + `_motion.scss` → `_layout.scss` (too granular for 5-10 variables each)
- **Merge** `_reset.scss` + `_custom-properties.scss` → `_base.scss` (same concern)
- **Add** `_accent-colors.scss` as its own module (8 colors x light/dark x contrast modes is a distinct concern from color schemes)
- **Watch** `_workspace.scss` — if it grows past ~150 lines, split into `_tabs.scss`, `_sidebar.scss`, `_workspace-chrome.scss`
- **Watch** `utilities/` — risk of becoming a junk drawer. Keep it small or integrate into components.

### SCSS Conventions

- **Ban `@import`** — mandate `@use` and `@forward` only (the charter should state this explicitly)
- **Avoid deep Sass features** like `@extend`, `%placeholder` selectors, or complex module namespacing — creates SCSS lock-in
- **Keep SCSS magic confined to generators** — components should be as close to plain CSS as possible for future portability

### Style Settings

- **Cap at ~50 options total** — each new option multiplies the test matrix combinatorially
- **Fix typos** in current YAML: "danvas styling" (line 353), "cnvas styling" (line 413)

### Selectors

- **Add a selector risk-tier reference** to CLAUDE.md (stable / medium-risk / high-risk with examples)
- **Flag `.cm-s-obsidian`** as legacy CodeMirror 5 — could be removed by Obsidian
- **Require comments on fragile selectors:** (a) why no stable alternative exists, (b) Obsidian version tested, (c) what breaks visually if selector stops matching
- **Consider `:where()` wrapper** for plugin overrides to keep specificity at 0

### Testing

- **No automated visual regression** — for 40+ schemes x 2 modes x variants, manual testing is insufficient. Consider screenshot diffing (Percy, BackstopJS, or even pixelmatch) as a future addition.
- **Style Settings panel check** should be item #1 in the manual testing checklist.

### Developer Experience

- **pnpm friction** — most designers who code know npm, not pnpm. Not a dealbreaker but worth acknowledging.
- **Symlink setup** — a `pnpm run setup --vault ~/my-vault` script would reduce friction vs raw `ln -s` commands.
- **Stylelint as gatekeeper** — acknowledge in charter: "If lint blocks you, ask — the constraint exists for a reason, and we'll either fix the approach or adjust the rule."

---

## Reviewer Verdicts Summary

| Reviewer | Overall Assessment |
|---|---|
| **Obsidian Veteran** | Architecture is directionally correct. Two critical issues: font distribution strategy and Style Settings selector bug. Needs specificity strategy doc. |
| **Design Systems Architect** | Fundamentally sound. Over-engineered in spots (3 token tiers, granular files). Generator is the right core abstraction but needs escape hatches. |
| **Designer Who Codes** | Charter invites designers in; supporting docs push them back out. Engineering-heavy project needs visual artifacts, design brief, and a guided first-contribution path. |
| **AI Development Specialist** | (Review completed but not delivered before team shutdown — findings incorporated where overlapping with other reviewers.) |
