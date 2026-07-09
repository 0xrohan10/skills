---
name: style-doc
description: >
  Style HTML documents requested by the user.
---

# Audit Report Visual Style Guide

Use this document to style future audit, review, architecture, or implementation-plan reports.

This is a Markdown style spec with copy-pasteable HTML snippets. Plain GitHub Markdown will not reproduce the full look because the source report relies on Tailwind, Inter, custom tracked-change classes, and HTML tables. Use the snippets when generating a standalone `.html` report, or give this document to another agent as the visual brief.

## Visual Direction

The style is a precise technical audit document, not a dashboard and not a marketing page.

Use:

- White background with zinc text.
- Inter as the primary typeface.
- Blue as the editorial/review accent.
- Emerald for inserted/corrected text.
- Red for removed/disproven text.
- Amber for caution and medium-risk status.
- Thin dividers, generous whitespace, and narrow reading width.
- Tables for evidence, not decorative cards.
- Short editorial notes that feel like review comments.

Avoid:

- Heavy cards around every section.
- Saturated backgrounds.
- Emoji.
- Dense nested bullets.
- Centered marketing copy.
- Decorative gradients outside small review-note surfaces.

## Page Anatomy

Use this order for reports:

1. Report header with eyebrow, title, and metadata grid.
2. Optional adversarial review banner with a tracked-change legend.
3. Executive summary with 2-4 paragraphs and a metric grid.
4. Evidence sections separated by top borders.
5. Tables for inventories, risk maps, route maps, or roadmap phases.
6. Review notes where claims require caveats.
7. Appendix with methodology and caveats.

## Design Tokens

| Element | Treatment |
| --- | --- |
| Page | `bg-white font-sans text-zinc-950` |
| Container | `mx-auto max-w-4xl px-4 sm:px-6 lg:px-8` |
| Section spacing | `py-12 sm:py-16` |
| Section divider | `border-t border-zinc-950/10` |
| Header divider | `border-b border-zinc-950/10` |
| Eyebrow | `font-mono text-base/7 tracking-wide text-blue-700 uppercase sm:text-sm/6` |
| H1 | `mt-3 max-w-[24ch] text-3xl font-semibold tracking-tight text-balance sm:text-4xl` |
| H2 | `text-2xl font-semibold tracking-tight text-balance` |
| H3 | `mt-8 text-lg font-semibold tracking-tight` |
| Body text | `text-base/7 text-pretty text-zinc-600 sm:text-sm/6` |
| Paragraph measure | `max-w-[70ch]` |
| Inline code | Monospace, `0.8125em`, wrap anywhere |
| Table text | `text-base/7 sm:text-sm/6` |
| Table dividers | `border-b border-zinc-950/10`, `divide-y divide-zinc-950/5` |

## HTML Shell

Start standalone reports with this shell.

```html
<!doctype html>
<html lang="en" class="antialiased">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Report Title</title>
    <link rel="preconnect" href="https://rsms.me/" />
    <link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style type="text/tailwindcss">
      @theme {
        --font-sans: 'InterVariable', ui-sans-serif, system-ui, sans-serif;
        --font-sans--font-feature-settings: 'cv02', 'cv03', 'cv04', 'cv11';
      }

      @layer base {
        code {
          font-family: var(--font-mono);
          font-size: 0.8125em;
          overflow-wrap: anywhere;
        }
      }

      @layer components {
        .tc-del {
          border-radius: 0.25rem;
          background: rgb(254 242 242);
          color: rgb(185 28 28);
          text-decoration-color: rgb(220 38 38 / 0.8);
          text-decoration-thickness: 0.08em;
        }

        .tc-ins {
          border-radius: 0.25rem;
          background: rgb(236 253 245);
          color: rgb(4 120 87);
          text-decoration: none;
          box-shadow: inset 0 -1px rgb(5 150 105 / 0.4);
        }

        .review-note {
          margin-top: 0.75rem;
          border-left: 3px solid rgb(37 99 235);
          background: linear-gradient(90deg, rgb(239 246 255), rgb(255 255 255));
          padding: 0.75rem 1rem;
          color: rgb(30 64 175);
        }

        .review-note strong {
          color: rgb(30 58 138);
          font-weight: 500;
        }
      }
    </style>
  </head>
  <body class="bg-white font-sans text-zinc-950">
    <main class="isolate">
      <!-- Report content goes here. -->
    </main>
  </body>
</html>
```

## Report Header

Use the header to establish scope and method quickly.

