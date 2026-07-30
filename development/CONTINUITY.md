# Workspace Continuity

## Progress

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

- Use `AGENTS.md` for stable contracts and `CONTINUITY.md` for progress, decisions, memories, and handoff notes.
- Commit changes separately in each affected Git repository, including nested custom app repositories under `frappe-bench/apps/`.
- Do not commit unrelated dirty files or runtime artifacts when making scoped workflow or implementation changes.
- Production deployment must reproduce durable local behavior through app install/migrate automation; manual local setup is incomplete until it is codified and verified.

## Memory

- Parent Git repository root is `/workspace`; the active project folder is `/workspace/development`.
- `frappe-bench/apps/academia_core`, `frappe-bench/apps/academia_catuc`, `frappe-bench/apps/frappe_pay_connect`, and `frappe-bench/apps/payments` are nested Git repositories.

## Handoff

- Root and bench-level DOX files are tracked by the parent repository after `.gitignore` exceptions.
- App-level DOX and continuity files should be committed from inside their nested app repositories.
- Payments install printed a non-fatal desktop icon warning (`NoneType` object has no attribute `startswith`) during `install-app`; subsequent migrate succeeded and the app is listed for `development.localhost`.
- The `CATUC Bamenda - MTN Mobile Money` MTN MoMo Settings record exists with matching `gateway_name`; the old `CATC Bamenda - MTN Mobile Money` record no longer exists.
- The MTN MoMo gateway stack now resolves as `MTN MoMo-CATUC Bamenda - MTN Mobile Money`, gateway account `MTN MoMo-CATUC Bamenda - MTN Mobile Money - XAF - CATUC BDA`, and clearing account `MTN MoMo-CATUC Bamenda - MTN Mobile Money - CATUC BDA`.
