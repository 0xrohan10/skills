---
name: fly-optimization
description: "Design and review Fly.io deployments: app topology, Postgres choice, Fly Proxy lifecycle, private networking, and observability."
---

# Fly.io Deployment Design

Use the current application requirements and deployed state. Treat prices, regions, retention, limits, and provider capabilities as volatile; verify them in the linked official docs during the run.

## 1. Choose the topology

Choose **process groups** only when every answer is yes:

- One image contains every process.
- One deployment may update every process together.
- Every process may receive every app secret.
- App-level ownership, network allocation, and release history are acceptable for every process.

Choose **separate apps** when any answer is no, or when a private service needs Fly Proxy autostart without sharing a public app's network exposure.

Write down the four answers and the selected topology. The step is complete when each process has an image, deploy cadence, secret boundary, owner, exposure, and wake path.

## 2. Choose the database

Choose exactly one:

- **Fly Managed Postgres (MPG):** required availability, backups, recovery, support, and connection pooling fit MPG; every required region and extension is listed in current MPG docs; current pricing fits the approved budget.
- **Unmanaged Fly Postgres:** the team explicitly accepts an unsupported, self-managed database and owns sizing, patching, backup/restore testing, monitoring, alerting, and outage recovery.
- **External Postgres:** an existing provider, compliance boundary, required feature, or portability requirement outweighs cross-provider latency and egress; TLS, pooling, region, and recovery are confirmed with that provider.

Record evidence for every criterion. If the deployment has no Postgres dependency, record `database: not applicable` and load no database reference.

Current-doc checks:

- [MPG overview, regions, capabilities, and pricing](https://fly.io/docs/mpg/)
- [MPG extensions](https://fly.io/docs/mpg/extensions/)
- [Unmanaged Fly Postgres responsibility boundary](https://fly.io/docs/postgres/getting-started/what-you-should-know/)

The step is complete when the selected option satisfies all of its criteria and no rejected option has an unmet requirement that the selection would solve.

## 3. Load only the applicable references

Load exactly one topology reference:

- Process groups: [`references/process-groups.md`](references/process-groups.md)
- Separate apps: [`references/separate-apps.md`](references/separate-apps.md)

Load exactly one database reference when Postgres applies:

- MPG: [`references/fly-mpg.md`](references/fly-mpg.md)
- Unmanaged Fly Postgres: [`references/fly-postgres.md`](references/fly-postgres.md)
- External Postgres: [`references/external-postgres.md`](references/external-postgres.md)

For every environment whose log production, search, retention, access, or export is changing, load the applicable profile:

- Production, or any environment containing production data: [`references/logging-production.md`](references/logging-production.md)
- Non-production without production data: [`references/logging-non-production.md`](references/logging-non-production.md)

Load both profiles when a change spans both classes. Load [`references/metrics.md`](references/metrics.md) when changing metrics, dashboards, API queries, or alerts. Do not load the other topology or database files.

The step is complete when the design applies every rule from the loaded files and contains no configuration copied from an inapplicable branch.

## 4. Apply shared guardrails

### Secrets

Use app secrets for runtime credentials. A secret is available to every Machine in its app, so process groups cannot provide secret isolation. Remember that setting a secret restarts or updates Machines unless it is staged.

### Environment identity

Use separate apps per environment and names that make the environment explicit, such as `prod-example` and `staging-example`. Keep each environment's config explicit and deploy with both `--config` and `--app`; never infer the production target from the current directory. Classify logging per environment from its data and operational requirements rather than assuming one policy applies to every app.

### Static assets

Use `[[statics]]` only to bypass the application server for files available inside the running Machine:

```toml
[[statics]]
  guest_path = "/app/public"
  url_prefix = "/public"
```

Fly Proxy routes matching requests to a static file server inside the Machine. This is not edge hosting or a CDN: a Machine must run, or autostart must wake one. Verify current behavior and caveats in the [`[[statics]]` configuration reference](https://fly.io/docs/reference/configuration/#the-statics-sections).

### Docker process and install lifecycle

Process-group commands replace image `CMD` and remain arguments to `ENTRYPOINT`. Prefer no wrapper; when one is required, forward the command and signals:

```sh
#!/bin/sh
set -eu
exec "$@"
```

Build with all dependencies needed to compile, then copy built artifacts into the runtime stage. Use a production-only install only after every workspace runtime dependency is correctly declared. For Bun, `--ignore-scripts` skips the root project's lifecycle scripts; dependency scripts are controlled separately by Bun's trust model. Use it only when required project install/prepare work is run explicitly, rather than as a blanket Docker default. Verify against [Bun lifecycle scripts](https://bun.com/docs/pm/cli/install#lifecycle-scripts) and [Docker `ENTRYPOINT`/`CMD`](https://docs.docker.com/reference/dockerfile/#understand-how-cmd-and-entrypoint-interact).

## 5. Apply and validate

Before changing Fly resources:

1. Replace every example app, region, port, Machine size, concurrency limit, timeout, and count with a requirement or a measured value.
2. Run `fly config validate --config <fly.toml>` for every config.
3. Confirm secrets, public and Flycast IPs, `.internal` or `.flycast` callers, release command, and rollback path. When a database applies, also confirm its attachment target.
4. Review the rendered diff; apply only when no example placeholder or unverified volatile claim remains.

After applying:

1. Run `fly status --app <app>` and `fly checks list --app <app>` for every app.
2. Exercise each public route and each private route from a Machine on the intended network.
3. Prove every always-on process restarts after a non-zero exit; prove every autostopped service wakes through Fly Proxy; prove direct `.internal` callers do not depend on waking stopped Machines.
4. Complete each applicable branch: verify database connectivity and migrations when a database applies; validate logs when a logging profile was loaded; run the required metrics query when the metrics reference was loaded.
5. Confirm the deployed Machine count, region, size, restart policy, IP allocation, and secret names match the design.

The deployment is complete only when every check passes with captured command output. On failure, stop rollout or roll back; do not classify a successful `fly deploy` alone as validation.
