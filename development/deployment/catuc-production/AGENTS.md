# CATUC Production

## Purpose

- Owns the immutable production image inputs, Compose topology, release manifests, backup and restore commands, cutover, health checks, and rollback for `portal.catuc.org`.

## Ownership

- `compose.yaml` owns the production Frappe, MariaDB, Redis, private-network, and Traefik attachment topology.
- `.env.example` documents non-secret deployment inputs; live values and credentials stay under `/srv/apps/academia/frappe/secrets`.
- `scripts/` owns strict operational commands for release validation, deployment, data movement, health checks, cutover, and rollback.
- `manifests/` owns reviewed source and image release records, never credentials or live snapshots.

## Local Contracts

- The live project root is `/srv/apps/academia/frappe` and the public hostname is `portal.catuc.org`.
- Every service joins the private external network named `frappe`; only `frontend` joins the discovered shared edge network.
- MariaDB, Redis, backend, websocket, workers, and scheduler publish no host ports.
- Images use `registry.gitlab.com/zinger-teams/academia/frappe:frappe16-[Y-m-d]` tags and production deploys pin the resolved image digest.
- Backend and worker roles resolve `portal.catuc.org` through Docker `host-gateway` so PDF generators fetch canonical HTTPS assets through Traefik without joining the edge network.
- Production email stays muted until the exact formerly failing document renders as a valid PDF and one approved print-attached message reaches `Sent`.
- Old-live data is rollback evidence. The checksummed `development.localhost` snapshot is authoritative and is restored without merging old-live records.

## Work Guidance

- Derive runtime commands and image behavior from the repository's official Frappe Docker production patterns.
- Refuse deployment when source trees are dirty, image refs are mutable, required secrets are absent, or the shared edge/private network inventory is unresolved.
- Keep Bash scripts strict, idempotent where practical, and explicit about target host, project, site, release, and snapshot.

## Verification

- `docker compose --env-file development/deployment/catuc-production/.env.example -f development/deployment/catuc-production/compose.yaml config`
- `bash -n development/deployment/catuc-production/scripts/*`
- Run `scripts/preflight` locally and remotely before build, restore, cutover, or rollback.

## Child DOX Index
