# Standalone HTML Shell

Read this reference when creating the report file or changing fonts, CSS delivery, or runtime dependencies. Choose one dependency mode before writing markup and record it in the report metadata or delivery notes.

## Choose the Dependency Mode

### Self-contained

Use for downloadable, archival, offline, security-sensitive, or long-lived reports. This is the default when the user has not confirmed that runtime network access is acceptable.

- Compile the report's Tailwind utilities ahead of delivery.
- Inline the compiled CSS in the document's `style` element.
- Use the system sans-serif fallback, or embed a licensed Inter font as a data URL.
- Make the final `.html` file render with networking disabled.

Tailwind CLI can compile a temporary CSS entry that contains the theme and scans the report HTML. Inline the resulting CSS into the shell; temporary build files are not part of the delivered report.

```html
<!doctype html>
<html lang="en" class="antialiased">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="report-dependencies" content="self-contained" />
    <title>Descriptive report title</title>
    <style>
      /* Inline the compiled report CSS here before delivery. */
    </style>
  </head>
  <body class="bg-white font-sans text-zinc-950">
    <main class="isolate">
      <!-- Report content -->
    </main>
  </body>
</html>
```

The final file is complete only after the compiled CSS replaces the comment and the page renders offline.

### Networked

Use for short-lived internal reports when the recipient can reach the public internet and accepts third-party delivery from `rsms.me` and `cdn.jsdelivr.net`. Tailwind Browser compiles styles at runtime, so the report remains one file but is not offline-capable.

```html
<!doctype html>
<html lang="en" class="antialiased">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="report-dependencies" content="networked: rsms.me, cdn.jsdelivr.net" />
    <title>Descriptive report title</title>
    <link rel="preconnect" href="https://rsms.me" />
    <link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style type="text/tailwindcss">
      @theme {
        --font-sans: "InterVariable", Inter, ui-sans-serif, system-ui, sans-serif;
        --font-sans--font-feature-settings: "cv02", "cv03", "cv04", "cv11";
        --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
      }

      @layer base {
        html {
          color-scheme: light;
        }

        body {
          min-width: 20rem;
        }

        code,
        kbd,
        samp {
          font-family: var(--font-mono);
          font-size: 0.8125em;
          overflow-wrap: anywhere;
        }

        :focus-visible {
          outline: 2px solid rgb(37 99 235);
          outline-offset: 3px;
        }

        @media print {
          a[href]::after {
            content: " (" attr(href) ")";
            overflow-wrap: anywhere;
          }
        }
      }
    </style>
  </head>
  <body class="bg-white font-sans text-zinc-950">
    <main class="isolate">
      <!-- Report content -->
    </main>
  </body>
</html>
```

Add conditional component CSS from the tracked-changes reference inside the existing `style` block. Keep one source for each class rather than opening a second theme block.

## Shell Integrity

- Set `lang` to the report's actual language.
- Make `title` identify the report rather than the template or repository alone.
- Keep visible content inside the single `main`; use `header`, `section`, `aside`, and `footer` within it as their semantics require.
- Use fragment links only with unique IDs that resolve.
- Keep scripts limited to the selected styling dependency unless the report genuinely needs behavior.
- For print delivery, verify page breaks and remove controls that have no printed meaning.
