# Unmanaged Fly Postgres

Fly Postgres is a Fly App assembled from Machines, volumes, private networking, and open-source tooling. Fly explicitly classifies it as unsupported and not managed. Choose it only with a named owner and tested operating procedures.

Current responsibility and operation references:

- [What Fly manages and what the team manages](https://fly.io/docs/postgres/getting-started/what-you-should-know/)
- [Create a cluster](https://fly.io/docs/postgres/getting-started/create-pg-cluster/)
- [Attach or detach an app](https://fly.io/docs/postgres/managing/attach-detach/)
- [Backup and restore](https://fly.io/docs/postgres/managing/backup-and-restore/)
- [Scale to zero](https://fly.io/docs/postgres/managing/scale-to-zero/)
- [Upgrades](https://fly.io/docs/postgres/managing/upgrades/)

## Provision and attach

Select cluster size, member count, region, VM, and volume from availability, recovery, load, and growth requirements. A fixed three-node production rule is not a substitute for those requirements.

Attach each app separately:

```bash
fly postgres attach prod-example-db --app prod-example-web
fly postgres attach prod-example-db --app prod-example-api
```

Attachment creates a database and user derived from the consuming app and sets its `DATABASE_URL` secret. Separate attachments therefore create separate credentials and databases by default; explicitly design shared-database access instead of assuming it. Confirm users with `fly postgres users list --app prod-example-db`.

Check `fly postgres attach --help` before attaching. The current CLI defaults the attached user to superuser; pass `--superuser=false` unless the consuming app has a demonstrated need for superuser privileges.

## Scale to zero

Scale-to-zero is an opt-in creation choice for new Development-configuration clusters. It is controlled by `FLY_SCALE_TO_ZERO`, waits for no open connections, and has a fixed timeout described by current docs. It is not enabled for every development cluster, and `fly scale count` is not the switch.

For production or any workload that cannot tolerate database wake latency, verify the variable is absent:

```bash
fly config save --app prod-example-db
fly image show --app prod-example-db
```

Inspect the saved config for `FLY_SCALE_TO_ZERO`. If present, remove it and redeploy using the exact current image reported by `fly image show`, following the current [scale-to-zero procedure](https://fly.io/docs/postgres/managing/scale-to-zero/). Do not paste an image version from an example.

## Required operating evidence

Before production, capture:

- Restore test results and recovery time/data-loss measurements; volume snapshots alone are not a complete backup plan.
- Disk, memory, connection, replication, and availability alerts with an owned response path.
- Patch and major-version upgrade procedure, including rollback.
- Failure recovery for a full volume, OOM, member loss, and unavailable region.
- Capacity evidence for per-app connection pools across every Machine.

Use built-in `pg_database_size_bytes`, `pg_stat_database_numbackends`, replication, volume, memory, and exit metrics where applicable. Alert thresholds must be measured from the cluster and its objectives rather than copied fixed percentages.
