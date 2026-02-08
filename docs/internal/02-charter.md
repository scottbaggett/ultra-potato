# Ultra Potato — Project Charter

## Mission

Build the most maintainable, beautiful, and AI-friendly Obsidian theme in the ecosystem. Ultra Potato is not just a theme — it is a reference implementation for how modern Obsidian themes should be built.

---

## Principles

### 1. Clean, Simple, Effective

Every line of code must earn its place. Complexity is a cost, not a feature.

- **One way to do things.** If there are two ways to achieve a result, pick one and use it everywhere. Consistency beats cleverness.
- **No dead code.** If it's commented out, it's deleted. If it's unreachable, it's deleted. Version control is the archive.
- **No specificity wars.** Structure the cascade correctly so `!important` is never needed. Zero is the target.
- **No duplication.** If a pattern appears twice, abstract it. The color scheme generator exists because 40 copy-pasted palettes is unacceptable.
- **Small files, clear names.** Every SCSS partial should do one thing. Its filename should tell you what's inside without opening it.
- **Flat over nested.** Prefer shallow selector nesting. Deep nesting creates specificity problems and makes code harder to trace.

### 2. Design-First

This is a design project that happens to be built with code. The aesthetic is the product.

- **Visual quality is non-negotiable.** Every change must be visually verified in Obsidian. If it looks wrong, the code is wrong — regardless of what the spec says.
- **Designers should feel at home.** The token system, file structure, and naming conventions are designed so a designer who codes can navigate, understand, and modify the theme without deep CSS expertise.
- **Tokens are the design language.** Colors, typography, spacing, and elevation are defined as tokens — not scattered hex values. Changing the look of the theme means changing tokens, not hunting through component files.
- **Color schemes tell a story.** Each scheme is a curated palette, not a random collection of values. 7-8 seed values define the entire mood. The generator handles the math.
- **Beauty in the details.** Transitions, shadows, radii, and spacing should feel intentional and cohesive. No jarring inconsistencies between components.
- **Inspire, then educate.** Someone discovering this theme should first think "this is beautiful" and then think "I could build something like this."

### 3. Agent-Friendly

This codebase is designed to be understood and modified by both humans and AI agents.

- **Self-documenting structure.** The directory layout, file names, and CLAUDE.md should give an agent enough context to make targeted changes without reading the entire codebase.
- **Predictable patterns.** Every color scheme file follows the same pattern. Every component file follows the same pattern. An agent that understands one understands all.
- **Atomic changes.** The modular architecture means most changes touch 1-2 files. Adding a color scheme is one file. Fixing a callout style is one file. This keeps agent context windows small and diffs reviewable.
- **Clear boundaries.** Tokens, foundations, color schemes, components, and plugins are separate concerns with explicit interfaces. An agent modifying callout styles should never need to touch the token system.
- **Documented constraints.** The CLAUDE.md spells out what is and isn't allowed. Agents should be able to work confidently within those guardrails without guessing.
- **Human-readable output.** The compiled `theme.css` uses expanded style (not minified) during development so humans and agents can inspect and debug the output.

---

## Working Agreements

### Development

1. **pnpm** is the package manager.
2. **SCSS (Dart Sass)** is the preprocessor. No PostCSS, no native nesting.
3. **Stylelint** enforces code quality. `pnpm run preflight` must pass before any commit.
4. **`theme.css` is a build artifact.** Never hand-edit it. All work happens in `src/`.
5. **`--up-` is the CSS variable prefix.** Forks should global-replace this with their own.

### Design Decisions

6. **Tokens are the source of truth.** If a value appears in a component, it should reference a token — not a raw value.
7. **Color schemes use the generator mixin.** No copy-pasting palettes. 7-8 seeds per scheme, the mixin does the rest.
8. **Style Settings options are curated, not exhaustive.** Expose meaningful choices that change the character of the theme. Don't expose every variable as a setting.

### Quality

9. **Zero `!important` target.** Exceptions require a comment explaining why and a plan to eliminate.
10. **No fragile selectors.** No `aria-label`, no `.svelte-*`, no deeply nested DOM path selectors. If a stable class doesn't exist, document the gap and use the most resilient alternative.
11. **Plugin overrides are isolated.** Each plugin gets its own file in `src/plugins/` with version documentation.
12. **Every change is visually verified.** Build it, open Obsidian, look at it. Toggle light/dark. Toggle a color scheme. This is the test suite.

### Collaboration

13. **CLAUDE.md is the agent onboarding doc.** Keep it current. If a convention changes, update CLAUDE.md first.
14. **docs/charter.md is the north star.** When in doubt about a decision, refer back to these principles.
15. **docs/claude-refactor.md is the technical blueprint.** It contains the audit findings and architecture details.
16. **Commits are meaningful.** Each commit should represent a coherent unit of work. Commit messages explain *why*, not *what*.

---

## Non-Goals

- **This is not a CSS framework.** It's a theme. Don't over-abstract.
- **This is not a plugin.** It styles Obsidian's existing UI. Don't add functionality via CSS hacks.
- **Backwards compatibility with Ultra Lobster is not required.** This is a fresh start built on a great foundation. Breaking changes from the original are expected and welcome.
- **Supporting every plugin is not a goal.** Support popular plugins that the maintainer uses. Others can be added on request.
