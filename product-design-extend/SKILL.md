---
name: product-design-extend
description: Extend a product with a native UI feature inside an established design system. Use when new workflows, screens, or components must feel native to the product; not for redesigning the system or propagating it onto legacy surfaces.
---

# Extend

Add the capability so it feels **native**: familiar in foundations and interaction, specific where the new workflow needs its own expression.

## Frame the feature

Inspect the running product, the feature's surrounding flow, and the design-system sources. Name:

- the user goal and entry point
- the primary happy path and completion signal
- the data and actions the UI must expose
- required loading, empty, error, disabled, permission, success, and destructive states
- existing behavior, content, navigation, and accessibility contracts that stay unchanged

Product uncertainty is not design freedom. Missing decisions stay visible.

The feature is framed when those five are named and no invented product decision, backend behavior, datum, setting, or control remains.

## Compose first

Map each part of the feature to existing tokens, layout patterns, components, icons, motion, and language. Reuse them in their intended roles.

Create a new system primitive or variant only when all are true:

- existing composition cannot express the need cleanly
- the need is likely to recur or represents a durable product concept
- its states and API can be named from user or domain meaning rather than this one screen
- adding it does not make the shared component harder to understand for existing consumers

Otherwise keep the feature-specific composition local, built from system foundations. A small local component beats an unrelated boolean on a shared one.

Composition is decided when every part of the feature maps to an existing primitive or a justified new one.

## One moment

Use the established design **thesis** to find one feature-specific **moment** of clarity or delight: a legible transition, tactile control, expressive data view, useful illustration, or composition that makes the workflow easier to understand.

That moment stays subordinate to the user's task. Hierarchy, spacing, and the moment do the work. Hunt **slop**: extra cards, pills, gradients, glows, icons, explanatory labels, fake data, motion without informational or tactile value.

New imagery or advanced motion earns its place when it strengthens understanding or the product's established character, and remains performant and accessible.

The moment is chosen when it is named from the thesis and subordinate to the task.

## Implement the flow

Build the smallest end-to-end **slice** that provides the requested value. Connect real application state and existing services. Stop at a static mockup only when the user asked for one.

Preserve the repository's architecture, component conventions, and boundaries. Implement every state named in the frame. Make responsive behavior intentional. Cover keyboard and pointer use, focus management, announcements, target sizes, interruption, reduced motion, and recovery from failure where they apply.

If the feature establishes a reusable design decision, update the design-system source and its relevant examples or documentation in the same change.

The flow is implemented when the slice uses real state, every framed state is handled, and any reusable decision is written back to the system source.

## Verify in context

Leave a running development server running. If a restart is required, ask the user.

Use repository checks and exercise the real flow in the browser or appropriate app runtime.

Verify:

- **Value** — a user can enter, understand, complete, and recover within the requested workflow using real state
- **Compose** — existing tokens and components are used by role; any extension has a clear reusable purpose and complete states
- **Native** — screenshots at representative mobile and desktop viewports sit beside surrounding product surfaces while retaining the one moment
- **States** — every framed state is handled wherever reachable
- **Access** — keyboard, focus, semantics, contrast, targets, reduced motion, and responsive reflow work for the feature
- **Moment** — no element, copy, container, effect, or abstraction exists solely to elaborate
- **Regression** — existing flows and unrelated working-tree changes remain intact, with targeted tests or behavioral checks for the new path

Deliver the working feature, the reused and newly extended system pieces, the user-flow and state proof, and representative visual evidence.
