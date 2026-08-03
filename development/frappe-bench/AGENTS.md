# Frappe Bench

## Purpose

- Owns the primary local Frappe development bench and the source mirrored into CATUC VPS staging.
- Contains app source trees, site state, bench configuration, process files, logs, and the Python virtual environment.

## Ownership

- `apps/` contains editable Frappe apps and has its own DOX file.
- `sites/` contains bench site configuration and generated site data; treat as runtime state unless the user explicitly asks for configuration changes.
- `config/`, `Procfile`, and `patches.txt` are bench runtime and process configuration.
- `env/`, `logs/`, `config/pids/`, and `sites/assets/` are generated/runtime artifacts and should not receive durable documentation edits unless the user explicitly asks.

## Local Contracts

- Run bench commands from `frappe-bench/` unless a command explicitly requires an app directory.
- Use `development.localhost` locally and `academia-frappe-catuc` on the CATUC VPS; commands must name the active site explicitly.
- The VPS serves `academia-frappe-catuc` by default, uses `academia-frappe-catuc.localhost` internally, and publishes it through `portal.catuc.zingersystems.com`.
- Do not edit upstream framework app code to implement Academia behavior; prefer custom apps, hooks, DocTypes, custom fields, patches, fixtures, and services.
- Preserve user or runtime data in `sites/`; avoid destructive site operations without explicit approval.

## Work Guidance

- Prefer focused Frappe checks such as `bench --site development.localhost migrate`, `bench build --app <app>`, or Python compile checks for touched apps.
- Use the bench virtual environment at `env/bin/python` for direct Python validation.
- Keep generated assets, caches, pids, and logs out of source-oriented changes.

## Verification

- `bench --site development.localhost migrate`
- `bench build --app <app>`
- `env/bin/python -m compileall apps/<app>/<module>`

## Child DOX Index

- `apps/AGENTS.md` covers installed Frappe app source trees and app-specific development rules.
