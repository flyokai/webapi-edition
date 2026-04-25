# flyokai/webapi-edition

> Project template for an async PHP Web API on the [Flyokai framework](https://github.com/flyokai/flyokai).

A starter for HTTP/JSON services: AMPHP HTTP server, OAuth 2.0 (password & client_credentials), declarative MySQL schema (`#[Table]`), search criteria with auto-join discovery, async DB drivers (MySQLi/PDO worker pools or native AMPHP), and Symfony Console commands. No socket-based data-service stack — for that, use [`flyokai/data-service-edition`](https://github.com/flyokai/data-service-edition).

## Bootstrap a new project

```bash
composer create-project flyokai/webapi-edition my-app dev-dev
cd my-app

vendor/bin/flyok-setup install \
    --db-host=localhost \
    --db-user=app \
    --db-pass=secret \
    --db-name=app \
    --base-url=http://localhost:8080
```

`vendor/bin/flyok-setup install` deploys the runtime scripts (`bin/flyok-setup`, `bin/flyok-console`, `bin/flyok-cluster`), creates `storage/flyok.config.json`, generates the OAuth signing keys, and reconciles the database from your `#[Table]`-tagged Solid DTOs.

## What's included

| Group | Modules |
|-------|---------|
| Application core | `flyokai/application`, `flyokai/amphp-injector`, `flyokai/revolt-event-loop` |
| DTOs / patterns | `flyokai/data-mate`, `flyokai/composition`, `flyokai/generic`, `flyokai/misc` |
| Schema & search | `flyokai/db-schema`, `flyokai/search-criteria` |
| Database | `flyokai/laminas-db`, `flyokai/laminas-db-driver-amp`, `flyokai/laminas-db-driver-async`, `flyokai/laminas-db-bulk-update`, `flyokai/zend-db-sql-insertmultiple` |
| Async infra | `flyokai/amp-mate`, `flyokai/amp-data-pipeline`, `flyokai/amp-csv-reader`, `flyokai/amp-opensearch`, `flyokai/amp-channel-dispatcher` |
| CLI | `flyokai/symfony-console` |
| Auth | `flyokai/user`, `flyokai/oauth-server` |
| Indexer | `flyokai/indexer` |
| Docs | `flyokai/flyokai` (metapackage) |

For the framework reference, see `vendor/flyokai/flyokai/README.md` after install.

## Day-to-day commands

```bash
# Subsequent setup runs (schema reconcile, etc.)
php bin/flyok-setup upgrade

# General CLI
php bin/flyok-console acl:role:create --name admin '[]'
php bin/flyok-console user:create --uname=admin --email=admin@example.com --role=admin --pass=secret123
php bin/flyok-console oauth:client:create admin client_credentials

# HTTP server (single process)
php bin/flyok-console http:start
php bin/flyok-console http:restart
php bin/flyok-console http:stop

# Or run as a cluster
php bin/flyok-cluster start
php bin/flyok-cluster stop
php bin/flyok-cluster restart

# Reindex (if you build indexers)
php bin/flyok-console indexer:reindex
```

## Branching

This template follows the same convention as the rest of the framework:

- `main` — stable; what you get if you don't pin a branch.
- `dev` — active development; what `composer create-project … dev-dev` resolves.

Inside the template, every flyokai package is required at `dev-dev`. To bump to a stable release line later, swap `dev-dev` for `^1.0` (or whatever the framework releases).

## `composer.lock` is committed — and that's intentional

This template ships with a `composer.lock` checked in, and `composer create-project` will use it.

### Why

1. **Reproducibility for new projects.** When someone runs `composer create-project flyokai/webapi-edition my-app`, they get the exact same set of vendor versions every member of the team gets. Without the lock, each new project would float to whatever HEAD looks like on every flyokai/* repo at install time — leading to subtle, hard-to-reproduce drift.
2. **Tested combinations.** The lock pins a combination of flyokai/* versions that has been smoke-tested together. Random `dev-dev` HEADs across 23 packages are not.
3. **Faster installs.** `composer install` (which is what `create-project` runs) reads the lock directly. `composer update` resolves the dependency graph from scratch — much slower and network-heavy.

### Update flow

In **this template repository**:

```bash
# 1. Bump dependency tips. This re-resolves the entire graph.
composer update

# 2. Run the local smoke test. (Repeat install, run setup, run tests.)
rm -rf vendor bin storage
vendor/bin/flyok-setup install --db-host=… --db-user=… …
php bin/flyok-console ping     # sanity check

# 3. If everything works, commit the new lock.
git add composer.lock
git commit -m "chore: bump flyokai/* to <date>"
git push origin dev
# Merge dev → main when you want the stable channel to advance.
```

When (not if) `composer update` produces an incompatible mix, `composer.json` is the place to add temporary version constraints (e.g. `"flyokai/foo": "dev-dev as 1.2.0"` or pin to a specific commit) until the upstream module is fixed.

In a **downstream project** (one that was created from this template):

```bash
# Pull whatever versions the template author tested.
composer install                                 # uses the lock as-is

# Or bump just the flyokai/* family at any time.
composer update 'flyokai/*'

# Or take whatever is HEAD on every dependency.
composer update
```

Downstream projects keep their own `composer.lock` once they diverge — they don't track the template's lock anymore.

### When to update the template's lock

- **Always:** before tagging a new release of either edition.
- **On security fixes** in any flyokai/* dependency.
- **On a regular cadence** (weekly or monthly) so the lock doesn't drift months away from the actual `dev` branches.
- **Never** silently — every lock bump should be a single commit with a recognisable message and a smoke test passing.

## Customising

A new project from this template typically:

1. Adds its own module package (e.g. `acme/api-module`) under a new `vendor/`-namespaced repo and requires it.
2. Registers DI config files via that module's `bootstrap.php` and `Bootstrap\Type\Web` to add HTTP routes.
3. Tags Solid DTOs with `#[Table]` so the schema framework reconciles the database on `setup:upgrade`.

The [`new-module`](.agents/skills/flyokai-new-module.md), [`new-endpoint`](.agents/skills/flyokai-new-endpoint.md), [`new-dto`](.agents/skills/flyokai-new-dto.md) skills (synced from `flyokai/flyokai`) scaffold each step.

## What's *not* in this edition

If your project needs **socket-based async services** (long-running services that accept channel-multiplexed requests over a socket, with OAuth handshake and request routing built in), use [`flyokai/data-service-edition`](https://github.com/flyokai/data-service-edition) instead. It includes everything here plus `flyokai/data-service` and `flyokai/data-service-message`.

> Note: `flyokai/amp-channel-dispatcher` is included in both editions because the cluster/worker IPC uses it. It is also the underlying primitive on which `flyokai/data-service` is built.

This edition also intentionally excludes:

- `flyokai/magento-dto`, `flyokai/magento-amp-mate` — Magento-specific DTOs and cache backend
- `unirgy/rapidflow-*`, `unirgy/license-*`, `unirgy/service-connector` — business modules

Add any of these via `composer require` if your project needs them.

## License

MIT
