# supervisord

The container init and process supervisor for OpenCharly images.

`supervisord` is the default init system for OpenCharly container images. The
layer installs the `supervisor` package — `/usr/bin/supervisord` runs as PID 1
and `/usr/bin/supervisorctl` is the control client — and provides a per-user
`XDG_RUNTIME_DIR` at `/tmp/xdg-runtime` (mode `0700`) so older podman/buildah and
Wayland/PipeWire tooling all accept it. Candies declare their long-running
processes with the unified `service:` schema; `charly` renders those declarations
into `/etc/supervisord.conf`, and the container's lifetime is supervisord's
lifetime.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `supervisord` |
| Package | `supervisor` (Fedora RPM / Arch pac / Debian-Ubuntu DEB) |
| Binaries | `/usr/bin/supervisord`, `/usr/bin/supervisorctl` |
| Environment | `XDG_RUNTIME_DIR=/tmp/xdg-runtime` |
| Init role | Default init system for container images (PID 1) |
| Control socket | `/tmp/supervisor.sock` |
| Service / port | none of its own |

Additional pieces:

- `templates/supervisord.header.conf` — the global config header
  (`[supervisord]`, `[unix_http_server]`, `[supervisorctl]`,
  `[rpcinterface:supervisor]`) prepended to every generated `supervisord.conf`.
- Service ordering is by `priority:` (lower starts earlier); `restart:`,
  `start_secs:` and `start_retries:` shape crash-recovery behaviour.

## How to use it

You rarely add `supervisord` to a box by hand: any candy that declares a
`service:` pulls it in automatically. A service-declaring candy looks like:

```yaml
my-service:
  candy:
    service:
      - name: my-service
        exec: /usr/local/bin/my-service --flag
        restart: always
        priority: 30
```

To compose the init layer explicitly:

```yaml
my-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-supervisord:v2026.240.0121'
```

Inspect and control services from the host:

```bash
charly service status <image>
charly service start <image> <name>
charly service restart <image> <name>
charly logs <image>
```

## Layout

- `charly.yml` — the candy manifest: the `supervisor` package, the
  `XDG_RUNTIME_DIR` environment, the `/tmp/xdg-runtime` creation step, the
  `plan:` checks, and the embedded `skill:` entity.
- `templates/supervisord.header.conf` — the global config header.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:supervisord`
- Service control: `/charly-core:service`, `/charly-core:logs`
- Sibling services (dbus, pipewire, compositors) all declare `service:` entries
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
