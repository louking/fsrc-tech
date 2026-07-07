# tmtility

Manages results received from a Time Machine manual race timer for input to scoring software like [RaceDay Scoring](https://info.runsignup.com/products/raceday/raceday-scoring/).

- Repo: https://github.com/louking/tm-csv-connector
- Docs: https://tm-csv-connector.readthedocs.io/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/tm-csv-connector/blob/master/CLAUDE.md
- Runs on a laptop, so there's no publicly accessible site
- Receives results from the [Time Machine](https://timemachine.org/) over Bluetooth, bib numbers scanned from participant bibs, and optionally chip reads from a Trident RFID reader
- Creates a CSV file which can be read by race scoring software (designed for and tested with RaceDay Scoring)
- Also runs in "simulator mode" so a prospective scoring operator can practice

## Tech stack

- Backend: Flask, SQLAlchemy, Flask-Migrate (Alembic)
- Frontend: Jinja2, flask-assets bundles, WebSocket-driven live results view (`results.js`)
- Packaging: PyInstaller-built Windows client executables, distributed as zips and installed via NSSM as Windows services
- Shared libraries: [loutilities](../framework/loutilities.md) (`DbCrudApi` for table views)

## Architecture notes

- **Two operating modes**, toggled by `SIMULATION_MODE`: normal mode auto-logs in a default user, shows public views only, and talks to real hardware; simulation mode exposes full multi-user login and `/admin/*` routes, replaying recorded events for operator training.
- **Three external client processes** run as Windows services (not in the Docker container, since they need direct access to USB/Bluetooth hardware): `tm-reader-client` (Time Machine Bluetooth), `barcode-scanner-client` (bib scanner), `trident-reader-client` (Trident RFID telnet connection). Each connects to the Flask backend over WebSocket and also POSTs results directly.
- **Data model duality**: `Result` and `ScannedBib` each carry two nullable foreign keys — `race_id` (normal mode) and `simulationrun_id` (simulation mode) — exactly one is ever non-null, and queries must filter on the right one.
- **File locking**: a `threading.Lock` (`fileformat.filelock`) serializes any operation touching the CSV output file or related multi-table DB writes (place recalculation, scanned-bib queue assignment).
- **Race start time**: `Race.start_time` is the time-of-day offset added to Time Machine elapsed times when writing the CSV. It can be set manually or auto-populated from the first live Trident `GUNTIME` marker for a race — but only once; later markers are ignored.
- **Release pipeline**: builds PyInstaller client executables, Sphinx docs, and a Docker image (via VS Code's "Build/Push/Release" task), then packages a main distribution zip plus a separate vendor-JS zip (only rebuilt when JS content actually changes, tracked via a committed SHA256 manifest).

## Gotchas worth knowing

- **Windows-only clients**: internal laptop Bluetooth (CNVi/PCIe-based) can't be passed through to a Docker container via USB/IP, so `tm-reader-client` must run on Windows directly — this is a deliberate, stable design choice, not a temporary workaround.
- **Local HTTPS via Caddy**: a separate `caddy-docker` reverse proxy provides automatic HTTPS for `*.localhost` domains in dev (`https://tm.localhost` → the app's nginx container). See [infrastructure/caddy.md](../infrastructure/caddy.md).
- **Bind mounts only take effect on container recreation**, not a plain restart — `docker compose down && up` is needed if a mount is missing.
- **DataTables poll returns the full dataset every tick** (not incremental) — the results view can't use DataTables server-side mode because it needs to detect row deletions across the whole finisher list; composite indexes on `Result` keep the per-race query fast enough regardless.
- **Trident reader connectivity is a real half-open-TCP trap**: a bare ICMP ping success doesn't prove the telnet session to the reader is still alive — if the reader power-cycles without ever sending a FIN/RST, the client can keep reporting "connected" indefinitely while no data actually flows. `trident-reader-client` now auto-reconnects on genuine drops and enables short-interval TCP keepalive so the OS itself detects a half-dead socket, plus debounces status flapping and shows a louder banner/beep on the results page for degraded connectivity.

For full detail on any of the above (dev setup, full release pipeline, hardware protocols, JS architecture), see the repo's own `CLAUDE.md` linked above.
