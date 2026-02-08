# 05 — Naming Convention Decision

**Date:** 2026-02-08
**Decision:** Keep `bg-*` / `text-*` naming convention with `--up-` prefix
**Alternatives considered:** shadcn/ui `{role}` / `{role}-foreground` pairing

---

## Decision

Use `--up-bg-*` / `--up-text-*` naming that directly mirrors Obsidian's native `--background-*` / `--text-*` convention.

## Rationale

1. **Self-documenting bridge** — `--background-primary: var(--up-bg-primary)` is immediately obvious. No translation needed.
2. **3 text tiers** — Obsidian needs normal/muted/faint. `text-normal`, `text-muted`, `text-faint` express this naturally.
3. **No ambiguity** — `--up-bg-secondary` is obviously a background. shadcn's `--up-secondary` could be confused with a button color.
4. **Obsidian ecosystem familiarity** — anyone who has themed Obsidian already thinks in `background-*` / `text-*` terms.
5. **8 semantic colors stay simple** — `--up-color-red` through `--up-color-pink` with no forced `-foreground` pairing.

## Why Not shadcn style tokens?

- `primary`/`secondary` mean action buttons in shadcn but background surfaces in Obsidian — guaranteed confusion.
- Only ~40% of tokens mapped cleanly; the rest needed Obsidian-specific extensions that diluted the consistency benefit.
- `foreground` is too generic for a 3-tier text system.

## What We Borrowed from shadcn

Explicit surface/text pairing for **component tokens** (Tier 2) where it makes sense:
- `--up-card-bg` / `--up-card-text`
- `--up-callout-bg` / `--up-callout-text`
- `--up-code-bg` / `--up-code-text`

This pairing is used only at the component level, not as a system-wide convention.

## Final Token List

### Tier 1 — Semantic Tokens (set per color scheme, output by generator)

```
Surfaces:
  --up-bg-primary         # Editor/page background
  --up-bg-secondary       # Sidebar, status bar, ribbon, tab bar
  --up-bg-tertiary        # Nested surfaces, alt backgrounds
  --up-surface            # Cards, elevated elements
  --up-hover              # Hover state overlay

Text:
  --up-text-normal        # Default body text
  --up-text-muted         # Secondary text, metadata
  --up-text-faint         # Placeholders, collapsed indicators
  --up-text-on-accent     # Text on accent-colored backgrounds

Accent:
  --up-accent             # Primary interactive color
  --up-accent-hover       # Accent hover state
  --up-accent-muted       # Low-opacity accent for highlights

Borders:
  --up-border             # Default border
  --up-border-hover       # Border on hover/focus

Semantic colors:
  --up-color-red
  --up-color-orange
  --up-color-yellow
  --up-color-green
  --up-color-cyan
  --up-color-blue
  --up-color-purple
  --up-color-pink

Utility:
  --up-shadow             # Shadow color/opacity
```

### Tier 2 — Component Tokens (added on demand in component SCSS files)

```
  --up-card-bg / --up-card-text
  --up-callout-bg / --up-callout-text
  --up-code-bg / --up-code-text
  --up-heading-color
  --up-link-color
  --up-tag-color
  --up-nav-bg
  --up-tab-active-bg
  --up-canvas-bg / --up-canvas-dot
  --up-graph-node / --up-graph-line / --up-graph-text
```

### Obsidian Bridge (in foundations/_obsidian-bridge.scss)

```scss
body {
  --background-primary:     var(--up-bg-primary);
  --background-primary-alt: var(--up-bg-tertiary);
  --background-secondary:   var(--up-bg-secondary);
  --text-normal:            var(--up-text-normal);
  --text-muted:             var(--up-text-muted);
  --text-faint:             var(--up-text-faint);
  --text-on-accent:         var(--up-text-on-accent);
  --interactive-accent:     var(--up-accent);
  --interactive-accent-hover: var(--up-accent-hover);
  --background-modifier-border: var(--up-border);
  --background-modifier-border-hover: var(--up-border-hover);
}
```
