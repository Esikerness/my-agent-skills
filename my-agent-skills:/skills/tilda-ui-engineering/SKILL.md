---
name: tilda-ui-engineering
description: Review, debug, and extend custom Tilda Vibe Block HTML, CSS, and JavaScript with evidence-based responsive layouts, accessible interactions, safe global scripts, and verified visual behavior.
---

# Tilda UI Engineering

Use this skill when the user works with Tilda custom code, Vibe Blocks, page-level `<head>` scripts, responsive layouts, animations, hover/touch behavior, or visual bugs on a Tilda site.

## Core operating rule

Diagnose the user-visible behavior before changing code. Separate:

- the geometry and content of a section;
- the page-wide background or canvas;
- decorative effects such as glows, spotlights, shadows, and gradients;
- interaction logic and the boundary where it should work.

Never assume that fixing a section's height fixes an effect that must continue outside that section. Check which ancestor clips the effect (`overflow`, masking, stacking context, or a viewport boundary).

## Workflow

1. Read the complete supplied block and identify its insertion context: Tilda Vibe Block, page HTML, or global Head code.
2. Inspect the live page when a URL is available. Reproduce the exact scenario, including scrolling to the edge of the block, moving the pointer beyond it, resizing, and testing a narrow viewport.
3. Record the expected and actual behavior. Inspect computed geometry and styles for the relevant element, its parent, `body`, and `#allrecords` when useful.
4. Form a specific root-cause hypothesis and test it before implementing. Do not treat a matching class, selector, or static screenshot as proof that runtime behavior works.
5. Preserve user-provided copy, links, IDs, class names, visual hierarchy, and composition unless the user explicitly requests a redesign. Make the smallest change that fixes the cause.
6. Re-test the original failing scenario after the change and check adjacent states for regressions.

## Tilda-specific checks

  - Tilda Vibe Blocks commonly render inside `#allrecords` and page wrappers that can have their own background, height, position, and overflow. Inspect the actual rendered ancestors before changing section dimensions.
- `width: 100vw` inside a page wrapper can create a horizontal scrollbar because the viewport width may include the scrollbar. Prefer `width: 100%`, `max-width: 100%`, and `box-sizing: border-box` unless true viewport bleed is required.
- If a visual effect must span the viewport or continue below a section, use a page-level or fixed layer with an intentional stacking order. Do not leave it as a pseudo-element inside a short section with `overflow: hidden`.
- Keep page background and spotlight coordinates separate. A section-local `::before` cannot follow the pointer in an area outside that section.
- For a global Head script, initialize after `DOMContentLoaded` or when the body exists. Guard optional APIs such as `IntersectionObserver`, `matchMedia`, and `requestAnimationFrame`.
- MutationObserver handlers must inspect the added node itself (`node.tagName`), not an undefined parent. Skip `SCRIPT`, `STYLE`, `SVG`, `IFRAME`, and other non-content tags before walking text or attaching behavior.
- Make global scripts idempotent where possible: avoid duplicate observers, duplicate layers, and duplicate listeners if Tilda re-renders or previews a block more than once.
- When an image must be replaceable through the Vibe editor, render a real `<img>` element rather than a CSS background, gradient, or a decorative `div`. Give it a valid non-empty placeholder `src`, meaningful `alt`, and, for repeated slots, a stable marker such as `data-image-slot`. An empty `src` or a background-only visual can prevent Tilda from exposing the image-upload control.
- Preserve the editable image's aspect ratio with its surrounding frame and use `object-fit` on the `<img>` itself. Do not make an image a purely presentational substitute for a text-bearing control.

## Responsive implementation

Use mobile-first defaults and progressive enhancement for fine pointers.

- No horizontal scrolling at 320–375 px widths; use shrinkable grid children (`minmax(0, 1fr)`), `min-width: 0`, and `overflow-wrap: anywhere` for long user-visible tokens.
- Keep body text at least 16 px on mobile unless the visual role clearly permits smaller text; keep interactive targets at least 44 px high and separated by at least 8 px.
- Do not rely on hover for essential behavior. Touch layouts must remain complete without pointer events.
- Use `@media (hover: hover) and (pointer: fine)` or a matching runtime check for cursor-only effects such as magnetic movement and cursor spotlights.
- Preserve composition when extending a background: change the canvas/background layer, not the flex alignment or content height, unless the user asks for a layout change.
- Test at small phone, large phone, tablet, desktop, and landscape widths when the task concerns responsive behavior.

## Motion and accessibility

- Motion must use `transform` and `opacity` where possible and must not create layout shift.
- Animate only the elements that communicate hierarchy or response; do not animate every child by default.
- Always implement `prefers-reduced-motion: reduce` so content is immediately visible and decorative animation is removed or simplified.
- Add visible `:focus-visible` states to links, buttons, and custom controls. Decorative SVGs beside visible labels use `aria-hidden="true"`; standalone controls need an accessible name.
- Make focus and interactive content remain visible when fixed layers, sticky elements, or overlays are present.

## Verification standard

Before reporting completion, verify proportionally to the change:

- syntax and structural checks for HTML/CSS/JS;
- no unintended `100vw` overflow or clipped focus state;
- the exact original failing interaction;
- mobile behavior without hover;
- reduced-motion behavior;
- keyboard focus and link targets;
- global-script behavior after dynamically added Tilda content.
- for every expected Vibe-editable visual, the source contains one real `<img>` with a non-empty `src`, usable `alt`, and the intended image-slot marker; confirm the editor exposes the upload control when live access is available.

Clearly distinguish live/browser verification from static code inspection. If a viewport or interaction could not be tested, say so instead of claiming full verification.

## Response format

Report briefly:

1. root cause;
2. minimal change made;
3. behavior intentionally preserved;
4. checks actually performed;
5. any remaining manual check for the user.
