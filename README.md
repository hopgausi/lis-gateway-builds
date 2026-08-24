# LIS Gateway Builds

Prebuilt binaries of **LIS Gateway** — a multi-protocol lab instrument integration
gateway with an embedded admin web UI. No installer, no dependencies to install
separately: each file below is a single self-contained executable.

> **A license key is required to run these.** Downloading is open to anyone,
> but the binary itself won't start without one — see
> [License key](#license-key) below for how to get one before you go any
> further.

| File                                                                                                         | Platform                      |
| ------------------------------------------------------------------------------------------------------------ | ----------------------------- |
| [`lis-gateway`](https://github.com/hopgausi/lis-gateway-builds/releases/latest/download/lis-gateway)         | Linux (Ubuntu/Debian, x86_64) |
| [`lis-gateway.exe`](https://github.com/hopgausi/lis-gateway-builds/releases/latest/download/lis-gateway.exe) | Windows (x86_64)              |

Those two links always resolve to the **most recent release**, whatever it is.
To grab a specific older version instead — or to see that exact build's file
size and SHA-256 — open the [Releases page](https://github.com/hopgausi/lis-gateway-builds/releases)
and pick a tag. Each release has `lis-gateway`, `lis-gateway.exe`, and a
`checksums.txt` attached.

## Download

**Browser:** click a file name in the table above (goes straight to the
current release's asset), or browse the [Releases page](https://github.com/hopgausi/lis-gateway-builds/releases)
for a specific version.

**Command line:**

```bash
# Linux — always the latest release
curl -LO https://github.com/hopgausi/lis-gateway-builds/releases/latest/download/lis-gateway
chmod +x lis-gateway
```

```powershell
# Windows (PowerShell) — always the latest release
Invoke-WebRequest -Uri https://github.com/hopgausi/lis-gateway-builds/releases/latest/download/lis-gateway.exe -OutFile lis-gateway.exe
```

To pin a specific version instead of "latest," swap `latest/download` for
`download/<tag>`, e.g. `.../releases/download/v1.0.0/lis-gateway`.

**Verify the download** against that release's own checksums:

```bash
# Linux
curl -LO https://github.com/hopgausi/lis-gateway-builds/releases/latest/download/checksums.txt
sha256sum -c checksums.txt
```

```powershell
# Windows — download checksums.txt from the release page, then:
certutil -hashfile lis-gateway.exe SHA256
# ...and compare the output by eye against the matching line in checksums.txt
```

> **Note:** `lis-gateway` and `lis-gateway.exe` also sit at the root of this
> repository's `main` branch, automatically kept in sync with the latest
> release's build. That's a convenience for browsing the repo directly — the
> Releases page above is still the authoritative source, since it's the only
> place a specific version's checksum is recorded.

## License key

`lis-gateway` checks for a valid, signed license key the moment it starts —
before it opens a database or listens on a port. Without one, it exits
immediately with `Cannot start: ...` and doesn't do anything else.

**To get a key, reach out via [github.com/hopgausi](https://github.com/hopgausi)**
— my profile there has ways to contact me.

Once you have a key, pass it as `--license-key` or set it in the
`LIS_LICENSE_KEY` environment variable (see [Configuration](#configuration)
below)

## Run it — Linux

```bash
./lis-gateway --license-key "<the key you were given>"
```

That's it. On first run it creates `./data/lis_gateway.db` (its own embedded SQLite
database, self-migrating) and prints a one-time bootstrap login to the terminal:

```
no users existed — created a bootstrap admin account. Log in and change this password immediately.
username="admin" password="<random>"
```

Copy that password — it's shown exactly once and isn't recoverable afterward.
Then open **http://localhost:8443** in a browser and sign in with it. Change it
(or create your own named account) from the **Users** page right away.

Stop it with `Ctrl+C`.

## Run it — Windows

Double-click `lis-gateway.exe` if you've set `LIS_LICENSE_KEY` as a permanent
environment variable, or run it from PowerShell/cmd with the key inline:

```powershell
.\lis-gateway.exe --license-key "<the key you were given>"
```

**Windows will likely show a SmartScreen warning** ("Windows protected your PC")
the first time, since this binary isn't code-signed. Click **More info** → **Run
anyway** to continue — this is expected for an unsigned executable, not a sign
of a problem with the build.

**Windows Defender Firewall may also prompt** to allow the app network access
(it needs to listen on a TCP port) — allow it, at least on private/lab networks.

Same as Linux: the first run prints a one-time bootstrap `admin` password to the
console window, creates `.\data\lis_gateway.db` next to the executable, and the
admin site is reachable at **http://localhost:8443**. Keep the console window
open — closing it stops the gateway (or run it as a background/scheduled service
if you want it to survive logging out; not covered here).

## Configuration

Every setting is a command-line flag (or an equivalent environment variable, if
you'd rather not repeat it every launch):

| Flag                      | Env var                     | Default                 | What it controls                                                                                                  |
| ------------------------- | --------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `--license-key`           | `LIS_LICENSE_KEY`           | _(required)_            | Your signed license key — see [License key](#license-key) above. Missing or invalid and the app refuses to start. |
| `--admin-port`            | `LIS_ADMIN_PORT`            | `8443`                  | Port the admin web UI + REST API listens on.                                                                      |
| `--db`                    | `LIS_DB`                    | `./data/lis_gateway.db` | Path to the database file. Created automatically if missing.                                                      |
| `--log-level`             | `LIS_LOG_LEVEL`             | `info`                  | Default log verbosity (`trace`/`debug`/`info`/`warn`/`error`).                                                    |
| `--rate-limit-per-minute` | `LIS_RATE_LIMIT_PER_MINUTE` | `600`                   | Per-IP request limit on the admin web UI/API before it starts rejecting requests. `0` disables it.                |

Example — run on a different port with a database file kept elsewhere:

```bash
./lis-gateway --license-key "<key>" --admin-port 9000 --db /var/lib/lis-gateway/lis_gateway.db
```

```powershell
.\lis-gateway.exe --license-key "<key>" --admin-port 9000 --db C:\lis-gateway\lis_gateway.db
```

Setting `LIS_LICENSE_KEY` once (in your shell profile, a systemd unit's
`Environment=`, a Windows service's environment, etc.) means every other
example on this page works without repeating `--license-key` each time.

TLS isn't built in — if you need HTTPS, put a reverse proxy (nginx, Caddy, IIS)
in front of it.

## What this actually is

LIS Gateway sits between lab instruments (hematology/chemistry analyzers, etc.)
and a hospital LIS: it speaks the instruments' own wire protocols (HL7 over
MLLP, ASTM), and forwards parsed results wherever you configure — a log, an
outbound REST endpoint, or both. Everything — which instruments exist, which
ports they're on, where results go, who's allowed to change any of it — is
managed from the admin web UI these binaries serve.

These two files are release builds straight from the project's source, not
separately packaged installers — there's nothing else to unpack or install.
