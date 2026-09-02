---
name: product-design-propagate
description: Apply an established design system across inconsistent product surfaces. Use when propagating approved tokens, components, layout, or interaction patterns; not for inventing a new visual direction.
---

# Apply

Apply the established system. Information, behavior, and product intent stay put.

## Resolve the source of truth

Locate the actual tokens, primitives, components, layout rules, assets, interaction patterns, and reference screens. Inspect them in code and rendered form. If prose documentation disagrees with a maintained component or approved reference screen, surface the conflict.

Preference is not evidence that the approved system should change. A gap may need a small system extension.

The source of truth is resolved when those artifacts are located in both code and rendered form, and every prose conflict is surfaced.

## Build a coverage map

Inventory the routes, major states, shared shells, and reusable components in scope. Group divergence by root cause:

- hard-coded values that should use semantic tokens
- bespoke controls that duplicate a system component
- inconsistent typography, spacing, grid, surfaces, or responsive behavior
- missing interaction, focus, loading, empty, error, disabled, or selected states
- feature-local patterns with no system equivalent

For each surface, record current divergence, intended system primitive or rule, behavior that must remain invariant, and the proof needed.

The **coverage** map is complete when every in-scope route, major state, shared shell, and reusable component has those four fields filled.

## Migrate surgically

Work the coverage map in dependency order:

1. Shared tokens and foundations
2. Shared primitives and components
3. Shared shells and layout structures
4. Feature surfaces and all user-visible states
5. Legacy styles, components, and imports this migration made unused

Start with shared roots that remove divergence from multiple screens, then migrate feature slices so each completed slice remains usable.

Preserve content, domain behavior, URLs, state transitions, analytics hooks, accessibility semantics, and deliberate feature distinctions. A meaningful exception stays.

When no existing primitive can express a recurring need, extend the system at its natural source of truth with the smallest general rule or component that serves the observed cases. A one-off stays local, composed from system foundations.

Prefer direct adoption. If a temporary boundary is required, make it explicit and remove it within the owned scope.

A slice is migrated when its coverage entries resolve to the approved system or a documented exception, and the legacy artifacts it made unused are gone.

## Review the product

Render each migrated slice at representative mobile and desktop **viewports** with realistic content. Compare it with approved reference surfaces for hierarchy, density, rhythm, component anatomy, and interaction character. Underlying decisions match; exact pixels need not.

Hunt **residue**: nested legacy spacing, old hover or focus treatments, one-off icons, mismatched empty states, and responsive layouts that only conform at one width.

A slice is reviewed when representative viewports share the reference surfaces' decisions and no residue remains.

## Completion gate

Leave a running development server running. If a restart is required, ask the user.

Use repository checks plus runtime verification.

Complete only when:

- every in-scope route, shared surface, and relevant state has a disposition in the coverage map
- visible values and patterns resolve to the approved system or a documented intentional exception
- shared fixes are used instead of repeated page-level patches where a real shared cause exists
- behavior, content, analytics, accessibility semantics, and feature-specific meaning remain intact
- representative mobile and desktop renders show coherent hierarchy, spacing, components, and states without overflow or clipping
- no temporary duplicate API or legacy artifact created by the migration remains
- repository checks and the narrow behavioral flows covering migrated surfaces pass
- unrelated working-tree changes and out-of-scope surfaces remain untouched

Deliver the coverage completed, system extensions made, intentional exceptions retained, visual and behavioral proof, and any explicitly out-of-scope surfaces.
