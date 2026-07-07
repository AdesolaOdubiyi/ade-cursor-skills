# Reference: Anti-AI-Slop Technique Checklist

Sourced from Anthropic's frontend-design research, Vercel's web design guidelines, and common pre-delivery checklists used by design-system generators. Use during mockup generation (step 1-3 of SKILL.md) and as a review pass before calling a visual direction done.

## Typography

- Avoid Inter/Roboto/Arial/system-font-only pairings — they're the most common default and read as generic.
- Pick a real pairing with contrast: a distinctive display/serif for headings + a clean sans for body, or extreme weight contrast (300 vs 700+) within one family.
- Use dramatic size jumps between hierarchy levels (headline vs. body should differ by 2.5-3x+, not 1.3x).
- Numbers that appear in columns or need to be scanned (prices, quantities, dates) get `font-variant-numeric: tabular-nums`.

## Color

- Commit to a cohesive system via CSS variables (or Tailwind theme tokens) — one dominant color + one sharp accent, not a timid even distribution across five hues.
- Never make a gradient the primary brand identity — gradients (especially purple-to-pink) are the single most recognizable AI-slop tell.
- Draw inspiration from a concrete reference (an IDE theme, a print medium, a specific era/subculture) rather than free-associating — concrete references produce cohesive results, free association produces generic ones.
- Contrast: 4.5:1 minimum for body text; prefer checking with APCA over WCAG2 where tooling allows, since it's a better perceptual match.

## Layout / chrome

- Boxed-card-with-shadow-and-rounded-corners is the default "AI dashboard" look. Alternatives that read as more designed: hairline dividers + generous whitespace, borderless sections separated by grid gap alone, or a strong border-only (no shadow, no radius) treatment.
- Every element should align to a grid or baseline — nothing "floating" without a reason.
- Respect safe areas and test at real breakpoints: 375 / 768 / 1024 / 1440.

## Motion

- Prefer CSS animations/transitions over JS for anything hoverable — GPU-accelerated properties only (`transform`, `opacity`), never animate `width`/`height`/`top`/`left` if a transform equivalent exists (exception: intentional bar-chart fills, which are fine to animate directly since they're infrequent and illustrate data).
- One well-executed sequence per screen (e.g. a staggered fade-up on load, ~40-50ms delay per item) beats scattered micro-interactions on every element.
- Always wrap non-essential animation in `@media (prefers-reduced-motion: reduce)` — set `animation: none` or `transition: none` inside it.
- Motion should communicate cause-and-effect (this click caused that change) or deliberate delight — never auto-play without a trigger.

## Accessibility

- Native semantic elements first: `<button>`, `<label>`, `<a>` before reaching for ARIA roles.
- Every interactive element needs a visible focus state — a browser default outline is acceptable only if it's not been globally suppressed elsewhere.
- Forms: labels associated via `<label for>` or wrapping, errors placed adjacent to the field they describe, submit buttons stay enabled (not disabled) until the request is actually in flight.
- No emoji as functional icons — use an SVG icon set (Lucide, Heroicons) so icons scale and recolor consistently.

## Red flags that mean "start over," not "iterate"

- The mockup would look at home as a generic SaaS landing page template with the logo swapped.
- The only differentiation from a default Tailwind/shadcn scaffold is the accent hue.
- Every card has the same shadow, same radius, same padding, and there's no other structural idea in the layout.
