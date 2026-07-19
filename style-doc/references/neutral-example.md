# Neutral Composition Example

Read this only when the `create` branch needs a baseline composition. It demonstrates assembly, not report content: replace every title, value, claim, source, date, and ID. The authoritative shell, tokens, table rules, and tracked-change rules remain in their named references.

Insert this composition inside the selected shell's `main` element:

```html
<header class="border-b border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <p class="font-mono text-sm/6 tracking-wide text-blue-700 uppercase">Technical assessment</p>
    <h1 class="mt-3 max-w-[24ch] text-3xl font-semibold tracking-tight text-balance sm:text-4xl">
      Service readiness review
    </h1>
    <dl class="mt-8 grid grid-cols-1 gap-x-6 gap-y-4 sm:grid-cols-2 lg:grid-cols-4">
      <div>
        <dt class="text-base/7 font-medium sm:text-sm/6">Scope</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">Application and deployment boundary</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium sm:text-sm/6">Date</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6"><time datetime="2026-07-19">19 July 2026</time></dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium sm:text-sm/6">Method</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">Configuration and runtime evidence</dd>
      </div>
      <div>
        <dt class="text-base/7 font-medium sm:text-sm/6">Dependencies</dt>
        <dd class="mt-1 text-base/7 text-zinc-600 sm:text-sm/6">Self-contained</dd>
      </div>
    </dl>
  </div>
</header>

<section id="summary" class="py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">Executive summary</h2>
    <div class="mt-6 max-w-[70ch] space-y-4 text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
      <p>
        The assessment found a stable core with one deployment assumption that requires runtime verification before release.
      </p>
      <p>
        Resolve that prerequisite first; the remaining recommendations can proceed independently after the evidence inventory is confirmed.
      </p>
    </div>
    <dl class="mt-10 grid grid-cols-1 gap-px overflow-hidden border-y border-zinc-950/10 bg-zinc-950/10 sm:grid-cols-3">
      <div class="bg-white px-4 py-5 sm:px-6">
        <dt class="text-base/7 text-zinc-600 sm:text-sm/6">Verified findings</dt>
        <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums">8</dd>
      </div>
      <div class="bg-white px-4 py-5 sm:px-6">
        <dt class="text-base/7 text-zinc-600 sm:text-sm/6">Require validation</dt>
        <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums">1</dd>
      </div>
      <div class="bg-white px-4 py-5 sm:px-6">
        <dt class="text-base/7 text-zinc-600 sm:text-sm/6">Recommended actions</dt>
        <dd class="mt-1 text-3xl font-semibold tracking-tight tabular-nums">3</dd>
      </div>
    </dl>
  </div>
</section>

<section id="findings" class="border-t border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">Findings</h2>
    <p class="mt-4 max-w-[70ch] text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
      Verified behavior is separated from configuration that still needs observation in the target environment.
    </p>
    <aside class="mt-6 border-l-2 border-blue-600 bg-blue-50/50 px-4 py-3 text-blue-950" aria-label="Reviewer note">
      <p class="text-base/7 text-pretty sm:text-sm/6">
        <strong class="font-medium">Reviewer note:</strong>
        The release decision depends on one runtime check; static configuration alone does not establish the deployed behavior.
      </p>
    </aside>
  </div>
</section>

<section id="recommendations" class="border-t border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">Recommendations</h2>
    <ol class="mt-6 max-w-[70ch] space-y-4 text-base/7 text-zinc-600 sm:text-sm/6">
      <li><strong class="font-medium text-zinc-950">Verify:</strong> run the environment-specific check and retain its output.</li>
      <li><strong class="font-medium text-zinc-950">Revise:</strong> update the configuration whose behavior differs from the intended state.</li>
      <li><strong class="font-medium text-zinc-950">Confirm:</strong> rerun the focused validation and record the result.</li>
    </ol>
  </div>
</section>

<section id="method" class="border-t border-zinc-950/10 py-12 sm:py-16">
  <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
    <h2 class="text-2xl font-semibold tracking-tight text-balance">Method and limits</h2>
    <p class="mt-4 max-w-[70ch] text-base/7 text-pretty text-zinc-600 sm:text-sm/6">
      State the inspected sources, excluded surfaces, date boundary, commands run, and residual uncertainty.
    </p>
  </div>
</section>
```

Add tables from `evidence-tables.md` only where the source contains comparable evidence. Add the reviewed-copy banner and semantic marks from `tracked-changes.md` only for the `reviewed` branch.
