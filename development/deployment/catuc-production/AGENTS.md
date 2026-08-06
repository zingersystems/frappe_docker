# CATUC Production

## Purpose

- Owns the immutable production image inputs, Compose topology, release manifests, backup and restore commands, cutover, health checks, and rollback for `portal.catuc.org`.

## Ownership

- `compose.yaml` owns the production Frappe, MariaDB, Redis, private-network, and Traefik attachment topology.
- `.env.example` documents non-secret deployment inputs; live values and credentials stay under `/srv/apps/academia/frappe/secrets`.
- `scripts/` owns strict operational commands for release validation, deployment, data movement, health checks, cutover, and rollback.
- `image/Containerfile` follows the official layered image and differs only by adding the SSH client to the disposable builder and mounting SSH and known-host data during private app clones.
- `apps.json` contains reviewed stable refs; the build script verifies each ref still resolves to the release manifest commit before invoking the official `bench init` flow.
- `manifests/` owns reviewed source and image release records, never credentials or live snapshots.

## Local Contracts

- The live project root is `/srv/apps/academia/frappe` and the public hostname is `portal.catuc.org`.
- The live Compose project is `academia-frappe`; service keys are role names without environment prefixes, and persistent volumes use explicit `academia-frappe_*` physical names.
- Every service joins the private external network named `frappe`; only `frontend` joins the discovered shared edge network.
- MariaDB, Redis, backend, websocket, workers, and scheduler publish no host ports.
- Images use `registry.gitlab.com/zinger-teams/academia/frappe:frappe16-[Y-m-d]` tags and production deploys pin the resolved image digest.
- Private app builds use forwarded SSH agent and known-host mounts through BuildKit; keys and credentials never enter the build context, image layers, or VPS filesystem.
- Site migrations run through the dedicated `migrator` service and must complete before the frontend starts.
- `restore` is an operations-only, one-shot service that verifies the authority snapshot, creates fresh production database credentials, restores database/files and the encryption key, and leaves the site in maintenance mode.
- Compose volume renames are offline migrations: stop both project names, copy with metadata preservation, validate source/target entry counts and byte totals, accept the new stack, and only then remove old volumes.
- The VPS bind filesystem root-squashes mounted ownership. Mounted secret and snapshot leaf files therefore require read bits inside containers; confidentiality is enforced by their mode-0700 host parent directories and by mounting them only into declared services.
- Cutover switches the frontend router first and must render the exact active Email Account PDF through canonical HTTPS before legacy workers stop or candidate workers/scheduler start. Any failed gate restores the legacy router.
- Production configures the built-in-name `Standard` Print Format selector with `pdf_generator=chrome`; this leaves Standard template rendering unchanged while making queued `attach_print` use the image's configured Chromium engine.
- Backend and worker roles resolve `portal.catuc.org` through Docker `host-gateway` so PDF generators fetch canonical HTTPS assets through Traefik without joining the edge network.
- Production email stays muted until the exact formerly failing document renders as a valid PDF and one approved print-attached message reaches `Sent`.
- Old-live data is rollback evidence. The checksummed `development.localhost` snapshot is authoritative and is restored without merging old-live records.
- Full authority and rollback snapshots contain secrets and private files; keep them under ignored local `.deployment-snapshots` or live `/srv/apps/academia/frappe/backups`, never Git.
- During routine in-place releases, clear site maintenance mode before waiting on the backend health check; the health endpoint returns a maintenance response and cannot become healthy while maintenance remains enabled.

## Work Guidance

- Derive runtime commands and image behavior from the repository's official Frappe Docker production patterns.
- Refuse deployment when source trees are dirty, image refs are mutable, required secrets are absent, or the shared edge/private network inventory is unresolved.
- Keep Bash scripts strict, idempotent where practical, and explicit about target host, project, site, release, and snapshot.

## Verification

- `docker compose --env-file development/deployment/catuc-production/.env.example -f development/deployment/catuc-production/compose.yaml config`
- `bash -n development/deployment/catuc-production/scripts/*`
- Run `scripts/preflight` locally and remotely before build, restore, cutover, or rollback.

## Child DOX Index
