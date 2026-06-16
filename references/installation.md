# Installation

How to get Alby Hub running. Once it's up, use the CLI to manage it — see [Initial Setup](./initial-setup.md) for the first-time setup flow.

## Before installing: check for an existing hub

**Always check whether Alby Hub is already installed or running before installing anything.** Installing a second copy on a machine that already has one leads to conflicting data directories and ports.

```bash
# Is Alby Hub already installed as a systemd service? (the recommended install method)
systemctl status albyhub

# Is a hub already responding on a default port?
curl -s http://localhost:8029/api/info   # systemd install default port
curl -s http://localhost:8080/api/info   # docker / manual binary default port
```

If a service exists or a hub responds, **do not install again.** Start it if stopped (`sudo systemctl start albyhub` for a systemd install) and go straight to [Initial Setup](./initial-setup.md) — or, if it is already set up, just `start`/`unlock` it (see [Authentication](./authentication.md)).

## Recommended: systemd service (Linux)

On a Linux server, **the default and recommended way to install Alby Hub is the systemd install script** ([x86_64](#linux-x86_64-recommended-for-servers) / [aarch64](#linux-aarch64-arm64-servers)). It installs the hub as a service that starts automatically on boot and is restarted on failure. Do **not** reach for the manual binary install or write your own systemd unit unless the user explicitly asks for a throwaway/test setup — see [HTTP Server Binary (manual)](#linux-x86_64--aarch64--http-server-binary-manual---not-recommended).

This applies regardless of backend: LDK, Bark, and other env-var-configured backends all run fine under systemd — see [Configuring env vars for a systemd install](#configuring-env-vars-for-a-systemd-install).

## Alby Cloud (Managed Hosting)

See [Alby Cloud](./alby-cloud.md).

## Linux x86_64 (Recommended for Servers)

Installs as a systemd service that starts automatically on boot.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/getAlby/hub/master/scripts/linux-x86_64/install.sh)" -- -y --skip-verify -s
```

Flags: `-y` non-interactive, `--skip-verify` skip PGP signature verification, `-s` install as systemd service.

After the script completes, verify with `systemctl status albyhub`. If the script fails, diagnose and fix the specific error then re-run it — do **not** fall back to a manual binary install or create your own systemd service.

Default port: `http://localhost:8029`

To update later: run `./update.sh` in the install directory. Backup data lives in `[install path]/data`.

## Linux aarch64 (ARM64 Servers)

Installs as a systemd service that starts automatically on boot. Same approach as x86_64, different script:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/getAlby/hub/master/scripts/linux-aarch64/install.sh)" -- -y --skip-verify -s
```

Flags: `-y` non-interactive, `--skip-verify` skip PGP signature verification, `-s` install as systemd service.

After the script completes, verify with `systemctl status albyhub`. If the script fails, diagnose and fix the specific error then re-run it — do **not** fall back to a manual binary install or create your own systemd service.

## Configuring env vars for a systemd install

The systemd install runs the hub from a fixed working directory, so a `.env` file is **not** read. Backends that need extra environment variables (e.g. Bark on signet, a custom `NETWORK` or esplora server) must have those vars set on the systemd unit, **before** running `setup`.

Add a drop-in override and restart:

```bash
sudo systemctl edit albyhub
```

In the editor, add (one `Environment=` line per var):

```ini
[Service]
Environment="LN_BACKEND_TYPE=BARK"
Environment="BARK_SERVER=https://ark.signet.2nd.dev"
```

Then apply and restart:

```bash
sudo systemctl restart albyhub
```

Data (database, keys, logs, and the bark wallet directory) stays in `[install dir]/data` — the persistent location the install script configures — so there is no need for `WORK_DIR=.`. See [Backends](./backends.md) and [Bark (Ark)](./bark.md) for which vars each backend needs.

## Linux x86_64 / aarch64 — HTTP Server Binary (Manual - not recommended)

> **Only use this if the user explicitly asks for a manual / no-systemd / throwaway test setup.** For any normal install, use the [systemd service](#recommended-systemd-service-linux) instead — including for Bark and other env-var-configured backends. Do not pick this path just because a backend needs env vars; configure those in the systemd unit (see [below](#configuring-env-vars-for-a-systemd-install)).

For testing or running in a specific folder without systemd. The binary starts an HTTP server you manage yourself.

**Download the latest release:**

```bash
# Asset filenames follow this pattern — use the one matching your architecture:
#   albyhub-Server-Linux-x86_64.tar.bz2   (x86_64)
#   albyhub-Server-Linux-aarch64.tar.bz2  (ARM64)
curl -L --no-progress-meter -o albyhub.tar.bz2 \
  https://github.com/getAlby/hub/releases/latest/download/albyhub-Server-Linux-x86_64.tar.bz2
tar -xjf albyhub.tar.bz2
```

The binary is at `bin/albyhub`.

**Configuration via `.env` file:**

Hub reads a `.env` file in the working directory automatically — do not pass env vars inline and do not `source` the file manually. Create `.env` before starting.

**Ask the user** whether they want to use the default data directory (`~/.local/share/albyhub`) or keep data in the current folder. If they want data in the current folder, set `WORK_DIR=.` in the `.env`:

```
WORK_DIR=.
# Add any other vars here, e.g. NETWORK, MEMPOOL_API, LDK_ESPLORA_SERVER
```

Setting `WORK_DIR=.` keeps all hub data (database, logs, keys) in the current folder.

**Running the hub:**

Run in the background — the hub manages its own logging:

```bash
./bin/albyhub &
```

Do **not** redirect stdout (e.g. `> albyhub.log`). Logs are written to `{WORK_DIR}/log/nwc.log`.

Default port: `http://localhost:8080`

## Raspberry Pi 4/5 (Experimental)

```bash
/bin/bash -c "$(curl -fsSL https://getalby.com/install/hub/pi-aarch64-install.sh)"
```

Installs to `/opt/albyhub`. Update via `./update.sh`. Official support is limited for this platform.

## Docker

```bash
docker run -v ~/.local/share/albyhub:/data \
  -e WORK_DIR='/data' \
  -p 8080:8080 \
  ghcr.io/getalby/hub:latest
```

Docker Compose file: https://raw.githubusercontent.com/getAlby/hub/master/docker-compose.yml

Default port: `http://localhost:8080`

## Desktop App (Mac / Windows / Linux)

> **Warning:** Do not suggest the desktop app to users working with an agent. Agents cannot control or interact with the desktop GUI. Use one of the server/binary options above instead.

Download the latest release for your platform from https://github.com/getAlby/hub/releases/latest.

Platforms: Mac (arm64), Windows (amd64), Linux (amd64). Designed to run 24/7 on an always-online machine.

## Node Distributions (Umbrel / Start9)

- **Umbrel**: https://github.com/getAlby/umbrel-community-app-store
- **Start9**: https://github.com/horologger/nostr-wallet-connect-startos

Install via your node OS app store.

## System Requirements

| Backend                  | RAM                          | Disk    |
| ------------------------ | ---------------------------- | ------- |
| LDK (default)            | 2 GB (or 512 MB + 2 GB swap) | 1 GB+   |
| External (LND, Phoenixd) | 256 MB                       | Minimal |

lightning port 9735 should be open and reachable for optimal channel connectivity.

## Default Hub URL

After installation, the hub is available at:

- `http://localhost:8080` — Docker, desktop app, manual binary
- `http://localhost:8029` — systemd install scripts
