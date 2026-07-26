# Workspace Continuity

## Progress

- 2026-07-12: Installed DOX `AGENTS.md` and initialized the workspace DOX hierarchy for the Frappe bench, app collection, custom apps, research, and VS Code examples.
- 2026-07-12: Added continuity and automatic commit workflow requirements so future agents record handoff context and commit validated changes per affected repository.
- 2026-07-26: Installed Frappe Payments on `development.localhost` from `frappe/payments` branch `version-16` at `cca07d9`; `bench --site development.localhost migrate` completed successfully.
- 2026-07-26: Renamed `MTN MoMo Settings` document `CATC Bamenda - MTN Mobile Money` to `CATUC Bamenda - MTN Mobile Money` on `development.localhost` using forced Frappe rename because the DocType has `allow_rename` disabled.
- 2026-07-26: Renamed the related MTN MoMo `Payment Gateway`, `Payment Gateway Account`, and XAF bank clearing `Account` records from `CATC` to `CATUC` labels on `development.localhost`.

## Decisions

- Use `AGENTS.md` for stable contracts and `CONTINUITY.md` for progress, decisions, memories, and handoff notes.
- Commit changes separately in each affected Git repository, including nested custom app repositories under `frappe-bench/apps/`.
- Do not commit unrelated dirty files or runtime artifacts when making scoped workflow or implementation changes.

## Memory

- Parent Git repository root is `/workspace`; the active project folder is `/workspace/development`.
- `frappe-bench/apps/academia_core`, `frappe-bench/apps/academia_catuc`, `frappe-bench/apps/frappe_pay_connect`, and `frappe-bench/apps/payments` are nested Git repositories.

## Handoff

- Root and bench-level DOX files are tracked by the parent repository after `.gitignore` exceptions.
- App-level DOX and continuity files should be committed from inside their nested app repositories.
- Payments install printed a non-fatal desktop icon warning (`NoneType` object has no attribute `startswith`) during `install-app`; subsequent migrate succeeded and the app is listed for `development.localhost`.
- The `CATUC Bamenda - MTN Mobile Money` MTN MoMo Settings record exists with matching `gateway_name`; the old `CATC Bamenda - MTN Mobile Money` record no longer exists.
- The MTN MoMo gateway stack now resolves as `MTN MoMo-CATUC Bamenda - MTN Mobile Money`, gateway account `MTN MoMo-CATUC Bamenda - MTN Mobile Money - XAF - CATUC BDA`, and clearing account `MTN MoMo-CATUC Bamenda - MTN Mobile Money - CATUC BDA`.
