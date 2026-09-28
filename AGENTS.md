# AGENTS.md — layer-supervisord

Standalone candy repo for the `supervisord` layer — the default container init
and process supervisor. The candy lives in `charly.yml` at the repo root: the
`supervisor` package, the `XDG_RUNTIME_DIR` environment, a `plan:` that creates
the per-user runtime directory and asserts the install, and the embedded
`skill:` entity projected into the marketplace corpus as
`/charly-infrastructure:supervisord`. The generated global config header lives at
`templates/supervisord.header.conf`.

Canonical files:

- `charly.yml` — the `supervisord:` candy entity and the `supervisord-skill:` skill entity.
- `templates/supervisord.header.conf` — the global `[supervisord]` /
  `[unix_http_server]` / `[supervisorctl]` / `[rpcinterface:supervisor]` header
  prepended to every generated `supervisord.conf`.
- `.github/workflows/` — the org-wide `charly/pr-validator` gate; there is no per-repo candy gate.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:supervisord` — the owning skill. Config generation, the
  `service:` → `[program:...]` mapping, priority ordering, `autostart` /
  `autorestart` / `startretries`, event listeners, and diagnostics. Load before
  editing or troubleshooting the layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, package sections, and the `service:`
  declaration consumed by this init system). Load before editing any entity
  field or plan step.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the org-wide `charly/pr-validator` (required check
  `validate / validate`); there is no per-repo candy gate.
- The candy's `plan:` ends in a runtime `check:` (`supervisorctl pid`,
  `context: [runtime]`) that proves supervisord is PID 1 and its control socket
  answers — prefer it over `supervisorctl status`, which exits non-zero when any
  program is legitimately non-RUNNING.
- Runtime control/diagnostics: `charly service status|start|stop|restart <image> [name]`
  and `charly logs <image>`.

## Modify this repo

- Edit the `supervisord:` candy entity AND the `supervisord-skill:` skill entity
  in `charly.yml` together. The skill is the projected usage source, so a
  behaviour change that is not mirrored in the skill leaves the corpus stale.
- Changes to the global config header belong in `templates/supervisord.header.conf`
  and must stay consistent with the plan checks and the skill's documented
  behaviour.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
