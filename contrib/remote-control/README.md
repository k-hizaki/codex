# Codex Remote Control Autopatch

This directory contains fork-local helper files for running a patched Codex
remote-control app-server and keeping it current with OpenAI Codex releases.

It is intended for hosts that install Codex from the standalone package under
`$CODEX_HOME/packages/standalone/current`.

## Files

- `codex-remote-originator-autopatch`: checks the latest stable `rust-vX.Y.Z`
  tag, applies `remote-originator.patch` when upstream does not contain the fix,
  builds a standalone package, switches `current`, and restarts remote control.
- `remote-originator.patch`: keeps ChatGPT Android/iOS remote clients from
  taking the app-server originator.
- `systemd/user/codex-remote-control.service`: runs the remote-control
  app-server as a persistent user service.
- `systemd/user/codex-remote-originator-autopatch.service` and `.timer`: run
  the autopatch check hourly.

## Install

From the repository root:

```bash
install -Dm755 contrib/remote-control/codex-remote-originator-autopatch \
  "$HOME/.local/bin/codex-remote-originator-autopatch"

install -Dm644 contrib/remote-control/remote-originator.patch \
  "$HOME/.codex/automation/remote-originator-autopatch/remote-originator.patch"

install -Dm644 contrib/remote-control/systemd/user/codex-remote-control.service \
  "$HOME/.config/systemd/user/codex-remote-control.service"

install -Dm644 contrib/remote-control/systemd/user/codex-remote-originator-autopatch.service \
  "$HOME/.config/systemd/user/codex-remote-originator-autopatch.service"

install -Dm644 contrib/remote-control/systemd/user/codex-remote-originator-autopatch.timer \
  "$HOME/.config/systemd/user/codex-remote-originator-autopatch.timer"

systemctl --user daemon-reload
systemctl --user enable --now codex-remote-control.service
systemctl --user enable --now codex-remote-originator-autopatch.timer
```

The default source checkout is `$HOME/code/openai-codex-latest`. Override it
when needed:

```bash
systemctl --user edit codex-remote-originator-autopatch.service
```

```ini
[Service]
Environment=CODEX_AUTOPATCH_REPO_SOURCE=/path/to/codex
```

## Check Status

```bash
systemctl --user status codex-remote-control.service
systemctl --user status codex-remote-originator-autopatch.timer
test -S "$HOME/.codex/app-server-control/app-server-control.sock"
"$HOME/.codex/packages/standalone/current/codex" --version
```