```html
<header class="border-b border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <p class="font-mono text-base/7 tracking-wide text-blue-700 uppercase sm:text-sm/6">
      Technical audit
    </p>
    <h1 class="mt-3 max-w-[24ch] text-3xl font-semibold tracking-tight text-balance sm:text-4xl">
      Codebase complexity audit: simplification, dead paths, and abstractions
    </h1>
    <dl class="mt-8 grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Repository</dt>
        <dd class="text-base/7 text-zinc-600 sm:text-sm/6">upteaming monorepo</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Date</dt>
        <dd class="text-base/7 text-zinc-600 sm:text-sm/6">July 8, 2026</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Scope</dt>
        <dd class="text-base/7 text-zinc-600 sm:text-sm/6">apps/api, apps/web, packages/shared</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Method</dt>
        <dd class="text-base/7 text-zinc-600 sm:text-sm/6">Import graph, code search, git history</dd>
      </div>
    </dl>
  </div>
</header>
```

## Tracked-Change Banner

Use this when the report is a reviewed or corrected copy.

```html
<section class="border-b border-blue-950/10 bg-blue-50/40 py-6">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <div class="flex flex-col gap-4 sm:flex-row sm:items-start sm:justify-between">
      <div>
        <p class="font-mono text-base/7 tracking-wide text-blue-700 uppercase sm:text-sm/6">
          Adversarial review applied
        </p>
        <p class="mt-2 max-w-[70ch] text-base/7 text-pretty text-blue-950 sm:text-sm/6">
          This copy preserves the original audit while marking corrections from a codebase cross-check.
        </p>
      </div>
      <dl class="grid shrink-0 grid-cols-3 gap-2 text-sm/6 sm:text-xs/5">
        <div class="rounded-md bg-red-50 px-2 py-1 text-red-700 ring-1 ring-red-600/20 ring-inset">
          <dt class="font-medium">Removed</dt>
          <dd>False claim</dd>
        </div>
        <div class="rounded-md bg-emerald-50 px-2 py-1 text-emerald-700 ring-1 ring-emerald-600/20 ring-inset">
          <dt class="font-medium">Inserted</dt>
          <dd>Correction</dd>
        </div>
        <div class="rounded-md bg-blue-50 px-2 py-1 text-blue-700 ring-1 ring-blue-600/20 ring-inset">
          <dt class="font-medium">Note</dt>
          <dd>Evidence</dd>
        </div>
      </dl>
    </div>
  </div>
</section>
```

## Executive Summary

Keep the summary compact and evidence-oriented. Use strong text sparingly.

```html
<section id="summary" class="py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">Executive summary</h2>
    <div class="mt-6 max-w-[70ch] space-y-4 text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
      <p>
        The codebase is not badly engineered; it is
        <strong class="font-medium text-zinc-950">badly labeled and incompletely migrated</strong>.
      </p>
      <p>
        One finding needs verification before anything else:
        <code>workflow-worker.ts</code> is maintained, but deployment config does not run it.
      </p>
    </div>
  </div>
</section>
```

## Metric Grid

Use large numbers for summary-level findings only.

```html
<dl class="mt-10 grid grid-cols-2 lg:grid-cols-4">
  <div class="pr-4 pb-4 lg:py-0 lg:pr-6">
    <dt class="truncate text-base/7 text-zinc-600 sm:text-sm/6">Candidate cleanup</dt>
    <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums">~9,500</dd>
  </div>
  <div class="border-l border-zinc-950/10 pb-4 pl-4 lg:px-6 lg:py-0">
    <dt class="truncate text-base/7 text-zinc-600 sm:text-sm/6">High-risk caveats</dt>
    <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums">6</dd>
  </div>
</dl>
```

## Evidence Section

Use this for every major part of the report.

```html
<section id="dead-code" class="border-t border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">2. Dead code and stale surfaces</h2>
    <p class="mt-4 max-w-[70ch] text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
      Separate verified dead code from routes that need product or migration confirmation.
    </p>
  </div>
</section>
```

## Evidence Table

Tables should be readable on mobile and precise on desktop. Wrap the table in an overflow container.

```html
<div class="-mx-4 -my-2 mt-6 overflow-x-auto whitespace-nowrap sm:-mx-6 lg:-mx-8">
  <div class="inline-block min-w-full px-4 py-2 align-middle sm:px-6 lg:px-8">
    <table class="w-full text-left text-base/7 sm:text-sm/6">
      <thead>
        <tr class="border-b border-zinc-950/10">
          <th class="py-2 pr-4 font-medium whitespace-nowrap">File</th>
          <th class="px-4 py-2 font-medium whitespace-nowrap">Finding</th>
          <th class="py-2 pl-4 font-medium whitespace-nowrap">Action</th>
        </tr>
      </thead>
      <tbody class="divide-y divide-zinc-950/5 text-zinc-600">
        <tr>
          <td class="py-2.5 pr-4 align-top"><code>apps/web/src/lib/sentry.ts</code></td>
          <td class="px-4 py-2.5 whitespace-normal">
            <del class="tc-del">Dead client-side Sentry.</del>
            <ins class="tc-ins">Live through client initialization and catch boundary capture.</ins>
          </td>
          <td class="py-2.5 pl-4">
            <span class="inline-flex items-center rounded-md bg-red-50 px-1.5 py-0.5 text-sm/5 font-medium text-red-700 ring-1 ring-red-600/20 ring-inset sm:text-xs/5">
              keep
            </span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
```

