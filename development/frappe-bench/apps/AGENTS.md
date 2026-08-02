# Frappe Apps

## Purpose

- Owns the installed Frappe app source trees for this bench.
- Separates local Academia implementation apps from upstream framework and product apps.

## Ownership

- `academia_core/` owns reusable admissions engine behavior and has its own DOX file.
- `academia_catuc/` owns CATUC-specific admissions configuration and has its own DOX file.
- `frappe_pay_connect/` owns provider-neutral payment policy and transaction behavior and has its own DOX file.
- `frappe/`, `erpnext/`, `education/`, and `payments/` are upstream apps; treat them as dependencies unless the user explicitly requests framework or upstream app edits.

## Local Contracts

- Implement Academia behavior in `academia_core`, `academia_catuc`, or `frappe_pay_connect` according to ownership, not in upstream `frappe`, `erpnext`, `education`, or `payments` code.
- Keep Frappe app package files, module packages, hooks, DocTypes, patches, public assets, templates, and `www/` routes consistent with Frappe conventions.
- Frappe module folders listed in `modules.txt` must be importable Python packages with `__init__.py` where required by the framework.
- Every durable schema, configuration, seed, and data change must reach production through standard DocType sync, `after_install`/`after_migrate`, an ordered patch, a fixture, or another idempotent app-owned provisioner. A manual Desk or database operation may be used for diagnosis but must be codified before closeout.
- `academia_core`, `academia_catuc`, and `frappe_pay_connect` use private GitLab origins beneath `Zinger-Teams/catuc-bamenda/frappe`, with `develop` for integration, `staging` for customer tests, and `main` for approved releases.
- Shared VPS development uses one checkout; developers must acquire the runtime session, set repository-local Git identity, branch from `develop`, and release the runtime after pushing their work.

## Work Guidance

- For Python code, follow each app's `pyproject.toml` conventions: Python 3.14 target, tabs for formatter indentation, double quotes, and 110 character line length.
- For public pages and assets, prefer app-owned templates, `www/` routes, `public/css`, and `public/js`.
- Avoid broad formatting or dependency changes across upstream apps.
- Verify new provisioning on both upgrade and fresh-install paths when their behavior differs, and preserve administrator-owned values unless the migration explicitly replaces a recognized generated value.

## Verification

- `cd /workspace/development/frappe-bench && env/bin/python -m compileall apps/<app>/<module>`
- `cd /workspace/development/frappe-bench && bench --site development.localhost migrate`
- `cd /workspace/development/frappe-bench && bench build --app <app>`

## Child DOX Index

- `academia_core/AGENTS.md` covers the reusable admissions engine.
- `academia_catuc/AGENTS.md` covers CATUC-specific admissions configuration.
- `frappe_pay_connect/AGENTS.md` covers payment policy and payment transaction integration.
