---
name: style-doc
description: Create, restyle, or present reviewed standalone HTML technical reports in a cohesive editorial system. Use for new reports, visual restyles, or corrected copies with tracked changes; report-producing skills may invoke it for standalone HTML presentation.
---

# Editorial Technical Reports

Produce a standalone technical document, not a dashboard or marketing page. Preserve the report's evidence-first character while making its hierarchy, caveats, and actions easy to scan.

## Route the Request

- **Create:** Turn source findings or a brief into a new standalone HTML report. Establish information hierarchy without inventing evidence.
- **Restyle:** Apply this system to an existing report. Preserve its claims, ordering, citations, and caveats unless the user also requests editorial changes.
- **Reviewed:** Present a corrected or adversarially reviewed copy. Preserve traceability between material source claims and corrections.

If branches overlap, use `reviewed` whenever corrections must remain visible; otherwise use `restyle` for existing content and `create` for new content.

## Core Invariants

- Use one valid standalone HTML file with one `main`, one `h1`, semantic sections, and a descriptive `title`.
- Keep a white canvas, zinc typography, blue review accents, thin dividers, restrained surfaces, and a `max-w-4xl` reading measure.
- Use Inter when available with a system sans-serif fallback; use monospace for paths, identifiers, and code.
- Let evidence determine the component: prose for reasoning, definition lists for metadata and metrics, semantic tables for comparison, and reviewer notes for caveats.
- Keep claims evidence-backed, uncertainty explicit, headings in sentence case, and editorial changes distinguishable without relying on color.

## Load References by Context

Read every reference whose condition applies before writing the report:

| Condition | Required reference |
| --- | --- |
| Creating a file or changing its fonts, CSS delivery, or runtime dependencies | Read [`references/standalone-shell.md`](references/standalone-shell.md) before choosing the networked or self-contained shell. |
| Creating or restyling report hierarchy, spacing, typography, metadata, metrics, or reviewer notes | Read [`references/report-anatomy.md`](references/report-anatomy.md) before composing sections. |
| The report contains any evidence, inventory, risk, route, or roadmap table | Read [`references/evidence-tables.md`](references/evidence-tables.md) before emitting the first table. |
| The branch is `reviewed`, or source claims must remain visibly corrected | Read [`references/tracked-changes.md`](references/tracked-changes.md) before marking any correction. |
| The `create` branch needs a baseline composition | Read [`references/neutral-example.md`](references/neutral-example.md) after the shell and anatomy references, then replace all sample content. |

## Application Process

1. Select `create`, `restyle`, or `reviewed`, and inventory the source sections, findings, metrics, citations, and caveats. Complete when every source item has a destination or an intentional omission requested by the user.
2. Load each context-triggered reference above. Complete when every applicable reference has informed the chosen structure and components.
3. Build or update the standalone HTML while preserving source meaning and applying the editorial system. Complete when the document opens as one coherent report and all source items are accounted for.

## Completion Gate

Before delivery, verify every applicable condition:

- **Coverage:** Every source section, finding, metric, citation, and caveat is present, or its intentional omission is documented; the `restyle` branch preserves meaning, and the `reviewed` branch keeps material corrections traceable.
- **Document integrity:** The file has a doctype, language, UTF-8 charset, viewport, descriptive title, one `main`, one `h1`, globally unique IDs, valid heading order, and no placeholder or sample content.
- **Dependency integrity:** The chosen network mode is explicit and consistent; networked assets load successfully, or the self-contained file renders with no runtime network dependency.
- **Semantics and access:** Metadata and metrics use definition lists; data comparisons use real tables with captions and scoped headers; links, focus states, labels, and editorial marks remain understandable without color.
- **Visual system:** Typography, measure, spacing, dividers, notes, tables, and status labels form one restrained editorial hierarchy; editorial-change colors stay separate from operational risk/action colors.
- **Responsive behavior:** At 390px and 1440px viewports, body text is legible, headings wrap cleanly, content stays within the page, and each wide table scrolls inside its labeled region without causing page-level horizontal overflow.
- **Evidence quality:** Claims use the strongest available evidence, unverified statements are qualified, adversarial claims include source locations, and reviewer notes contain a concrete caveat, source, or sequencing constraint.

Deliver only when every applicable condition passes.
