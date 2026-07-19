# External Postgres

Use an external provider when an existing platform, compliance boundary, extension, service feature, or portability requirement justifies leaving Fly's database products.

Provider products, prices, regions, pooling endpoints, TLS modes, retention, branching, and scale-to-zero behavior are volatile. Verify them in the provider's current official documentation during the run; keep the resulting URLs and decision date with the design.

## Connection design

For every consuming Fly App:

1. Select a database region from measured application-to-database latency and data-residency requirements.
2. Select the provider's application pooler endpoint when compatible; use a direct endpoint only for operations that require it.
3. Require certificate verification or the strongest supported TLS mode. `sslmode=require` encrypts but does not necessarily verify server identity; use the provider's documented CA and verification mode when available.
4. Create least-privilege credentials per app when isolation is required.
5. Set the URL as an app secret and confirm the change's restart impact.

```bash
fly secrets set DATABASE_URL='<provider-url>' --app prod-example-web
fly secrets set DATABASE_URL='<provider-url>' --app prod-example-api
```

Do not commit the URL or print it during validation.

## Capacity and failure criteria

Calculate aggregate connections as all Fly Machines multiplied by each client pool's maximum, plus migrations and operations. Keep that below the provider's current pooled/direct limits with measured headroom.

Test from every Fly region in use:

- DNS, IPv4/IPv6 compatibility, TLS verification, and authentication.
- Connect and query latency at p50 and p99 under expected concurrency.
- Connection recovery after a Fly Machine restart, provider failover, and idle timeout.
- Migration behavior through the selected direct or pooled endpoint.
- Provider backup restoration against recovery objectives.

Include cross-region and cross-provider data transfer in the current cost estimate. If latency or egress dominates, colocate the app and database or revisit the database choice.
