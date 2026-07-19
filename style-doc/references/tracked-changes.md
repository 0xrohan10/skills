# Reviewed Copies and Tracked Changes

Read this reference for the `reviewed` branch or whenever a material source claim must remain visible beside its correction.

## Editorial Palette

Tracked changes have a dedicated palette:

- **Fuchsia plus strikethrough:** source text removed or disproven.
- **Cyan plus underline:** correction or narrowing inserted by review.
- **Blue note:** evidence or rationale supplied by the reviewer.

Operational badges use red, amber, emerald, blue, and zinc as defined in `evidence-tables.md`. Keep fuchsia and cyan exclusive to editorial change marks so a correction cannot be mistaken for risk or action status.

Add these classes to the shell's existing style block only when tracked changes are present:

```css
@layer components {
  .tc-del {
    border-radius: 0.25rem;
    background: rgb(253 244 255);
    color: rgb(162 28 175);
    text-decoration-color: rgb(192 38 211 / 0.85);
    text-decoration-thickness: 0.08em;
  }

  .tc-ins {
    border-radius: 0.25rem;
    background: rgb(236 254 255);
    color: rgb(14 116 144);
    text-decoration-line: underline;
    text-decoration-color: rgb(8 145 178 / 0.65);
    text-decoration-thickness: 0.08em;
    text-underline-offset: 0.12em;
  }
}
```

For self-contained reports, include the compiled form of these classes in the inlined stylesheet.

## Reviewed-Copy Banner

Place one banner after the report header. The legend names each mark so the meaning survives monochrome rendering and color-vision differences.

```html
<section class="border-b border-blue-950/10 bg-blue-50/40 py-6" aria-labelledby="review-status">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <div class="flex flex-col gap-5 sm:flex-row sm:items-start sm:justify-between">
      <div>
        <h2 id="review-status" class="font-mono text-sm/6 tracking-wide text-blue-700 uppercase">
          Reviewed copy
        </h2>
        <p class="mt-2 max-w-[70ch] text-base/7 text-pretty text-blue-950 sm:text-sm/6">
          Material corrections are marked against the source report and supported by reviewer notes.
        </p>
      </div>
      <dl class="grid shrink-0 grid-cols-1 gap-2 text-xs/5 sm:grid-cols-3">
        <div class="rounded-md bg-fuchsia-50 px-2 py-1 text-fuchsia-800 ring-1 ring-fuchsia-600/20 ring-inset">
          <dt class="font-medium">Removed</dt>
          <dd>Struck through</dd>
        </div>
        <div class="rounded-md bg-cyan-50 px-2 py-1 text-cyan-800 ring-1 ring-cyan-600/20 ring-inset">
          <dt class="font-medium">Inserted</dt>
          <dd>Underlined</dd>
        </div>
        <div class="rounded-md bg-blue-50 px-2 py-1 text-blue-800 ring-1 ring-blue-600/20 ring-inset">
          <dt class="font-medium">Reviewer note</dt>
          <dd>Evidence</dd>
        </div>
      </dl>
    </div>
  </div>
</section>
```

The banner's `h2` participates in document hierarchy. If the report structure cannot support an `h2` at this position, use a labeled `p` and keep the section's `aria-label` explicit.

## Markup

Use semantic `del` and `ins` elements for material corrections:

```html
<p class="text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
  Availability is
  <del class="tc-del" datetime="2026-07-19">limited to one verified region</del>
  <ins class="tc-ins" datetime="2026-07-19">confirmed in two regions, with a third still requiring validation</ins>.
</p>
```

Add `cite` when a stable review artifact or evidence URL exists. Use a valid machine-readable value for `datetime`.

Mark only claims that were contradicted, narrowed, materially reclassified, or changed in a way the reader must audit. Apply ordinary copyediting silently. When a paragraph would become unreadable under dense marks, preserve the original in a nearby quoted block and present the corrected paragraph once, followed by a reviewer note explaining the material differences.

## Traceability

- Keep the source claim and correction in the same local context whenever readability permits.
- Follow a disputed claim with a reviewer note containing the source path, line range, command output, citation, or uncertainty.
- Preserve the source's degree of confidence unless the review evidence justifies changing it.
- Keep correction language factual; place rationale in the reviewer note.
- Count every material correction during validation and verify that each has evidence or an explicit unresolved status.
