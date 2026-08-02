# VPS Deployment

## Purpose

- Owns reproducible container, ingress, backup, environment-switching, and developer-session configuration for Academia VPS projects.

## Ownership

- `catuc/` owns the single `academia-frappe-catuc` development stack and its operational commands.
- `sandbox-edge/` owns the shared VPS Traefik template, client authentication middleware, and Fail2ban configuration.

## Local Contracts

- CATUC uses one application Compose project and one active Frappe database at a time.
- `sandbox-edge` is shared infrastructure; application databases and Redis services never join its network.
- Secrets, backups, runtime locks, generated manifests, and certificate state stay outside Git.
- Environment changes must be backed up before source or schema changes are applied.
- Never copy developer private SSH keys into the repository, VPS project tree, or devcontainer.

## Work Guidance

- Validate Compose with `docker compose config` before deployment.
- Keep scripts POSIX-oriented Bash with strict error handling and explicit target validation.
- Add future clients through isolated project directories and the shared `sandbox-edge` network.

## Verification

- `docker compose --env-file development/deployment/catuc/.env.example -f development/deployment/catuc/compose.yaml config`
- `docker compose -f development/deployment/sandbox-edge/compose.yaml config`
- `bash -n development/deployment/catuc/scripts/*`

## Child DOX Index

