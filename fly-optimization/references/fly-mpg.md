# Fly Managed Postgres

Choose MPG when the workload fits the currently documented regions, extensions, roles, pool modes, and budget, and the team wants Fly to own availability, backups/recovery, monitoring, scaling, support, and incident response.

Verify volatile facts at decision time:

- [Capabilities, regions, storage, and current pricing](https://fly.io/docs/mpg/)
- [Creation and attachment](https://fly.io/docs/mpg/create-and-connect/)
- [Pool modes, users, roles, and databases](https://fly.io/docs/mpg/cluster-configuration/)
- [Supported extensions](https://fly.io/docs/mpg/extensions/)

## Provision and attach

Create with the dashboard or `fly mpg create`, selecting a region near the application and storage from measured growth plus restoration headroom. Do not copy plan prices or region lists into durable deployment guidance.

Attach each consuming app with the MPG command:

```bash
fly mpg list
fly mpg attach <cluster-id> --app prod-example-web
fly mpg attach <cluster-id> --app prod-example-api
```

`fly mpg attach` adds the pooled connection URL as an app secret and restarts the selected app. It is distinct from unmanaged `fly postgres attach`. Use `--variable-name <name>` or the dashboard when `DATABASE_URL` already exists. Confirm the secret name with `fly secrets list --app <app>`; Fly does not reveal its value.

Use the pooled PgBouncer connection for normal application traffic. Use the direct connection only for a verified client requirement such as a migration operation incompatible with the selected pool mode. Session mode is the safer compatibility default; select transaction mode only after checking prepared statements, advisory locks, and other session behavior.

## Boundaries

Create separate MPG users or roles when separate apps need credential or privilege isolation. Validate every required extension against the current extension page before choosing MPG. Confirm the application's Fly private network can reach the cluster; do not infer custom-network support from default-network examples.

After attachment, connect from every app, run a read/write smoke test with the intended role, inspect pool behavior under expected concurrency, and test the documented backup/recovery path against the recovery objective.
