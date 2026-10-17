# systemd Service Generator

A browser-based generator for systemd unit files. Fill in a form, watch the unit file build as you type, and get syntax checks and install commands alongside it.

**Live:** https://emptyteeth.github.io/sdtool/

Single file, no build step, no backend. Everything runs in your browser, and what you type stays in `localStorage` — nothing is uploaded anywhere.

## Features

**Generation**
- Grouped by the three unit sections, `[Unit]` / `[Service]` / `[Install]`, covering roughly 90 common directives
- Dependencies and ordering: `After` / `Wants` / `Requires` / `Before` / `PartOf` / `Conflicts`, plus `StartLimitIntervalSec` + `StartLimitBurst` for crash-loop rate limiting
- Process: `Type` (simple / exec / forking / oneshot / notify / dbus / idle) and the full `ExecStart` family (Pre / Post / Stop / StopPost / Reload)
- Restart and timeouts: `Restart` policy, `RestartSec`, `TimeoutStopSec`, `WatchdogSec`, `RemainAfterExit`
- Hardening: `NoNewPrivileges`, `ProtectSystem` / `ProtectHome`, `ProtectProc`, `DevicePolicy`, `DeviceAllow`, `SystemCallFilter`, `CapabilityBoundingSet` and more, with one-click "Apply recommended hardening"
- Resource limits: `MemoryMax`, `CPUQuota`, `LimitNOFILE`, `Nice`, `UMask` and others

**Three directives worth calling out**
- `DynamicUser` — emits `DynamicUser=yes`. If it conflicts with `User=`, you get a warning and the ignored `User=` is dropped from the output instead of silently producing a broken unit
- `Environment` — a multi-line `KEY=VALUE` box that emits one `Environment=` per line. Values containing spaces are quoted and escaped automatically, `#` lines stay comments, and malformed lines are reported
- `StateDirectory` — provided together with `CacheDirectory`, `LogsDirectory`, `ConfigurationDirectory` and `RuntimeDirectory`. These take a directory name only; an absolute path or illegal characters raise an error and are left out of the output

**Checks and helpers**
- Live syntax checks in three severities (error / warning / note), for example:
  - `ExecStart` is not an absolute path
  - `Restart` is set without `RestartSec` (the default is only 100ms, so a crash loop floods the journal)
  - `ProtectSystem=strict` without `ReadWritePaths`
  - `WatchdogSec` on a unit that is not `Type=notify`
  - `MemoryDenyWriteExecute` will kill JIT runtimes
  - `DeviceAllow` has no effect unless `DevicePolicy` is `strict` or `closed`
- An "Install commands" tab with the full sequence: write the file, `systemd-analyze verify`, `daemon-reload`, `enable --now`, `journalctl -f`
- English and Chinese interface, switchable in the header; the first visit picks a language from the browser
- Dark / light / follow-system themes
- Two-column desktop layout; on phones the output collapses into a bottom drawer

## Using it locally

Open `index.html` in a browser. Nothing to install.

To serve it over HTTP instead (optional):

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Project structure

```
index.html                       the whole app: HTML, CSS and JS in one file
.github/workflows/pages.yml      GitHub Pages deployment
```

A few key structures in the code:

- `SCHEMA` — field definitions. Each group maps to one unit file section, and each field's copy is written as a `{zh, en}` pair
- `STR` — UI strings, check messages and generated-file comments, looked up by key with `{0}` placeholders
- `buildLines()` — turns form values into the sequence of unit file directives
- `validate()` — the syntax check rules
- `render()` — reruns all of the above after every input and updates the UI

## Deployment

The repo is set up to publish with GitHub Actions. Pushing to `main` publishes `index.html` as the site root automatically, and the workflow can also be triggered from the Actions tab.

On a fresh fork, set **Settings → Pages → Build and deployment → Source** to **GitHub Actions**.

## Known limitations

- The checks are heuristic and cover common mistakes, but they are not `systemd-analyze verify`. Run that command before deploying for real; the generated install commands include it
- There is no reverse parser for importing an existing unit file
- The directive set covers common options, not all of systemd

## License

MIT
