# Academia VPS Development

## Production boundary

Live `portal.catuc.org` deployment artifacts are owned by `catuc-production/` and install under `/srv/apps/academia/frappe`. The image repository is `registry.gitlab.com/zinger-teams/academia/frappe`; dated tags use `frappe16-[Y-m-d]`, and running releases must pin the resolved digest.

The production stack uses one external private network named `frappe` for every service. Only its `frontend` service also joins the shared Traefik edge network discovered from the live host. Backend and worker containers map `portal.catuc.org` to Docker `host-gateway`, preserving canonical HTTPS and certificate validation while PDF generators fetch print assets through Traefik without joining the edge network.

Run `catuc-production/scripts/preflight` before building or deploying. It refuses dirty app trees, blocked release manifests, missing Docker Buildx/Compose, placeholder edge inventory, and invalid image naming.

## Layout

- Shared ingress: `/srv/deploy/edge`, Compose project and external network `sandbox-edge`.
- CATUC repository: `/srv/projects/academia/catuc/frappe/frappe_docker`.
- CATUC backups: `/srv/projects/academia/catuc/backups`.
- CATUC secrets: `/srv/projects/academia/catuc/secrets` and `/srv/deploy/edge/secrets`.
- Frappe site: `academia-frappe-catuc`; internal URL `http://academia-frappe-catuc.localhost:8000`; public URL `https://portal.catuc.zingersystems.com`.

Future projects use their own Compose project, private data network, volumes, secrets, ports, and backup directory. Only their HTTP service joins `sandbox-edge`.

## First deployment

1. Create the `sandbox-edge` Docker network and the edge `dynamic`, `secrets`, and `logs` directories.
2. Generate the CATUC htpasswd and issued plaintext credentials with mode `0600`. Store the MariaDB root-password file as `colong:developers` with mode `0640` so the non-root devcontainer can use it through its supplemental developer group. Never commit any of these files.
3. Start `sandbox-edge`, then start the CATUC Compose project.
4. Run `catuc/scripts/bootstrap-bench` inside the `frappe` service.
5. Transfer a checksummed Frappe backup and restore it with `catuc/scripts/restore-site`.
6. Restart `frappe`, verify internal access, then verify the public authenticated route.

The Fail2ban templates under `sandbox-edge/fail2ban` must be installed into the matching `/etc/fail2ban` directories and Fail2ban reloaded with sudo. The jail polls the Docker bind-mounted Traefik log, parses its explicit UTC JSON timestamp, watches only CATUC 401 responses, bans after three failures in ten minutes for one hour, and inserts its HTTP/HTTPS-only rule into Docker's `DOCKER-USER` chain so an authentication ban does not block SSH. It can be inspected or reversed with:

```sh
sudo fail2ban-client status traefik-catuc-auth
sudo fail2ban-client set traefik-catuc-auth unbanip ADDRESS
```

The reviewed `sandbox-edge/install-host-integration` installer installs those templates, verifies the filter against a non-empty existing access log, enables agent forwarding only for the `developers` group, validates SSH configuration, restarts Docker, reloads Fail2ban, and prints the jail status. It is safe to rerun when host-integration templates change.

An IP ban affects every user behind the same public NAT address. Confirm the source address in the jail status before banning or unbanning, and use the manual unban immediately if a school or office gateway is caught by repeated bad credentials.

## Developer workflow

Each developer connects to the VPS with VS Code Remote SSH and reopens the repository in the existing devcontainer. Do not mount personal `.ssh` directories into the shared container. Developers register their own GitLab keys and use agent forwarding for remote Git operations.

Use `umask 0002` in the developer shell. The repositories are configured with Git shared-repository mode so new Git metadata remains writable by the `developers` group.

Acquire the shared runtime before changing branches:

```sh
development/deployment/catuc/scripts/runtime-session start USERNAME "Full Name" email@example.com feature/USERNAME/topic
```

The command verifies every repository is clean and pushed, updates `develop`, applies the repository-local Git identity, creates or resumes the same work branch in all four repositories, and then holds the runtime lock. Commit and push the feature branch, open GitLab merge requests targeting `develop`, then release the checkout; release refuses unpushed commits and returns every repository to `develop`:

```sh
development/deployment/catuc/scripts/runtime-session release
```

Only one developer may own the checkout. `staging` is frozen during announced customer tests; approved staging commits are promoted to `main` and tagged.

## Data and environment switching

`backup-site CATEGORY` writes a checksummed database, public files, private files, and site configuration beneath `/srv/backups`. The scheduler retains daily and weekly generations. `switch-environment develop|staging|main` backs up the active branch, holds the persistent Bench processes, checks out and verifies the target manifest, restores that branch's latest snapshot, migrates, builds custom assets, restarts Bench, requires an internal HTTP health check, and then leaves maintenance mode.

Environment switching is disruptive and must run only after notifying developers and enabling the public maintenance window. A target branch must already have a snapshot. Targeted data cleanup requires a reviewed Frappe command with dry-run output; raw SQL deletion is not an operational workflow.
