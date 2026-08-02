# Workspace Continuity

## Progress

- 2026-08-02: Provisioned `/srv/projects/academia/catuc/frappe/frappe_docker` as the single `academia-frappe-catuc` Compose stack and `/srv/deploy/edge` as the shared `sandbox-edge` Traefik project. Restored the checksummed `development.localhost` database and files into site `academia-frappe-catuc`; matched 839 tables, 14 users, 164 File records, and 47 physical files totaling 1,799,858 content bytes. Internal HTTP, public Let's Encrypt HTTPS, project Basic Auth, Socket.IO, Mailpit, MTN sandbox configuration, worker/scheduler operation, network isolation, scheduled/branch backups, and a process-safe `develop -> staging -> develop` data/source switch were verified. Remote runs passed Python compilation plus 196 Core, 70 CATUC, and 65 Pay Connect tests. Removed the one orphan File row created by the CATUC end-to-end test through Frappe's document API and refreshed all three clean branch baselines.
- 2026-08-02: Ran `/srv/deploy/edge/install-host-integration`; SSH agent forwarding successfully accessed the private CATUC GitLab origin, and both Compose projects, the internal/public routes, and the scheduler recovered after the Docker restart. A live three-request Fail2ban test exposed that the auto backend recorded zero failures despite the filter matching the same JSON offline. Changed the jail to deterministic polling, pinned the Traefik UTC JSON timestamp, and added an installer-side filter validation; the revised host templates require installation and a repeated live enforcement/unban test.

- 2026-07-30: Established production reproducibility as a workspace contract: every durable schema, configuration, seed, and data change must be delivered through app install/migrate mechanisms rather than manual Desk or database replication. Current custom apps already expose install/migrate hooks; the Educational Level presentation refactor is covered by standard DocType sync, a guarded Core patch, and idempotent CATUC provisioning.
- 2026-07-27: Added optional Student Admission Processing Rules beneath Application Payment Policy to require confirmed payment before review and/or an admission decision. Pay Connect owns the default-off controls; Core enforces Paid or Waived before the configured lifecycle boundaries. Migration, visual verification, all 97 Core tests, and all 26 Pay Connect tests passed.
- 2026-07-27: Replaced legacy aggregate application-fee labels with explicit Not Required, Unpaid, Pending Payment, Proof Submitted, Pending Verification, Paid, Proof Rejected, Failed, Refunded, and Waived states. Migrated live applicants from payment evidence, made Education's `paid` checkbox derived/read-only, and updated online/proof projections plus Desk and portal behavior. All 96 Core, 25 Pay Connect, and 22 CATUC tests passed with all three app builds.
- 2026-07-26: Replaced the custom admissions collection-mode configuration with a requirement plus independent online, proof-upload, and pay-later options. Added private proof review and settlement accounting, updated the applicant Payment step and status behavior, migrated existing policy values, and refreshed the admissions requirements guide. All 23 Pay Connect, 94 Core, and 22 CATUC tests passed with all three app builds.
- 2026-07-12: Installed DOX `AGENTS.md` and initialized the workspace DOX hierarchy for the Frappe bench, app collection, custom apps, research, and VS Code examples.
- 2026-07-12: Added continuity and automatic commit workflow requirements so future agents record handoff context and commit validated changes per affected repository.
- 2026-07-26: Installed Frappe Payments on `development.localhost` from `frappe/payments` branch `version-16` at `cca07d9`; `bench --site development.localhost migrate` completed successfully.
- 2026-07-26: Created income account `7066 - HND Application Fee Income - CATUC BDA` under `706 - Services vendus - CATUC BDA` for HND application-fee Payment Policies on `development.localhost`; verified it is the sole matching CATUC account and uses XAF.
- 2026-07-26: Renamed `MTN MoMo Settings` document `CATC Bamenda - MTN Mobile Money` to `CATUC Bamenda - MTN Mobile Money` on `development.localhost` using forced Frappe rename because the DocType has `allow_rename` disabled.
- 2026-07-26: Renamed the related MTN MoMo `Payment Gateway`, `Payment Gateway Account`, and XAF bank clearing `Account` records from `CATC` to `CATUC` labels on `development.localhost`.

## Decisions

- CATUC uses one shared checkout and one active database in Compose project `academia-frappe-catuc`; branch-aligned data snapshots and manifests provide environment switching rather than parallel application stacks.
- VPS-wide HTTP/HTTPS ingress belongs to the separate shared-infrastructure project `sandbox-edge`; only application web containers join its external Docker network.
- Developer SSH keys remain off the VPS. Git remote access requires each developer's forwarded agent, repository-local identity, and the serialized runtime session.
- Use `AGENTS.md` for stable contracts and `CONTINUITY.md` for progress, decisions, memories, and handoff notes.
- Commit changes separately in each affected Git repository, including nested custom app repositories under `frappe-bench/apps/`.
- Do not commit unrelated dirty files or runtime artifacts when making scoped workflow or implementation changes.
- Production deployment must reproduce durable local behavior through app install/migrate automation; manual local setup is incomplete until it is codified and verified.

## Memory

- Parent Git repository root is `/workspace`; the active project folder is `/workspace/development`.
- `frappe-bench/apps/academia_core`, `frappe-bench/apps/academia_catuc`, `frappe-bench/apps/frappe_pay_connect`, and `frappe-bench/apps/payments` are nested Git repositories.

## Handoff

- The active VPS runtime is `develop`, all four shared repositories are on `develop`, and their SSH origin URLs are restored. The client gate password is retrievable by the project owner from `/srv/projects/academia/catuc/secrets/catuc-bda.initial-password`; do not copy it into Git or continuity docs.
- Install the revised polling-based Fail2ban template with `sudo /srv/deploy/edge/install-host-integration`, then repeat the three-failure ban and manual-unban test. The other host-integration checks are complete: agent forwarding reaches GitLab, both Compose projects recover after Docker restart, internal HTTP returns 200, the unauthenticated public route returns 401, and the scheduler is enabled.
- Root and bench-level DOX files are tracked by the parent repository after `.gitignore` exceptions.
- App-level DOX and continuity files should be committed from inside their nested app repositories.
- Payments install printed a non-fatal desktop icon warning (`NoneType` object has no attribute `startswith`) during `install-app`; subsequent migrate succeeded and the app is listed for `development.localhost`.
- The `CATUC Bamenda - MTN Mobile Money` MTN MoMo Settings record exists with matching `gateway_name`; the old `CATC Bamenda - MTN Mobile Money` record no longer exists.
- The MTN MoMo gateway stack now resolves as `MTN MoMo-CATUC Bamenda - MTN Mobile Money`, gateway account `MTN MoMo-CATUC Bamenda - MTN Mobile Money - XAF - CATUC BDA`, and clearing account `MTN MoMo-CATUC Bamenda - MTN Mobile Money - CATUC BDA`.
