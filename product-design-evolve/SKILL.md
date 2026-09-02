---
name: product-design-evolve
description: Evolve an existing product's visual language into a more distinctive design system while retaining its recognizable colours, fonts, and product identity. Use when the user wants a design-system or distinctive redesign of an existing UI; not for greenfield branding.
---

# Evolve

Keep the **invariants**. Rewrite the **elastic**. Leave **debt** behind.

## Classify the DNA

Inspect the running product and its implementation. Capture screenshots at representative desktop and mobile **viewports**, then trace each visible choice to tokens, styles, assets, and components.

Classify every visible choice:

- **Invariant** — primary brand colours, typefaces, logos, product meaning, accessibility, established interaction expectations. Preserve these unless the user changes the brief.
- **Elastic** — colour roles and proportions, type scale, spacing rhythm, grid, density, shape, borders, elevation, imagery, motion, composition. This is the creative material.
- **Debt** — accidental inconsistency, duplicated values, one-off components, inaccessible combinations, broken responsiveness, styling with no product rationale.

A token name is not evidence that a value is visibly important. Compare implementation with the rendered product.

DNA is classified when every visible choice on the representative viewports is invariant, elastic, or debt, each with evidence from both the render and its source.

## Write the thesis

Derive direction from the product's purpose, content, and strongest existing cues. When the brief leaves room, explore a few interpretations that change composition or interaction character, not decoration.

Commit to one **thesis**:

- the feeling the interface should create
- which invariants anchor it to the product
- the one or two deliberate tensions that make it memorable
- what the system will refuse

Whimsy comes from a repeatable idea — unusual rhythm, expressive type scale, tactile interaction, illustration, controlled asymmetry.

The thesis is set when one is chosen and its anchors and tensions are named. If alternatives were explored, they are recorded.

## Build a vertical slice

Implement the smallest representative surface that exercises the system under real content and interaction. Prefer an important existing screen that contains typography, navigation, controls, content, and responsive behavior.

From that **slice**, establish the project's natural source of truth for:

- semantic colour roles and accessible states
- typography roles using the existing font families
- spacing, grid, breakpoints, radii, borders, and elevation
- core controls and content primitives
- iconography, imagery, motion, and reduced-motion behavior where they serve the thesis

Reuse the project's architecture and component conventions. Introduce a token or primitive only when it represents a recurring decision. New work lands in production UI. No parallel component library, speculative variants, or disconnected showcase.

The slice is done when every system decision it exercises has a source-of-truth location, and the slice uses those locations.

## Subtract

Review the rendered slice for **slop** and remove anything without a named job: extra cards, pills, ornamental copy, gradients, glows, uniform rounding, fake metrics, needless icons, motion on every element. Prefer hierarchy, spacing, proportion, imagery, and a few intentional moments.

Native or established controls win when a custom treatment does not improve meaning or experience.

Subtraction is done when every remaining flourish has a named job.

## Verify

Leave a running development server running. If a restart is required, ask the user.

Use the repository's existing checks and verify the actual interface in a browser or appropriate app runtime.

Deliver only when:

- **Invariants** — main colours and fonts remain recognizable; the result reads as an evolution of the same product
- **Thesis** — tokens, components, layout, imagery, and motion express one thesis
- **Slice** — the representative surface works with realistic content, long and short text, empty or error states it exposes, and existing interactions
- **Viewports** — chosen mobile and desktop compositions are deliberate, with no overflow, clipping, or accidental collapse
- **Access** — contrast, keyboard use, focus, semantics, targets, and reduced motion remain sound
- **Subtraction** — every prominent flourish supports hierarchy, meaning, brand, or delight
- **Diff** — existing behavior and unrelated screens remain intact; every changed line belongs to the system or its slice

Deliver the implemented system, the thesis, the preserved invariants, the source-of-truth locations, and visual proof from the representative viewports.