## Status Badges

Use badges as table statuses. Keep them short.

```html
<span class="inline-flex items-center rounded-md bg-emerald-50 px-1.5 py-0.5 text-sm/5 font-medium text-emerald-700 ring-1 ring-emerald-600/20 ring-inset sm:text-xs/5">delete</span>

<span class="inline-flex items-center rounded-md bg-amber-50 px-1.5 py-0.5 text-sm/5 font-medium text-amber-700 ring-1 ring-amber-600/20 ring-inset sm:text-xs/5">verify</span>

<span class="inline-flex items-center rounded-md bg-red-50 px-1.5 py-0.5 text-sm/5 font-medium text-red-700 ring-1 ring-red-600/20 ring-inset sm:text-xs/5">keep</span>

<span class="inline-flex items-center rounded-md bg-blue-50 px-1.5 py-0.5 text-sm/5 font-medium text-blue-700 ring-1 ring-blue-600/20 ring-inset sm:text-xs/5">rename</span>
```

## Tracked Changes

Use tracked changes when preserving the original claim matters.

```html
<p class="text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
  Delete <del class="tc-del">all section 2 items with no risk</del>
  <ins class="tc-ins">only the verified-dead set after preserving live invite and notification flows</ins>.
</p>
```

Do not use tracked changes for every edit. Reserve them for claims that were contradicted, narrowed, or materially reclassified.

## Reviewer Notes

Use notes for evidence, caveats, and sequencing constraints.

```html
<p class="review-note text-base/7 text-pretty sm:text-sm/6">
  <strong>Reviewer note:</strong> remove source-imported dependencies only after their orphaned files are deleted.
</p>
```

## Roadmap Table

Roadmaps should separate work, risk, and payoff.

```html
<table class="w-full text-left text-base/7 sm:text-sm/6">
  <thead>
    <tr class="border-b border-zinc-950/10">
      <th class="py-2 pr-4 font-medium whitespace-nowrap">Phase</th>
      <th class="px-4 py-2 font-medium whitespace-nowrap">Work</th>
      <th class="px-4 py-2 font-medium whitespace-nowrap">Risk</th>
      <th class="py-2 pl-4 font-medium whitespace-nowrap">Payoff</th>
    </tr>
  </thead>
  <tbody class="divide-y divide-zinc-950/5 text-zinc-600">
    <tr>
      <td class="py-2.5 pr-4 align-top font-medium text-zinc-950">0. Verify</td>
      <td class="px-4 py-2.5 whitespace-normal">Confirm whether the worker runs in production.</td>
      <td class="px-4 py-2.5 align-top">
        <span class="inline-flex items-center rounded-md bg-red-50 px-1.5 py-0.5 text-sm/5 font-medium text-red-700 ring-1 ring-red-600/20 ring-inset sm:text-xs/5">urgent</span>
      </td>
      <td class="py-2.5 pl-4 whitespace-normal">Possible production incident caught.</td>
    </tr>
  </tbody>
</table>
```

## Copy Rules

Use direct, evidence-backed language.

- Prefer "appears unconsumed" over "dead" unless route/file deletion is proven.
- Prefer "requires verification" over "zero risk" unless there is test or runtime evidence.
- Include file paths and line references for adversarial claims.
- Keep paragraphs under 90 words where possible.
- Use sentence case for headings and table headers.
- Use full sentences in review notes.

## Generation Prompt

Use this prompt when asking an agent to create a new document in this style:

```text
Create a standalone HTML technical audit report using the visual style in docs/audit-report-visual-style.md.

Requirements:
- Use Inter, Tailwind browser v4, white background, zinc text, blue accents, and max-w-4xl content width.
- Structure the report with a header, optional review banner, executive summary, metric grid, evidence sections, tables, roadmap, and appendix.
- Use tracked-change markup only for contradicted or materially revised claims.
- Use reviewer notes for caveats and sequencing constraints.
- Keep typography compact, editorial, and evidence-first.
- Avoid cards except small status badges and the review banner legend.
- Make tables horizontally scrollable on mobile.
```

## Final Checklist

Before shipping a report in this style, verify:

- The page has one `main` element and clear section IDs.
- Body text remains at least `text-base` on mobile.
- Every table is wrapped in `overflow-x-auto`.
- Tracked changes are used sparingly and only where they add editorial value.
- Review notes include concrete evidence or a sequencing constraint.
- The roadmap does not label work as low risk without verification.
- The document is readable without relying on color alone.
