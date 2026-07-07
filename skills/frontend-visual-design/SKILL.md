---
name: frontend-visual-design
description: Design and implement sleek, human-feeling UI that doesn't read as AI-generated. Use when redesigning or building the visual layer of a UI (palette, typography, layout chrome, motion) — as opposed to `ux-product`, which handles interaction/product thinking. Trigger on "redesign the UI", "make this look less AI", "mock up a design direction", "pick a visual style".
---

# Frontend Visual Design

Pairs with `ux-product`: that skill decides what the interface should *do* (user goals, hierarchy, flow); this skill decides how it should *look* (palette, type, chrome, motion) so it reads as designed, not generated. Run `ux-product` first for anything with new interaction surface.

## Why this exists

Claude defaults to generic choices — Inter at every weight, purple/blue gradients, boxed-card-with-shadow chrome, timid or absent motion — because those dominate training data. That's the "AI slop" look. Undoing it takes a deliberate process, not just a better prompt. See [REFERENCE.md](REFERENCE.md) for the full technique checklist (typography, color, motion, accessibility, anti-patterns).

## Process — never skip straight to real code

1. **Generate 2-3 distinct static HTML mockups** of one real screen (Tailwind CDN + Google Fonts, no build step, lives outside the repo). Each direction should differ in more than color — different type pairing, different chrome (boxed vs. hairline-divider vs. borderless), different personality. Open all of them so the user compares side by side instead of describing preferences in the abstract.
2. **Get a direction pick.** Expect at least one direction to flop — that's a feature, not a failure; it calibrates what "wrong" looks like for this project.
3. **Mock up 2-3 more real screens** in the winning direction, using the project's actual data/fields/copy, not placeholder lorem-ipsum. Cover a form-heavy screen, a data/table screen, and a list screen if the app has them — this is what proves the direction survives contact with real content instead of looking good only on one hero screen.
4. **Get confirmation the direction holds up** before touching the real app.
5. **Implement primitives first.** Rewrite shared design tokens (CSS variables) and shared components (Button, Input, Card, nav) so the new look cascades through every page automatically. Only after that, do page-specific touch-ups for content that doesn't fit the shared primitives (custom charts, stat tiles, tables).
6. **Verify nothing broke**: typecheck, run the test suite, load every route. A visual-only pass should not change business logic, so any test failure is a real regression.

## Non-negotiable defaults

Apply these regardless of which visual direction wins — they're accessibility/quality floor, not style choices:

- `tabular-nums` on every money/quantity figure
- Visible `:focus-visible` ring in the accent color, not the browser default
- `prefers-reduced-motion` respected on any animation
- 150–300ms transitions on hover/focus states
- One dominant accent color used deliberately — never a gradient as the primary identity
- Two-tier status coloring (ok/warn) reads calmer than three-tier traffic lights unless the domain genuinely needs three states

## After implementation

Run `frontend-pre-merge-reviewer` as the merge gate — it checks the code matches these decisions and framework-specific failure modes.
