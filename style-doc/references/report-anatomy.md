# Report Anatomy and Editorial System

Read this reference before composing or restyling hierarchy, spacing, typography, metadata, metrics, or reviewer notes.

## Visual Direction

The report should feel like a precise editorial review: quiet, evidence-dense, and authored. Use white space and rules to establish hierarchy instead of enclosing every section in a card.

| Element | Treatment |
| --- | --- |
| Page | White canvas, zinc-950 primary text, zinc-600 supporting text |
| Accent | Blue-700 for eyebrows, links, focus, and reviewer commentary |
| Container | `mx-auto max-w-4xl px-4 sm:px-6 lg:px-8` |
| Section | `border-t border-zinc-950/10 py-12 sm:py-16` |
| Eyebrow | `font-mono text-sm/6 tracking-wide text-blue-700 uppercase` |
| H1 | `mt-3 max-w-[24ch] text-3xl font-semibold tracking-tight text-balance sm:text-4xl` |
| H2 | `text-2xl font-semibold tracking-tight text-balance` |
| H3 | `mt-8 text-lg font-semibold tracking-tight` |
| Body | `text-base/7 text-pretty text-zinc-600 sm:text-sm/6` |
| Prose measure | `max-w-[70ch]` |
| Divider | `border-zinc-950/10` for major rules, `border-zinc-950/5` for row rules |

Body text stays at `text-base` on mobile. The `sm:text-sm` shift makes wide reports denser only after the viewport can support it.

## Page Order

Use the sections that the evidence needs, in this order:

1. Header with report type, title, and metadata.
2. Reviewed-copy banner when tracked changes are present.
3. Executive summary with the conclusion and decision-critical caveats.
4. Summary metrics when the numbers are meaningful without surrounding prose.
5. Evidence sections separated by top borders.
6. Roadmap or recommendations after the evidence they depend on.
7. Appendix for method, exclusions, definitions, and residual uncertainty.

Omit empty or decorative sections. Keep the summary to the few claims that change how the reader interprets the rest of the report.

## Header and Metadata

The header establishes scope and method without marketing copy.

```html
<header class="border-b border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <p class="font-mono text-sm/6 tracking-wide text-blue-700 uppercase">Technical review</p>
    <h1 class="mt-3 max-w-[24ch] text-3xl font-semibold tracking-tight text-balance sm:text-4xl">
      Report title that states the subject and decision
    </h1>
    <dl class="mt-8 grid grid-cols-1 gap-x-6 gap-y-4 sm:grid-cols-2 lg:grid-cols-4">
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Scope</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">Defined system boundary</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Date</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">
          <time datetime="2026-07-19">19 July 2026</time>
        </dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Method</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">Evidence sources used</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium text-zinc-950 sm:text-sm/6">Dependencies</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">Self-contained</dd>
      </div>
    </dl>
  </div>
</header>
```

Use ISO dates in `datetime`; display the date in the report's editorial convention. Let metadata values wrap naturally.

## Summary and Metrics

Start the summary with the strongest supported conclusion. Follow it with the caveat or prerequisite most likely to alter action.

Use a definition list for metrics because each number names one measure:

```html
<dl class="mt-10 grid grid-cols-1 gap-px overflow-hidden border-y border-zinc-950/10 bg-zinc-950/10 sm:grid-cols-3">
  <div class="bg-white px-4 py-5 sm:px-6">
    <dt class="text-base/7 text-zinc-600 sm:text-sm/6">Verified findings</dt>
    <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums text-zinc-950">12</dd>
  </div>
  <div class="bg-white px-4 py-5 sm:px-6">
    <dt class="text-base/7 text-zinc-600 sm:text-sm/6">Require validation</dt>
    <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums text-zinc-950">3</dd>
  </div>
  <div class="bg-white px-4 py-5 sm:px-6">
    <dt class="text-base/7 text-zinc-600 sm:text-sm/6">Recommended actions</dt>
    <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums text-zinc-950">5</dd>
  </div>
</dl>
```

Display only metrics that can be defined precisely. Preserve approximation marks and units.

## Evidence Sections

Give every major finding group a stable ID and a sentence that explains the evidentiary distinction.

```html
<section id="evidence" class="border-t border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">Evidence</h2>
    <p class="mt-4 max-w-[70ch] text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
      State what is verified, what remains uncertain, and which sources establish the distinction.
    </p>
  </div>
</section>
```

Number headings only when the source document has meaningful numbered references. Use `h3` for finding groups inside a section; keep card-like treatment for compact legends or statuses rather than ordinary prose.

## Reviewer Notes

Use a reviewer note for a source-backed caveat, a sequencing constraint, or a qualification that would interrupt the main paragraph.

```html
<aside class="mt-6 border-l-2 border-blue-600 bg-blue-50/50 px-4 py-3 text-blue-950" aria-label="Reviewer note">
  <p class="text-base/7 text-pretty sm:text-sm/6">
    <strong class="font-medium">Reviewer note:</strong>
    State the concrete evidence, caveat, or sequencing constraint in a full sentence.
  </p>
</aside>
```

Blue is reserved here for review commentary, links, and focus. Operational risk/action labels and tracked editorial changes use the separate palettes defined in their own references.

## Editorial Copy

- Use `appears unconsumed` when static evidence is incomplete; use `dead` only when deletion safety is established.
- Use `requires verification` when tests or runtime evidence are missing; use a risk claim only at the confidence the evidence supports.
- Cite file paths, line ranges, commands, URLs, or other source locations for adversarial claims.
- Keep paragraphs focused on one conclusion and its support.
- Use full sentences in reviewer notes, captions, and caveats.
- Make recommendation verbs explicit: retain, verify, revise, remove, sequence, or defer.
