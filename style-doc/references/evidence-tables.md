# Evidence Tables and Operational Statuses

Read this reference before emitting any evidence, inventory, risk, route, or roadmap table. Tables carry comparable data; prose carries narrative reasoning.

## Accessible Overflow Pattern

Place overflow on a labeled region around the table. Keep the table's native semantics, give it a caption, scope every header, and set a useful minimum width so columns wrap intentionally rather than collapsing.

```html
<div
  class="mt-6 overflow-x-auto rounded-sm focus-visible:ring-2 focus-visible:ring-blue-600 focus-visible:ring-offset-2 focus-visible:outline-none"
  role="region"
  aria-labelledby="evidence-table-1-caption"
  tabindex="0"
>
  <table class="w-full min-w-[48rem] table-fixed text-left text-base/7 sm:text-sm/6">
    <caption id="evidence-table-1-caption" class="sr-only">
      Evidence inventory with source, finding, confidence, and action
    </caption>
    <thead>
      <tr class="border-b border-zinc-950/10 text-zinc-950">
        <th scope="col" class="w-3/12 py-2 pr-4 font-medium">Source</th>
        <th scope="col" class="w-4/12 px-4 py-2 font-medium">Finding</th>
        <th scope="col" class="w-2/12 px-4 py-2 font-medium">Confidence</th>
        <th scope="col" class="w-3/12 py-2 pl-4 font-medium">Action</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-zinc-950/5 text-zinc-600">
      <tr>
        <th scope="row" class="py-3 pr-4 align-top font-normal text-zinc-950">
          <code class="break-words">src/example/module.ts:24-38</code>
        </th>
        <td class="px-4 py-3 align-top whitespace-normal">State the evidence-backed finding.</td>
        <td class="px-4 py-3 align-top whitespace-normal">Verified</td>
        <td class="py-3 pl-4 align-top whitespace-normal">
          <span class="inline-flex rounded-md bg-blue-50 px-1.5 py-0.5 text-xs/5 font-medium whitespace-nowrap text-blue-700 ring-1 ring-blue-600/20 ring-inset">
            Revise
          </span>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

The scroll region receives keyboard focus so keyboard users can reach horizontally clipped columns. Give every table instance a unique caption ID and use the same value in its region's `aria-labelledby`; incrementing the example suffix is sufficient. The `caption` is the region's accessible name and summarizes the comparison. A visible caption is preferable when readers need that context on screen; replace `sr-only` with `caption-top pb-3 text-left text-sm/6 text-zinc-600`.

## Semantic Rules

- Use the first identifying cell in each row as `th scope="row"`; use `td` when no row label exists.
- Use `th scope="col"` for every column heading and `th scope="rowgroup"` when rows have meaningful grouped labels.
- Keep source order logical in the DOM; CSS controls presentation without changing reading order.
- Use `colspan` only for a real grouped relationship and provide the corresponding header scope.
- Put explanatory text outside the table when it applies to the table as a whole.
- Use links with descriptive labels inside cells; include raw URLs only when the URL is evidence.

## Overflow and Wrapping

- Apply `overflow-x-auto` only to the labeled region, not to the page, section, or `table` element.
- Give comparison tables a content-appropriate `min-w-*`; `44rem` to `56rem` fits most three-to-five-column reports.
- Let body cells wrap with `whitespace-normal`.
- Let paths and identifiers wrap with `overflow-wrap:anywhere` or `break-words`.
- Keep short status labels on one line with `whitespace-nowrap`.
- Preserve comfortable desktop columns with `table-fixed` plus explicit widths when the content shape is known; use the automatic table layout when values determine useful widths better.

At 390px, only the table region should scroll horizontally. At 1440px, the table should fill the report measure without forced page overflow or unnecessarily stretched prose.

## Operational Palette

Operational labels state their meaning in text. Their colors never encode editorial insertion or deletion; fuchsia and cyan remain reserved for tracked changes.

### Risk

```html
<span class="inline-flex rounded-md bg-red-50 px-1.5 py-0.5 text-xs/5 font-medium text-red-700 ring-1 ring-red-600/20 ring-inset">High risk</span>
<span class="inline-flex rounded-md bg-amber-50 px-1.5 py-0.5 text-xs/5 font-medium text-amber-800 ring-1 ring-amber-600/20 ring-inset">Verify</span>
```

### Action

```html
<span class="inline-flex rounded-md bg-emerald-50 px-1.5 py-0.5 text-xs/5 font-medium text-emerald-700 ring-1 ring-emerald-600/20 ring-inset">Proceed</span>
<span class="inline-flex rounded-md bg-blue-50 px-1.5 py-0.5 text-xs/5 font-medium text-blue-700 ring-1 ring-blue-600/20 ring-inset">Revise</span>
<span class="inline-flex rounded-md bg-zinc-100 px-1.5 py-0.5 text-xs/5 font-medium text-zinc-700 ring-1 ring-zinc-600/20 ring-inset">Retain</span>
```

Use red and amber only for operational risk or validation urgency. Use emerald, blue, and zinc for the action itself. Pair separate risk and action columns when both dimensions matter.

## Roadmaps

Use one row per phase or independently executable action. Recommended columns are phase, work, prerequisite, risk, and payoff; include only dimensions supported by the source. Sequence verification before any action whose safety depends on it.

```html
<div
  class="mt-6 overflow-x-auto rounded-sm focus-visible:ring-2 focus-visible:ring-blue-600 focus-visible:ring-offset-2 focus-visible:outline-none"
  role="region"
  aria-labelledby="roadmap-1-caption"
  tabindex="0"
>
  <table class="w-full min-w-[52rem] table-fixed text-left text-base/7 sm:text-sm/6">
    <caption id="roadmap-1-caption" class="sr-only">
      Delivery roadmap with prerequisites, risks, and observable outcomes
    </caption>
    <thead>
      <tr class="border-b border-zinc-950/10 text-zinc-950">
        <th scope="col" class="w-2/12 py-2 pr-4 font-medium">Phase</th>
        <th scope="col" class="w-3/12 px-4 py-2 font-medium">Work</th>
        <th scope="col" class="w-3/12 px-4 py-2 font-medium">Prerequisite</th>
        <th scope="col" class="w-2/12 px-4 py-2 font-medium">Risk</th>
        <th scope="col" class="w-2/12 py-2 pl-4 font-medium">Outcome</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-zinc-950/5 text-zinc-600">
      <tr>
        <th scope="row" class="py-3 pr-4 align-top font-normal text-zinc-950">1. Verify</th>
        <td class="px-4 py-3 align-top whitespace-normal">Run the target-environment behavior check.</td>
        <td class="px-4 py-3 align-top whitespace-normal">Representative data and an observable success signal.</td>
        <td class="px-4 py-3 align-top whitespace-normal">
          <span class="inline-flex rounded-md bg-amber-50 px-1.5 py-0.5 text-xs/5 font-medium whitespace-nowrap text-amber-800 ring-1 ring-amber-600/20 ring-inset">
            Requires evidence
          </span>
        </td>
        <td class="py-3 pl-4 align-top whitespace-normal">The assumption is confirmed or disproved.</td>
      </tr>
    </tbody>
  </table>
</div>
```

Every risk label needs a stated basis. Every payoff should describe an observable result rather than a generic benefit.
